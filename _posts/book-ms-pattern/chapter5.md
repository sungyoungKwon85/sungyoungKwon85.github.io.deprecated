#

## 5. 비즈니스 로직 설계

`비즈니스 로직`이 여러 서비스에 흩어져 있는 마이크로서비스 아키텍처의 문제 

- 도메인 모델은 대부분 상호 연관된 클래스가 거미줄처럼 엮여 있음
- 마이크로서비스 아키텍쳐 트규의 트랜잭션 관리 제약 조건하에서도 작동되도록 로직 설계 필요

이 두 문제는 여러 애그리거트로 구성하는 `DDD 애그리거트 패턴`으로 해결 가능하다. 

- 애그리거트를 사용하면 객체 레퍼런스가 `서비스 경계`를 넘나들 일이 없다
  객체 참조 대신 `기본 키`를 이용하여 서로` 참조`함
- `한 트랜잭션`으로 `하나의 애그리거트`만 생성/수정 할 수 있다. 따라서 애그리거트는 마이크로서비스 트랜잭션 모델의 제약조건에 잘 맞는다

따라서 ACID 트랜잭션은 반드시 하나의 서비스 내부에만 걸리게 된다.

---

### 5.1 비즈니스 로직 구성 패턴

주문 서비스는 비즈니스 로직과 인바운드/아웃바운드 어댑터가 주변을 감싼 `육각형 아키텍쳐` 구조다.    

- `REST API Adapter`
- Order`CommandHandlers`: 메시지 채널에서 들어온 커맨드 메시지를 받아 비즈니스 로직을 호출하는 인바운드 어댑터
- `DB Adapter`
- `Domain event publish Adapter`: 이벤트를 메시지 브로커에 발행하는 아웃바운드 어댑터

마이크로 서비스 아키텍처에서는 OOD만으로는 해결해야할 문제가 많아 이를 개선한 DDD가 필요하다.  

다음은 DDD에서 도메인 모델을 구축하는데 흔히 쓰이는 빌딩 블록이다.  

- entity
- value object: 여러 값을 모아놓은 객체. 
- factory
- repository
- service

이 외에 그동안 잘 고려하지 않았던 `aggregate`가 있다.  

aggregate는 마이크로서비스 개발에 정말 유용한 개념이다.  

---

### 5.2 DDD aggregate 패턴

전통적인 객체 지향 설계에 기반한 도메인 모델은 클래스와 클래스 간 관계를 모아 놓는다.  

클래스는 보통 패키지로 구성된다.  

Consumer, Order, Restaurant, Courier 등 비즈니스 객체와 PaymentInfo, DeliveryInfo, OrderLineItem, MenuItem, Address, Location 등 비즈니스 객체와 대응되는 클래스가 있다.  

그런데 이 도메인 모델은 비즈니스 객체들의` 경계`가 불분명하다. 

이를테면 어느 클래스가 Order라는 비즈니스 객체의 일부인지 분명하지 않다.  



---

Aggregate는 한 단위로 취급 가능한 경계 내부의 도메인 객체들이다.  

하나의 루트 엔티티와 하나 이상의 기타 엔티티 + 밸류 객체로 구성된다.  

주문, 소비자, 음식점 같은 것들이 바로` aggregate`이다.  

- 주문: Order(aggregate root), DeliveryInfo(value obejct), PaymentInfo(value object), OrderLineItem(value object)
- 소비자: Consumer(root), DeliveryInfo(value), PaymentInfo(value)
- 음식점: ...



---

Aggregate는 몇가지 지켜야 할 규칙이 있다. 

- `Aggregate root`만 참조하라
  - 예를 들어 클라이언트가 직접 품목 수량을 수정할 수  없고 주문 애그리거트 루트에 있는 메서드를 호출하도록 해 최소 주문량 같은 불변 값(주문 규칙)이 강제되도록 한다
- Aggregate 간 참조는 반드시 `기본키`를 사용하라
  - FK를 사용하면 애그리거트는 느슨하게 결합되고 서로의 경계가 분명해진다
  - 애그리거트 자체가 저장 단위이므로 저장 로직도 간단해진다
- `하나의 트랜잭션`으로 오직 `하나의 Aggregate`를 수정하라
  - `여러 애그리거트`를 수정/생성하는 작업은 `saga`로 해결
  - 단순 트랜잭션만 지원하는 `NoSQL` 사용하기에도 좋다



