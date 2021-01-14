# book-microservice-pattern
`마이크로서비스 패턴, 길벗` 도서 요약 정리입니다.   
`FTGO`라는 `음식배달` 서비스를 예제로 합니다.    
기존에는 `WAR`파일로 구성된 `모놀리스`로 개발되었으나 서비스가 커지면서 많은 문제가 발생합니

#
## 4. 트랜잭션 관리: `사가`
마이크로서비스 아키텍처에서 트랜잭션에 대한 내용을 다룬다.  
단일 서비스 내부의 트랜잭션은 `ACID`가 보장하지만, 여러 서비스의 데이터를 업데이트하는 트랜잭션은 고려해야 하는 점이 많다.  


여러 서비스에 걸친 작업의 데이터 일관성을 유지하려면 ACID 트랜잭션 대신 `사가(saga)`라는 메시지 주도 방식의 로컬 트랜잭션을 사용해야 한다.  
사가는 ACID에서 `I(격리성)`가 빠진 `ACD(원자성, 일관성, 지속성)`만 지원하고 격리가 되지 않기 때문에 동시 비정상의 영향을 방지하거나 줄일 수 있는 대책 또한 필요하다.  

 
#
createOrder()는 주문 가능한 소비자인지 확인하고, 주문 내역을 확인하고, 소비자의 신용카드를 승인하고, DB에 주문을 생성하는 작업이다.  
모놀로식과는 달리 여러 서비스에 흩어져 있기 때문에 소비자 서비스, 주문 서비스, 회계 서비스 등 여러 데이터에 접근해야 한다.  
서비스마다 DB가 따로 있기 때문에 여러 DB에 걸쳐 데이터 일관성을 유지할 수단을 강구해야 한다.  


예전에는 분산 트랜잭션을 이용했었는데 문제점이 많다.  
NoSQL DB와 메시지 브로커는 분산 트랜잭션을 지원하지 않는다.  
또한 동기형 IPC라서 가용성도 떨어진다.  


`메시지`를 DB 트랜잭션의 일부로 전송하는 방법, 느슨하게 결합된 `비동기` 서비스 메커니즘인 사가를 알아보자.  

#
#### 사가 패턴
여러 서비스의 데이터를 업데이트 하는 시스템 커맨드마다 사가를 하나씩 정의한다.  
일련의 `로컬 트랜잭션`이다.  


주문 생성 사가를 예제로 설명한다.  
주문 서비스의 createOrder() 작업은 이 사가로 구현된다.  
6개의 로컬 트랜잭션으로 구성된다. 

첫 번째 로컬 트랜잭션은 주문 생성이라는 외부 요청에 의해 시작된다.  
나머지 5개의 로컬 트랜잭션은 각자 자신의 선행 트랜잭션이 완료되면 트리거된다.  

- 주문 서비스: 주문을 APPROVAL_PENDING 상태로 생성
- 소비자 서비스: 주문 가능한 소비자인지 확인
- 주방 서비스: 주문 내역으 확인하고 티켓을 CREATE_PENDING 상태로 생성
- 회계 서비스: 소비자 신용카드를 승인
- 주방 서비스: 티켓 상태를 AWAITING_ACCEPTANCE 로 변경
- 주문 서비스: 주문 상태를 APPROVED 로 변경


서비스는 `로컬 트랜잭션`이 완료되면 `메시지`를 발행하여 다음 사가 단계를 `트리거`한다.  
메시지를 통해 사가 참여자를 느슨하게 결합하고 사가가 반드시 완료되도록 보장해야 한다.  
메시지 수신자가 일시 불능 상태라면, 메시지 브로커는 다시 메시지를 전달할 수 있을 때까지 메시지를 `버퍼링`한다.  
도중 에러가 발생하면 `보상 트랜잭션`으로 변경분을 롤백해야 한다.  


#
#### 보상 트랜잭션
ACID 트랜잭션은 쉽게 `롤백`이 가능하다.  
하지만 사가는 단계마다 로컬 DB에 변경분을 커밋하기 때문에 자동 롤백은 안된다.  
가령 4단계(회계 서비스)에서 실패하면 1~3단계의 변경분을 `명시적`으로 `undo` 해야 한다.  
즉, `보상 트랜잭션(compensating transaction)`을 미리 작성해야 한다.


