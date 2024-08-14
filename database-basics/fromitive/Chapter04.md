
### 데이터 이상

* 삽입 이상
* 삭제 이상
* 갱신 이상

### 정규화

* 1 정규화
* 2 정규화 A->B / B->C 이면 (AB, BC)로 테이블을 분리해야 한다.
* 3 정규화 학생 강의, 강의  수강료 이면


### 트렌젝션

속성

1. 원자성 - 모두가 반영되거나 반영되지 않거나
2. 일관성 -  트렌젝션을 하기 전과 후의 데이터가 정합성이 맞아야 한다.
4. 격리성 - 다른 트렌젝션과 격리가 되어야 한다.
5. 영속성 - 종속이 되면 다시 참조 할 수 있어야 함


DeadLock
- 발생조건 (4가지)

Banker's Algorirthm 오버헤드 많대요
잘 조절해야한다?


```
        me1_0.id=?
2024-08-08T07:07:51.763Z ERROR 1 --- [chongdae] [nio-8080-exec-8] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed: java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "this.originPrice" is null] with root cause

java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because "this.originPrice" is null
	at com.zzang.chongdae.offering.domain.OfferingPrice.validateOriginPrice(OfferingPrice.java:30) ~[!/:0.0.1-SNAPSHOT]
	at com.zzang.chongdae.offering.service.OfferingService.saveOffering(OfferingService.java:114) ~[!/:0.0.1-SNAPSHOT]
	at jdk.internal.reflect.GeneratedMethodAccessor88.invoke(Unknown Source) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:569) ~[na:na]
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:354) ~[spring-aop-6.1.10.jar!/:6.1.10]
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:716) ~[spring-aop-6.1.10.jar!/:6.1.10]
	at com.zzang.chongdae.offering.service.OfferingService$$SpringCGLIB$$0.saveOffering(<generated>) ~[!/:0.0.1-SNAPSHOT]
	at com.zzang.chongdae.offering.controller.OfferingController.saveOffering(OfferingController.java:79) ~[!/:0.0.1-SNAPSHOT]
	at jdk.internal.reflect.GeneratedMethodAccessor87.invoke(Unknown Source) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
	at java.base/java.lang.reflect.Method.invoke(Method.java:569) ~[na:na]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:255) ~[spring-web-6.1.10.jar!/:6.1.10]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:188) ~[spring-web-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:118) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:926) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:831) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1089) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:979) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1014) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:914) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at jakarta.servlet.http.HttpServlet.service(HttpServlet.java:590) ~[tomcat-embed-core-10.1.25.jar!/:na]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:885) ~[spring-webmvc-6.1.10.jar!/:6.1.10]
	at jakarta.servlet.http.HttpServlet.service(HttpServlet.java:658) ~[tomcat-embed-core-10.1.25.jar!/:na]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:195) ~[tomcat-embed-core-10.1.25.jar!/:na]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140) ~[tomcat-embed-core-10.1.25.jar!/:na]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51) ~[tomcat-embed-websocket-10.1.25.jar!/:na]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164) ~[tomcat-embed-core-10.1.25.jar!/:na]
	at org.apache.catalina.core.A

```

Time