---

### 5.3 도메인 이벤트 발행

비즈니스 로직은 Order aggreate, OrderService, OrderRepository, 하나 이상의 사가들로 구성된다. 

OrderService는 OrderRepository를 이용하여 Order를 조회/저장한다. 

간단한 요청은 Order aggregate를 직접 업데이트하고, 여러 서비스에 걸친 업데이트 요청은 사가를 생성해서 처리한다. 



DDD 맥락에서 `도메인 이벤트`는`애그리거트`에 `발생한 사건`이다. 

클래스로 표현되며 대부분 어떤 `상태 변경`을 나타낸다. 

가령 Order 애그리거트라면 `주문 생성됨`, `주문 취소됨`, `주문 배달됨` 등 `상태`가 바뀌는 `이벤트`가 `발생`한다. 

상태가 전이 될 때마다 이에 관련된 컨슈머를 위해 `이벤트를 발행`한다. 

---

#### 변경 이벤트를 발행하는 이유

다른 구성원(사용자, 다른 애블리케이션, 내부의 다른 컴포넌트)들이 애그리거트의 상태 변경을 궁금해 하기 때문에 도메인 이벤트는 유용하다. 

- `사가`를 이용하여 여러 서비스에 걸쳐 데이터 일관성을 유지
- `레플리카`를 둔 서비스에 소스 데이터가 변경됨을 알림(CQRS)
- `webhook`, `메시지 브로커`를 통해 비즈니스 프로세스의 다음 단계를 진행하도록 다른 애플리케이션에 알림
- 사용자 브라우저에 `웹 소켓` 메시지를 보내거나, `엘라스틱서치` 같은 텍스트 DB를 업데이트 하기 위해 다른 컴포넌트에 알림
- 사용자에게 (텍스트 메시지, 이메일) 알림
- 도메인 이벤트를 `모니터링`
- 사용자 행동 파악을 위한 이벤트 `분석`

---

#### 도메인 이벤트 식별

요즘은 `이벤트 스토밍` 방법을 많이 사용하는 추세다. 

- 이벤트 브레인 스토밍: 포스트지에 모든 이벤트를 적어서 타임라인에 배치
- 이벤트 트리거 식별
  - 사용자 액션
  - 외부 시스템
  - 기타 도메인 이벤트
  - 시간 경과
- 애그리거트 식별

---

도메인 이벤트는 애그리거트가 발행한다

직접 발행하면 인프라 관심사와 비즈니스 로직이 뒤엉키므로 책임을 분리하는게 좋다

```java
public class Ticket {
  public List<TicketDomainEvent> accept(LocalDateTime readyBy) {
    ...
    this.acceptTime = LocalDateTime.now(); // ticket update
    this.readyBy = readyBy;
    return singletonList(new TicketAcceptedEvent(readyBy)); // 이벤트 반환
  }
}

public class KitchenService {
  @Autowired
  private TicketRepository ticketRepository;
  
  @Autowired
  private TicketDomainEventPublisher domainEventPublisher;
  
  public void accept(long ticketId, LocalDateTime readyBy) {
    Ticket ticket = ticketRepository.findById(ticketId)
            .orElseThrow(() -> new TicketNotFoundException(ticketId));
    List<TicketDomainEvent> events = ticket.accept(readyBy);
    domainEventPublisher.publish(ticket, events); // 도메인 이벤트 발행
  }
```



서비스는 DB에서 애그리거트를 업데이트하는 트랜잭션의 일부로 이벤트를 발행하기 위해 `트랜잭셔널 메시징`을 사용해야 한다. 

`Eventuate Tram` 프레임워크는 이런 매커니즘이 구현되어 있다.

`AbstractAggregateDomainEventPublisher`는 `타입-안전`한 `도메인 이벤트 발행`용 인터페이스이다. 

아래는 `Ticket` 애그리거트의 도메인 이벤트를 발행하는 `TicketDomainEventPublishe` 클래스이다.  

정의에 따라 `TicketDomainEvent`의 하위 글래스에 해당하는 이벤트만 발행한다. 

```java
import io.eventuate.tram.events.aggregates.AbstractAggregateDomainEventPublisher;
import io.eventuate.tram.events.publisher.DomainEventPublisher;

public class TicketDomainEventPublisher extends AbstractAggregateDomainEventPublisher<Ticket, TicketDomainEvent> {

  public TicketDomainEventPublisher(DomainEventPublisher eventPublisher) {
    super(eventPublisher, Ticket.class, Ticket::getId);
  }

}
```