- 주문 서비스, createOrder(), `rejectOrder()`
- 소비자 서비스: verifyConsumerDetails(), -
- 주방 서비스: createTicket(), `rejectTicket()`
- 회계 서비스: authorizeCreditCard(), -
- 주방 서비스: approveTicket(), -
- 주문 서비스: approveOrder()


- 예를 들어 신용카드 승인이 실패하면 작동 순서는 아래와 같다. 
  - 주문 서비스: `주문`을 `APPROVAL_PENDING 상태`로 생성
  - 소비자 서비스: 소비자가 주문을 할 수 있는지 확인
  - 주방 서비스: 주문 내역을 확인하고 `티켓`을 `CREATE_PENDING 상태`로 생성
  - 회계 서비스: 소비자의 신용카드 승인 요청이 거부됨!!
  - 주방 서비스: `티켓 상태`를 `CREATE_REJECTED` 로 변경
  - 주문 서비스: `주문 상태`를 `REJECTED` 로 변경


이런 일반 트랜잭션과 보상 트랜잭션의 `순서화`는 `사가 편성 로직`이 하면 된다.  


# 
#### 사가 편성
사가 편성은 두 가지 종류가 있다.  
- 코레오크래피(choreography): 순서화를 사가 참여자에게 맡긴다
- `오케스트레이션(orchestration)`: 편성 로직을 중앙화, 오케스트레이터가 커멘드 메시지를 보내 작업을 지시한다


- 코레오그래피 장점
  - 단순함: 변경할 때 서비스가 이벤트를 발행
  - 느슨한 결합: 참여자는 이벤트를 구독할 뿐 서로 알지 못함
- 코레오그래피 단점
  - 이해하기 어려움: 사가가 어느 한곳에 정의된 게 아니여서 로직이 흩어져 있음
  - 서비스 간 순환 의존성: 참여자가 서로 이벤트를 구독하므로 순환 의존성 발생하기 쉬움
  - 단단히 결합될 위험성: 사가 참여자는 각자 자신에게 영향을 미치는 이벤트를 모두 구독하므로 주기에 영향을 받을 수 있음


- 오케스트레이션 사가 장점
  - 의존 관계 단순화: 오케스트레이터가 참여자를 호출하지만 참여자는 오케스트레이터를 호출 하지 않으므로 순환 의존성 없음
  - 낮은 결합도: 각 서비스는 오케스트레이터가 호출하는 API를 구현할 뿐 
  - 관심사를 더 분리하고 비즈니스 로직을 단순화
- 오케스트레이션 사가 단점
  - 비즈니스 로직을 오케스트레이터에 너무 많이 중앙화 하면 부하
  순서화만 담당하고 여타 비즈니스 로직은 갖고 있지 않도록 설계해야 한다 

#

오케스트레이션 사가에서는 사가 참여자가 할 일을 알려주는 `오케스트레이터 클래스`를 정의한다.  
커맨드/비동기 응답 상호 작용을 하며 참여자와 통신한다.  
참여자가 무슨 일을 해야 하는지 커맨드 메시지에 적어 보낸다.  
사가 참여자가 작업을 마친 뒤 응답을 주면, 오케스트레이터는 응답 메시지를 처리 한 후 다음 사가 단계를 어느 참여자가 수행할지 결정한다.  

#
#### 비격리 처리 문제
사가는 격리성이 빠져 있다고 했었다.  
두 가지 문제를 야기한다.  
- 한 사가가 실행 중에 접근하는 데이터를 도중에 다른 사가가 바꿔치기 한다.  
- 한 사가가 업데이트 하기 전에 데이터를 다른 사가가 읽어버릴 수 있다.   


그래서 사가를 동시 실행한 결과와 순차 실행한 결과가 달라질 수 있다.  


- 대책
  - 시맨틱 락(semantic lock)
    - 플래그를 세팅하는 대책
  - 교환적 업데이트(commutative updates)
  - 비관적 관점(pessimistic view)
  - 값 다시 읽기(reread value)
  - 버전 파일(version file)
  - by value



#
#### 주문 서비스 및 주문 생성 사가 설계
저자가 개발한 Eventuate Tram 프레임워크를 사용한 코드를 예제로 보여준다.  

