# [IntelliJ로 JSP 프로젝트 생성하기](https://velog.io/@ruddms936/IntelliJ%EB%A1%9C-JSP-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%83%9D%EC%84%B1#%ED%95%84%EC%9A%94%EC%82%AC%ED%95%AD)



- 프로젝트 생성
- Java 모듈
- 프레임워크 지정 안함
- 템플릿 지정안함
- --- 생성됨



- 상위 폴더에서 우클릭
- 프레임워크 지원추가 클릭
- Java EE에서 웹 서비스 체크
- Generate sample server code 체크 해제
- Version > Apache Axis
- Libraries > Set up library later 선택
- --- 확인



- server-config.wsdd 삭제
- web.xml 에서 <web-app> 부분 제외하고 삭제



- 톰캣 설정
- 오른쪽 상단의 구성 추가 클릭
- '+' 클릭
- tomcat server > local 클릭
- 톰캣 하나 다운받아서 그 경로 지정하기
- VM Option에 -Dfile.encoding=UTF-8 추가
- Deployment tab
  - '+' artifact 추가
  - application context를 '/' 로 변경



- 서블릿 생성
- src에서 java 생성 (Servlet.java)
- extends javax.servlet.http.HttpServlet
- 우클릭해서 디펜던시를 추가하자
- Servlet.java 삭제하고 새로 생성(단, 서블릿으로 생성할 것)



끝