---

#### 도메인 이벤트 소비

`도메인 이벤트`는 결국 `메시지`로 바뀌어 `메시지 브로커`에 발행된다. 

`Eventuate Tram` 프레임워크가 제공하는 고수준 API를 써서 `도메인 이벤트`를 적절한 `핸들러 메서드`로 `디스패치`하는 것이 간편하다

아래는 음식점 메뉴가 갱신될 때마다 `음식점 서비스`가 `발행`하는 `이벤트`를 `구독`하는 `컨슈머`이다.

```java
import io.eventuate.tram.events.subscriber.DomainEventEnvelope;
import io.eventuate.tram.events.subscriber.DomainEventHandlers;
import io.eventuate.tram.events.subscriber.DomainEventHandlersBuilder;

public class KitchenServiceEventConsumer {

  @Autowired
  private KitchenService kitchenService;

  // 이벤트와 이벤트 핸들러를 매핑
  public DomainEventHandlers domainEventHandlers() {
    return DomainEventHandlersBuilder
            .forAggregateType("net.chrisrichardson.ftgo.restaurantservice.domain.Restaurant")
            .onEvent(RestaurantMenuRevised.class, this::reviseMenu)
            .build();
  }
  
  // RestaurantMenuRevised 이벤트 핸들러
  public void reviseMenu(DomainEventEnvelope<RestaurantMenuRevised> de) {
    long id = Long.parseLong(de.getAggregateId());
    RestaurantMenu revisedMenu = de.getEvent().getRevisedMenu();
    kitchenService.reviseMenu(id, revisedMenu);
  }
}
```



---



### 5.4 주방 서비스 비즈니스 로직

`주방 서비스`는 `음식점`이` 주문`을` 관리`할 수 있게 해주는 서비스이다. 

`Restaurant` 애그리거트와 `Ticket `애그리거트가 이 서비스의 `메인 애그리거트`이다. 

Restaurant 애그리거트는 음식점 메뉴 및 운영 시간을 알고 있는 상태에서 주문을 검증할 수 있다. 

티켓을 배달원이 픽업할 수 있게 음식점이 미리 준비해야 할 주문이다. 

그 외에도 `KitchenService`,` TicketRepository`, `RestaurantRepository` 등이 있다. 

`인바운드 어댑터`는 3개가 있다

- `REST API`
- `KitchenServiceCommandHandler`: 사가가 호출하는 비동기 요청/응답 API, KitchenService를 호출해 Ticket을 생성
- `KitchenServiceEventConsumer`: RestaurantService가 발행한 이벤트를 구독, KitchenService를 호출해 Restaurant를 생성/수정한다

`아웃바운드 어댑터`는 2개

- `DB Adapter`: TicketRepository, RestaurantRepository 인터페이스를 구현해 DB 접근
- `DomainEventPublishingAdapter`: DomainEventPublisher 인터페이스를 구현해 Ticket 도메인 이벤트를 발행

---

`Ticket` 애그리거트는 `음식점 주방` `관점`에서 바라본 `주문`을 나타낸 것이다. 

신원, 배달 정보, 지불 내역 등 소비자 관련 정보가 없고, 오직 `음식점 주방`이 배달원이 픽업할 주문을 준비하는데만 `집중`한다. 

`다른 애그리거트`를 `기본키`로 `참조`한다. 

```java
@Entity
@Table(name = "tickets")
@Access(AccessType.FIELD)
public class Ticket {

  @Id
  private Long id;

  @Enumerated(EnumType.STRING)
  private TicketState state;

  private Long restaurantId;

  @ElementCollection
  @CollectionTable(name = "ticket_line_items")
  private List<TicketLineItem> lineItems;

  private LocalDateTime readyBy;
  private LocalDateTime acceptTime;
  private LocalDateTime preparingTime;
  private LocalDateTime pickedUpTime;
  private LocalDateTime readyForPickupTime;
  ...
```

restaurantId는 Restaurant 객체를 가리키는 레퍼런스가 아닌 Long형 필드이다. 

---

Ticket 애그리거트는 Ticket을 생성하는 정적 팩토리 메서드 create() 등 음식점이 주문 상태를 업데이트 하기 위한 메서드를 가진다.