- OrderService
```java
@Transactional
public class OrderService { 
    @Autowired
    private SagaManager<CreateOrderSagaState> createOrderSagaManager;
    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private DomainEventPublisher eventPublisher;
    
    public Order createOrder(OrderDetail orderDetail) {
        ...
        ResultWithEvents<Order> orderAndEvents = Order.createOrder(...); // order생성
        Order order = orderAndEvents.result;
        orderRepository.save(order); // DB에 order 저장
 
        // 도메인 이벤트 발행
        eventPublisher.publish(Order.class,
            Long.toString(order.getId()),
            orderAndEvents.events);
        
        // CreateOrderSaga 생성
        CreateOrderSagaState data = new CreateOrderSagaState(order.getId(), orderDetail);
        createOrderSagaManager.create(data, Order.class, order.getId());
        
        return order;
    }
    ...
}
```

- CreateOrderSaga
```java
public class CreateOrderSaga implements SimpleSaga<CreateOrderSagaState> {
    private SagaDefinition<CreateOrderSagaState> sagaDefinition;

    public CreateOrderSaga(OrderServiceProxy orderService, 
        ConsumerServiceProxy consumerService,
        KitchenServiceProxy kitchenService,
        AccoutingServiceProxy accountingService) {
        this.sagaDefinition = 
            step()
            .withCompensation(orderService.reject, CreateOrderSagaState::makeRejectOrderCommand)
            .step()
            .invokeParticipant(consumerService.validateOrder, CreateOrderSagaState::makeValidateOrderByConsumerCommand)
            .step()
            .invokeParticipant(kitchenService.create, CreateOrderSagaState::makeCreateTicketCommand)
            .onReply(CreateTicketReply.class, CreateOrderSagaState::handleCreateTicketReply)
            .withCompensation(kitchenService.cancel, CreateOrderSagaState::makeCancelCreateTicketCommand)
            .step()
            .invokeParticipant(accountingService.authorize, CreateOrderSagaState::makeAuthorizeCommand)
            .step()
            .invokeParticipant(kitchenService.confirmCreate, CreateOrderSagaState::makeConfirmCreateTicketCommand)
            .step()
            .invokeParticipant(orderService.approve, CreateOrderSagaState::makeApproveOrderCommand)
            .build();
    }
    
    @Override
    public SagaDefinition<CreateOrderSagaState> getSagaDefinition() {
        return sagaDefinition;
    }
}
```

- CreateOrderSagaState 
```java
public class CreateOrderSagaState {
    private Long orderId;
    private OrderDetails orderDetails;
    private long ticketId;
    
    public Long getOrderId() {
        return orderId;
    }

    private CreateOrderSagaState() {}

    public CreateOrderSagaState(Long orderId, OrderDetails orderDetails) {
        this.orderId = orderId;
        this.orderDetails = orderDetails;
    }

    CreateTicket makeCreateTicketCommand() {
        return new CreateTicket(getOrderDetails().getRestaurantId(),
            getOrderId(), makeTicketDetails(getOrderDetails())); // 커맨드 메시지 생성
    }

    void handleCreateTicketReply(CreateTicketReply reply) {
        logger.debug("getTicketId {}", reply.getTicketId());
        setTicketId(reply.getTicketId());   // 새로 만든 티켓 아이디 저장
    }

    CancelCreateTicket makeCancelCreateTicketCommand() {
        return new CancelCreateTicket(getOrderId()); // 커맨드 메시지 생성
    }
}   
```

- KitchenServiceProxy
```java
public class KitchenServiceProxy {
    public final CommandEndPoint<CreateTicket> create = 
        CommandEndpointBuilder
            .forCommand(CreateTicket.class)
            .withChannel(KitchenServiceChannels.kitchenServiceChannel)
            .withReply(CreateTicketReply.class)
            .build();

    public final CommandEndPoint<ConfirmCreateTicket> confirmCreate = 
        CommandEndpointBuilder
            .forCommand(ConfirmCreateTicket.class)
            .withChannel(KitchenServiceChannels.kitchenServiceChannel)
            .withReply(Success.class)
            .build();

    public final CommandEndPoint<CancelCreateTicket> confirmCreate = 
            CommandEndpointBuilder
                .forCommand(CancelCreateTicket.class)
                .withChannel(KitchenServiceChannels.kitchenServiceChannel)
                .withReply(Success.class)
                .build();    
}
```   

    


