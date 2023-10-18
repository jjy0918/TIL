# 21 Reactive Streaming 데이터 처리

- Spring WebFlux는 SSE(Server-Sent Events)를 이용하여 데이터를 Streaming 할 수 있다.
  - SSE는 Spring 4.2 부터 지원되었으며, Spring 5 부터는 Flux를 사용하여 더 편리한 방법으로 사용할 수 있다.
  - Spring MVC 에서는 `SseEmitter`를 통해 구현해야 한다.
- SSE란 클라이언트가 HTTP 연결을 통해 서버로부터 전송되는 업데이트 데이터를 지속적으로 수신할 수 있는 단방향 서버 푸시 기술이다.
  - SSE는 주로 클라이언트 측에서 서버로부터 전송되는 이벤트 스트림을 자동으로 수신하기 위해 사용된다.
- Spring WebFlux에서 Streaming 방식으로 데이터를 전송하기 위해서는 response Type을 `Flux`로 설정해야 하며, Content-Type이 `text/event-stream` 이어야 한다.
- responseType을 `ServerSentEvent`로 설정할 경우 별도의 Content-Type을 명시하지 않아도 된다.

```java
// 일반적인 Flux + Content-Type 설정
@Bean
public RouterFunction<?> routeStreamingBook(BookService bookService,
                                            BookMapper mapper) {
    return route(RequestPredicates.GET("/v11/streaming-books"),
            request -> ServerResponse
                    .ok()
                    .contentType(MediaType.TEXT_EVENT_STREAM)
                    .body(bookService
                                    .streamingBooks()
                                    .map(book -> mapper.bookToResponse(book))
                            ,
                            BookDto.Response.class));
}

// 별도의 설정 없이 ServerSentEvent 사용
@GetMapping("/stream-sse")
public Flux<ServerSentEvent<String>> streamEvents() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(sequence -> ServerSentEvent.<String> builder()
        .id(String.valueOf(sequence))
          .event("periodic-event")
          .data("SSE - " + LocalTime.now().toString())
          .build());
}
```