```java
public class Ticket {
  public static ResultWithDomainEvents<Ticket, TicketDomainEvent> create(long restaurantId, Long id, TicketDetails details) {
    return new ResultWithDomainEvents<>(new Ticket(restaurantId, id, details));
  }
  
  public List<TicketDomainEvent> accept(LocalDateTime readyBy) {
    switch (state) {
      case AWAITING_ACCEPTANCE:
        // Verify that readyBy is in the futurestate = TicketState.ACCEPTED;
        this.acceptTime = LocalDateTime.now();
        if (!acceptTime.isBefore(readyBy))
          throw new IllegalArgumentException("readyBy is not in the future");
        this.readyBy = readyBy;
        return singletonList(new TicketAcceptedEvent(readyBy));
      default:
        throw new UnsupportedStateTransitionException(state);
    }
  }
  
  public List<TicketDomainEvent> preparing() {
    switch (state) {
      case ACCEPTED:
        this.state = TicketState.PREPARING;
        this.preparingTime = LocalDateTime.now();
        return singletonList(new TicketPreparationStartedEvent());
      default:
        throw new UnsupportedStateTransitionException(state);
    }
  }
  
  public List<TicketDomainEvent> readyForPickup() {
    switch (state) {
      case PREPARING:
        this.state = TicketState.READY_FOR_PICKUP;
        this.readyForPickupTime = LocalDateTime.now();
        return singletonList(new TicketPreparationCompletedEvent());
      default:
        throw new UnsupportedStateTransitionException(state);
    }
  }
  
  public List<TicketDomainEvent> cancel() {
    switch (state) {
      case AWAITING_ACCEPTANCE:
      case ACCEPTED:
        this.previousState = state;
        this.state = TicketState.CANCEL_PENDING;
        return emptyList();
      default:
        throw new UnsupportedStateTransitionException(state);
    }
  }
  
```

---

KitchenService는 주방 서비스의 인바운드 어댑터가 호출한다. 

```java
public class KitchenService {
  @Autowired
  private TicketRepository ticketRepository;

  @Autowired
  private TicketDomainEventPublisher domainEventPublisher;

  public void accept(long ticketId, LocalDateTime readyBy) {
    Ticket ticket = ticketRepository.findById(ticketId)
            .orElseThrow(() -> new TicketNotFoundException(ticketId));
    List<TicketDomainEvent> events = ticket.accept(readyBy);
    domainEventPublisher.publish(ticket, events); // 도메인 이벤트 발행
  }
```

---

KitchenServiceCommandHandler 클래스는 주문 서비스에 구현된 사가가 전송한 커맨드 메시지를 처리하는 어댑터이다. 

```java
public class KitchenServiceCommandHandler {

  @Autowired
  private KitchenService kitchenService;

  public CommandHandlers commandHandlers() {
    return SagaCommandHandlersBuilder
            .fromChannel(KitchenServiceChannels.kitchenServiceChannel)
            .onMessage(CreateTicket.class, this::createTicket)
            .onMessage(ConfirmCreateTicket.class, this::confirmCreateTicket)
            .onMessage(CancelCreateTicket.class, this::cancelCreateTicket)
            .build();
  }

  private Message createTicket(CommandMessage<CreateTicket>
                                                cm) {
    CreateTicket command = cm.getCommand();
    long restaurantId = command.getRestaurantId();
    Long ticketId = command.getOrderId();
    TicketDetails ticketDetails = command.getTicketDetails();


    try {
      Ticket ticket = kitchenService.createTicket(restaurantId, ticketId, ticketDetails);
      CreateTicketReply reply = new CreateTicketReply(ticket.getId());
      return withLock(Ticket.class, ticket.getId()).withSuccess(reply);
    } catch (RestaurantDetailsVerificationException e) {
      return withFailure();
    }
  }

  private Message confirmCreateTicket
          (CommandMessage<ConfirmCreateTicket> cm) {
    Long ticketId = cm.getCommand().getTicketId();
    kitchenService.confirmCreateTicket(ticketId);
    return withSuccess();
  }
  ...
```

---

### 5.5 주문 서비스 비즈니스 로직

지금까지는 비교적 간단한 서비스 비즈니스 로직이었고 복잡한 주문 서비스를 본다

`주문 서비스`는 주문을 생성, 수정, 취소하는 API를 제공하는 서비스이다. 

