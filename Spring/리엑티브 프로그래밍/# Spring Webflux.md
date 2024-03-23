# Spring Webflux

- Spring Webflux는 Reactor 기반의 리엑티브 프로그래밍을 지원한다.
- 기본적으로 Reactor(Publisher, Subscirber) 기반으로 동작한다.
- Webflux에서는 Publisher를 간단하게 작성할 수 있는 Mono, Flux를 제공한다.

## Webflux의 장점, 단점

- 기본적으로 Spring MVC는 request 당 thread를 할당한다.
- 즉, 요청이 들어오는 수 만큼 스레드가 할당되기 때문에 요청이 굉장히 많을 경우 대기 시간이 길어진다.
  - 예를 들어 tomcat의 thread 최대 개수인 200개 이상의 요청이 들어오고 이때 해당 요청 처리 시간이 길어질 경우 그 이후의 요청은 블록되어 실행되지 않는다.
- Spring Webflux는 기본적으로 non-blocking 방식을 지원한다.
- 즉, 요청 당 스레드 할당이 아니라 요청을 처리하는 스레드가 있고 실제 로직을 처리하는 스레드가 따로 있으며 로직은 이벤트 단위로 동작한다.
- Webflux에서는 이벤트 단위로 동작하며 요청을 처리하는 스레드가 이벤트 루프를 돌면서 처리가능한지 확인한 후 가능하면 처리하는 방식으로 동작한다.
- 즉 처음 요청을 받는 스레드와 실제 로직을 처리하는 스레드, 그리고 응답을 반환하는 스레드 모두 달라질 수 있다는 것이다.
  - 이벤트가 나뉘기 떄문에 처리하는 스레드가 달라진다.
- 위 예시처럼 요청이 200개 보다 많이 들어오더라도, 요청 처리 시간이 길어지더라도 요청을 받는 스레드와 실제 이벤트 처리 스레드가 다르기 떄문에 요청은 받을 수 있게 된다.
- 다만 주의할 점은 webflux는 non-blocking 방식이기 떄문에 blocking 로직이 들어올 경우 오히려 성능이 떨어질 수 있다.
  - jdbc와 같이 blocking-io의 경우에는 해당 스레드를 계속해서 점유하고 있기 떄문에 성능이 떨어진다.
- non-blocking 방식에서는 블록킹 되는 순간에 해당 스레드를 점유하고 있는 것이 아니라, 반환하여 다른 작업을 수행한다.
  - 예를 들어, i/o 작업이나 network 통신할 때 커널단에서 처리하도록 한 후 스레드는 다른 작업을 수행한다.
- 이를 통해 더 적은 스레드로 더 많은 처리를 할 수 있게 된다. 

## Controller

- webflux에서 controller는 Spring MVC와 다르게 Mono, Flux를 리턴한다.
- Mono와 Flux는 Publisher의 구현체이기 때문에 당연하게도 subscribe 메소드가 실행되어야 로직이 실행된다.
- Mono의 Flux의 subscribe는 스프링이 실행해준다.
- 즉, 간단하게 보자면 개발자는 비즈니스 로직만 작성하고 해당 로직 실행과 결과는 스프링에게 맡기는 것이다.

## Netty

- Spring MVC는 기본적으로 Tomcat을 기반으로 동작한다.
  - 기본적으로 요청과 응답은 Tomcat(Was)에서 처리하며 실제 비즈니스 로직은 Spring Container에서 처리한다.
  - 정확하게는 Tomcat의 Servlet container가 요청을 담당하고, 해당 Tomcat이 Spring container에게 요청하는 방식이다.
    - Servlet container란 Servlet을 담당하는 역할을 하며, Servlet이란 요청을 담당하는 부분을 의미한다.
  - Spring MVC는 요청 당 스레드를 할당하는데, 이때 스레드를 할당하는 부분이 Tomcat이다.
  - request -> Tomcat 스레드 할당 -> DispatcherServlet에서 적절한 Servlet 찾음(Handler Mapping) -> 요청 처리
  - servlet --> 하나의 Http 요청 처리
- Spring Webflux는 Tomcat이 아니라 Netty가 기본이다.
  - 설정을 통해 Tomcat으로 동작하도록 할 수 있다.
- Spring MVC 처럼 요청에 대한 부분은 Netty가 담당하고, 개발자는 비즈니스 로직에 집중하도록 한다.
- Netty는 Tomcat처럼 Servlet을 통한 구현이 아니라, Selector를 통해 이벤트 중심으로 요청을 처리한다.
  - 즉, Netty에서도 Tomcat 처럼 웹 소켓에 대한 부분은 대신 처리해준다.
- 가장 큰 차이점으로는 Netty에서는 Tomcat처럼 요청 당 스레드를 생성하는 것이 아니라 이벤트 루프를 통해 요청을 처리한다는 것이다.
  - 요청이 들어오면 이벤트 큐에 등록되고, 이벤트 루프가 이벤트 큐에서 이벤트를 가져와 처리하는 것이다.
- 또한 이벤트 처리는 Tomcat처럼 200개의 스레드가 아니라 core*2개의 스레드만 사용한다.
  - 해당 수는 core 당 최적화된 스레드 개수이며, 이벤트 루프 + non-blocking 작업을 통해 스레드가 유휴 시간 없이 동작할 수 있게 한다.
- Webflux는 기본적으로 컨트롤러에서 Mono 또는 Flux를 반환하며, 실제 해당 Publisher의 실행 부분(subscibe)는 Netty에서 진행된다.
- 그렇기 때문에 실행 로직에서 Blocking 동작이 있다면 몇개 없는 스레드를 오래 점유하기 때문에 성능이 떨어진다.

