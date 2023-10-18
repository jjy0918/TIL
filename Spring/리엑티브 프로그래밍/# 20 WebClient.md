# 20 WebClient

## 20.1 WebClient란?

- WebClient는 Spring5 부터 지원하는 Non-Blocking HTTP request를 위한 리액티브 웹 클라이언트다.
- WebClient는 내부적으로 HTTP 클라이언트 라이브러리에게 HTTP request를 위임한다.
  - 기본 HTTP 클라이언트 라이브러리는 Reactor Netty 다.
- 기존에 Spring MVC 기반에서 사용하던 RestTemplate은 Blocking 만을 지원하기 떄문에 리액티브 프로그래밍을 위해서는 WebClient를 사용하는 것이 좋다.

```java
WebClient webClient = WebClient.create("http://localhost:8080");
Mono<BookDto.Response> response =
        webClient
                .patch()
                .uri("http://localhost:8080/v10/books/{book-id}", 20)
                .bodyValue(requestBody)
                .retrieve()
                .bodyToMono(BookDto.Response.class);
```

## 20.3 WebClient Connection Timeout 설정

- WebClient는 특정 서버 엔진의 HTTP Client Connector 설정을 통해 Timeout을 설정할 수 있다.

```java
HttpClient httpClient =
        HttpClient
                .create()
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 500)
                .responseTimeout(Duration.ofMillis(500))
                .doOnConnected(connection ->
                    connection
                            .addHandlerLast(
                                    new ReadTimeoutHandler(500,
                                                        TimeUnit.MILLISECONDS))
                            .addHandlerLast(
                                    new WriteTimeoutHandler(500,
                                                        TimeUnit.MILLISECONDS)));

Flux<BookDto.Response> response =
        WebClient
                .builder()
                .baseUrl("http://localhost:8080")
                .clientConnector(new ReactorClientHttpConnector(httpClient))  // Netty 가 아니라 다른 Connector 설정 가능
                .build()
                .get()
                .uri(uriBuilder -> uriBuilder
                        .path("/v10/books")
                        .queryParam("page", "1")
                        .queryParam("size", "10")
                        .build())
                .retrieve()
                .bodyToFlux(BookDto.Response.class);

```

## 20.4 exchangeToMono()를 사용한 응답 디코딩

- retrieve() 대신 exchangeToMono()나 exchangeToFlux() 를 사용하여 response를 사용자의 요구에 맞게 제거할 수 있다.
- retrieve: ResponeEntity를 받아 디코딩 처리
  - 상태에 대한 처리는 별도 로직 필요.
- exchange: ClientResponse를 받아서 상태 값과 헤더 등을 같이 처리