이런 API는 컨슈머가 주로 호출한다. 

비즈니스 로직은 `Order/Restaurant 애그리거트` 뿐만 아니라 `OrderService, OrderRepository, RestaurantRepository`, `CreateOrderSaga` 같은 `여러 사가`로 구성된다. 

`인바운드 어댑터`는 아래

- REST API: 컨슈머가 사용하는 UI가 호출하는 REST API, OrderService를 호출해 Order를 수정/생성한다
- OrderEventConsumer: 음식점 서비스가 발행한 이벤트를 구독. OrderService를 호출해 Restaurant 레플리카를 수정
- OrderCommandHandler: 사가가 호출하는 비동기 요청/응답 기반의 API, OrderService를 호출해 Order를 수정
- SagaReplyAdapter: 사가 응답 채널을 구독하고 사가를 호출

`아웃바운드 어댑터`

- DB Adapter
- DomainEventPublishingAdapter: Order 도메인 이벤트를 발행
- OutboundCommandMessageAdapter: CommandPublisher 인터페이스를 구현해 커멘드 메시지를 사가 참여자에게 보냄

---

##### Order 애그리거트

Order가 애그리거트 루트이고 OrderLineItem, DeliveryInfo, PaymentInfo 등 여러 밸류 객체가 있다

```java
@Entity
@Table(name = "orders")
@Access(AccessType.FIELD)
public class Order {
  @Id
  @GeneratedValue
  private Long id;

  @Version
  private Long version;

  @Enumerated(EnumType.STRING)
  private OrderState state;

  private Long consumerId;
  private Long restaurantId;

  @Embedded
  private OrderLineItems orderLineItems;

  @Embedded
  private DeliveryInformation deliveryInformation;

  @Embedded
  private PaymentInformation paymentInformation;

  @Embedded
  private Money orderMinimum = new Money(Integer.MAX_VALUE);
```

---

##### 상태 기계

주문을 생성/수정하려면 OrderService는 반드시` 다른 서비스`와 `사가`로 `협동`해야 한다

Order 메서드를 호출해 수행 가능한 작업인지 확인한 후 해당 `주문`을 `APPROVAL_PENDING` `상태로 변경`한다(`시맨틱 락`)

아래는 주문 생성 도중 호출되는 메서드들이다. 

```java
public class Order {
  public static ResultWithDomainEvents<Order, OrderDomainEvent>
  createOrder(long consumerId, Restaurant restaurant, List<OrderLineItem> orderLineItems) {
    Order order = new Order(consumerId, restaurant.getId(), orderLineItems);
    List<OrderDomainEvent> events = singletonList(new OrderCreatedEvent(
            new OrderDetails(consumerId, restaurant.getId(), orderLineItems,
                    order.getOrderTotal()),
            restaurant.getName()));
    return new ResultWithDomainEvents<>(order, events);
  }
  
  public Order(long consumerId, long restaurantId, List<OrderLineItem> orderLineItems) {
    this.consumerId = consumerId;
    this.restaurantId = restaurantId;
    this.orderLineItems = new OrderLineItems(orderLineItems);
    this.state = APPROVAL_PENDING;
  }
  ...
  public List<OrderDomainEvent> noteApproved() {
    switch (state) {
      case APPROVAL_PENDING:
        this.state = APPROVED;
        return singletonList(new OrderAuthorized());
      default:
        throw new UnsupportedStateTransitionException(state);
    }

  }
  
  public List<OrderDomainEvent> noteRejected() {
    switch (state) {
      case APPROVAL_PENDING:
        this.state = REJECTED;
        return singletonList(new OrderRejected());

      default:
        throw new UnsupportedStateTransitionException(state);
    }

  }
```



아래는 주문 변경 메서드들이다

```java
public class Order {
  
  public ResultWithDomainEvents<LineItemQuantityChange, OrderDomainEvent> revise(OrderRevision orderRevision) {
    switch (state) {

      case APPROVED:
        LineItemQuantityChange change = orderLineItems.lineItemQuantityChange(orderRevision);
        if (change.newOrderTotal.isGreaterThanOrEqual(orderMinimum)) {
          throw new OrderMinimumNotMetException();
        }
        this.state = REVISION_PENDING;
        return new ResultWithDomainEvents<>(change, singletonList(new OrderRevisionProposed(orderRevision, change.currentOrderTotal, change.newOrderTotal)));

      default:
        throw new UnsupportedStateTransitionException(state);
    }
  }
  
  public List<OrderDomainEvent> confirmRevision(OrderRevision orderRevision) {
    switch (state) {
      case REVISION_PENDING:
        LineItemQuantityChange licd = orderLineItems.lineItemQuantityChange(orderRevision);

        orderRevision.getDeliveryInformation().ifPresent(newDi -> this.deliveryInformation = newDi);

        if (!orderRevision.getRevisedLineItemQuantities().isEmpty()) {
          orderLineItems.updateLineItems(orderRevision);
        }

        this.state = APPROVED;
        return singletonList(new OrderRevised(orderRevision, licd.currentOrderTotal, licd.newOrderTotal));
      default:
        throw new UnsupportedStateTransitionException(state);
    }
  }
```



---

##### OrderService

비즈니스 로직의 진입점이다

주문을 생성/수정하는 메서드가 모두 여기 있다

이 클래스의 메서드는 대부분 `사가`를 만들어 Order 애그리거트 생성/수정을 `오케스트레이션` 한다

따라서 디펜던시가 많다

```java
@Transactional
public class OrderService {
  @Autowired
  private OrderRepository orderRepository;
	@Autowired
  private RestaurantRepository restaurantRepository;
	@Autowired
  private SagaManager<CreateOrderSagaState> createOrderSagaManager;
	@Autowired
  private SagaManager<CancelOrderSagaData> cancelOrderSagaManager;
	@Autowired
  private SagaManager<ReviseOrderSagaData> reviseOrderSagaManager;
	@Autowired
  private OrderDomainEventPublisher orderAggregateEventPublisher;
  
  public Order createOrder(long consumerId, long restaurantId,
                           List<MenuItemIdAndQuantity> lineItems) {
    Restaurant restaurant = restaurantRepository.findById(restaurantId)
            .orElseThrow(() -> new RestaurantNotFoundException(restaurantId));

    List<OrderLineItem> orderLineItems = makeOrderLineItems(lineItems, restaurant); // Order 애그리거트 조회

    ResultWithDomainEvents<Order, OrderDomainEvent> orderAndEvents =
            Order.createOrder(consumerId, restaurant, orderLineItems);

    Order order = orderAndEvents.result;
    orderRepository.save(order); // order를 DB에 저장

    orderAggregateEventPublisher.publish(order, orderAndEvents.events); // 도메인 이벤트 발행

    OrderDetails orderDetails = new OrderDetails(consumerId, restaurantId, orderLineItems, order.getOrderTotal());

    CreateOrderSagaState data = new CreateOrderSagaState(order.getId(), orderDetails);
    createOrderSagaManager.create(data, Order.class, order.getId()); // CreateOrderSaga 생성

    meterRegistry.ifPresent(mr -> mr.counter("placed_orders").increment());

    return order;
  }
  
  public Order reviseOrder(long orderId, OrderRevision orderRevision) {
    Order order = orderRepository.findById(orderId).orElseThrow(() -> new OrderNotFoundException(orderId)); // Order 조회
    ReviseOrderSagaData sagaData = new ReviseOrderSagaData(order.getConsumerId(), orderId, null, orderRevision);
    reviseOrderSagaManager.create(sagaData); // ReviseOrderSaga 생성
    return order;
  }
```

---



이렇게 보면 모놀로식과 아주 달라 보이지 않는다

모놀로식도 서비스와 JPA 기반 엔티티, 레포지터리 등으로 구성된다. 



다만 설계 제약 조건이 부과된 DDD 애그리거트 도메인 모델을 구성하고, 상이한 애그리거트의 클래스는 객체 레퍼런스가 아닌, 기본키 값으로 참조하는 차이점이 있다. 

또한 트랜잭션은 꼭 하나의 애그리거트만 생성/수정할 수 있으므로 애그리거트 상태 변경 시 도메인 이벤트를 발행해야 한다. 

또한 사가를 이용해서 여러 서비스에 걸쳐 데이터 일관성을 유지해야 한다. 



주방 서비스는 사가에 참여할 뿐 사가를 시작하지 않았지만, 주문 서비스는 주문을 생성하고 수정할 때 사가에 전적으로 의존한다. 

그래서 OrderService 메서드는 대부분 직접 Order를 업데이트하지 않고 사가를 만든다. 



