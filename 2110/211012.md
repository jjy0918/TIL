# REST API

## REST API란

- REST 아키텍쳐 스타일을 따르는 API

## REST

- 분산 하이퍼미디어 시스템(EX. 웹)을 위한 아키텍쳐 스타일

## 아키텍쳐 스타일

- 제약조건의 집합
- => 제약조건을 모두 지켜야 REST를 따른다는 것이라고 말할 수 있다.

## REST를 구성하는 스타일

- client-server
- stateless
- cache
- uniform interface
- layered system
- code-on-demand(optional)
  => 대부분 http를 사용하면 잘 지킬 수 있다.(uniform interface 제외)

## Uniform Interface의 제약 조건

- identification of resources
  - 리소스가 URI로 식별되면 된다.
  - 잘 지켜진다.
- manipulation of resources through representations
  - 리소스를 만들거나 업데이트, 삭제 시 HTTP 메세지에 표현을 담아서 전송해야 한다.
  - 잘 지켜진다.
- self-descriptive message
  - 메시지는 스스로 설명해야 한다.
  - 잘 안지켜진다.
- hypermedia as the engine of application state(HATEOAS)
  - 애플리케이션의 상태는 Hyperlink를 이용해서 전이되어야 한다.
  - 잘 안지켜진다.

## self-descriptive message

- 메시지는 스스로 설명해야 한다.
- host, content-type 이 없거나
- 각 내용이 뭘 의미하는지 알 수 없는 경우 성립하지 않는다고 할 수 있다.
- 메시지 내용으로 완벽하게 해석할 수 있어야 한다.

```
HTTP/1.1 200 OK
[ {"op":"remove","path":"/a/b/c"}]
=> 해당 본문이 무엇을 의마하는지 알 수 없다.

HTTP/1.1 200 OK
Content-Type : application/json
[ {"op":"remove","path":"/a/b/c"}]
=> 해당 본문 json 타입임을 명시하여, 파싱할 수 있도록 안내한다.
=> 하지만 여전히 무엇을 의미하는지 알 수 없다.

HTTP/1.1 200 OK
Content-Type : application/json-patch+json
[ {"op":"remove","path":"/a/b/c"}]
=> json-patch+json이라는 미디어 타입으로 정의된 메시지이기 때문에
   해당 명세를 찾아가서 이해한 다음 메시지를 해석할 수 있다.
   ===> 메시지 내용으로 온전히 해석이 가능해졌다.

```

## hypermedia as the engine of application state(HATEOAS)

- 애플리케이션의 상태는 Hyperlink를 이용해서 전이되어야 한다.

```
HTTP/1.1 200 OK
Content-Type:text/html
<html>
<head></head>
<body><a href="/test">teset</a></body>
</html>

=> 하이퍼링크를 이용해 전이되기 때문에 HATEOAS를 만족한다.

HTTP/1.1 200 OK
Content-Type:application/json
Link:</articles/1>; rel="previes",
     </articles/3>; rel="next";
{
  "title":"The second article",
  "contents":"blah blah..."
}

=> json 안에 하이퍼 링크를 통해 다른 상태로 전이 가능하기 때문에 만족한다.

```

## 왜 Uniform Interface?

- 독립적인 진화
  - 서버와 클라이언트가 각각 독립적으로 진화한다.
  - 서버의 기능이 변경되어도 클라이언트를 업데이트할 필요가 없다.
  - REST를 만들게 된 계기 : "How do I improve HTTP without breaking the Web"

## 웹

- 웹 페이지를 변경했다고 웹 브라우저를 업데이할 필요가 없다.
- 웹 브라우저를 업데이트 했다고 웹 페이지를 변경할 필요도 없다.
- HTTP 명세가 변경되어도 웹은 잘 동작한다.
- HTML 명세가 변경되어도 웹은 잘 동작한다.

## REST API

- 하이퍼텍스트를 포함한 self-desciptive한 메시지의 uniform interface를 통해 리소스에 접근하는 API

## 왜 API는 REST가 잘 안되는가?(웹과 비교)

- Protocol
  - 웹페이지 : HTTP
  - HTTP API : HTTP
- 커뮤니케이션
  - 웹페이지 : 사람 - 기계
  - HTTP API : 기계 - 기계
- Media Type

  - 웹페이지 : HTML
  - HTTP API : JSON

- Hyperlink
  - HTML : 됨(a 태그 등)
  - JSON : 정의되어 있지 않음
- self-descriptive
  - HTML : 됨(HTML 명세)
  - JSON : 불완전(문법 해석은 가능하지만, 의미를 해석하려면 별도의 문서가 필요하다.)

## HTML의 Self-descriptive, HATEOAS

- 응답 메시지의 Content-Type을 보고 media type이 text/html 임을 확인한다.
- HTTP 명세에 media type은 IANA에 등록되어 있다고 하므로, IANA에서 text/html의 설명을 찾는다.
- IANA에 따르면 text/html의 명세는 http://www.w3.org/TR/html 이므로 링크를 찾아가 명세를 해석한다.
- 명세에 모든 태그의 해석방법이 구체적으로 나와있으므로 이를 해석하여 문서 저자가 사용자에게 전달할 수 있다.
- a태그를 이용해 표현된 링크를 통해 다음 상태로 전이될 수 있으므로 HATEOAS를 만족한다.

## JSON의 Self-descriptive, HATEOAS

- 응답 메시지의 Content-Type을 보고 media type이 application/json 임을 확인한다.
- HTTP 명세에 media type은 IANA에 등록되어 있다고 하므로, IANA에서 application/json의 설명을 찾는다.
- IANA에 따르면 application/json의 명세는 draft-ietf-jsonbis-rfc7159bis-04 이므로 링크를 찾아가 명세를 해석한다.
- 명세에 json 문서를 파싱하는 방법이 명시되어있으므로 성공적으로 파싱하지만, "id", "title" 이 무엇을 의미하는지 알 방법이 없다.
- 다음 상태로 전이할 링크가 없기 때문에 HATEOAS도 만족하지 않는다.

## self-descriptive message

- 확장 가능한 커뮤니케이션을 제공
- 서버나 클라이언트가 변경되더라도 오고 가는 메시지는 언제나 self-descriptive 하므로 언제나 해석이 가능하다.

## HATEOAS

- 애플리케이션 상태 전이의 late binding
- 어디서 어디로 전이가 가능한지 미리 결정되지 않는다.
- 어떤 상태로 전이가 완료되고 나서야 그 다음 전이될 수 있는 상태가 결정된다. 쉽게 말해서 링크는 동적으로 변경될 수 있다.

## HTTP API를 REST API로 바꾸기(self-descriptive)

1.  Media type

- 미디어 타입을 하나 정의한다.
- 미디어 타입 문서를 작성한다. 이 문서에 "id"가 뭐고, "title"이 뭔지 정의한다.
- IANA에 미디어 타입을 등록한다. 이 때 만든 문서를 미디어 타입의 명세로 등록한다.
- 이제 이 메시지를 보는 사람은 명세를 찾아갈 수 있으므로 이 메시지의 의미를 온전히 해석할 수 있다.
- => 매번 media type을 정의해야 한다는 단점이 있다.

2. Profile

- "id"가 뭐고, "title"이 뭔지 의미를 정의한 명세를 작성한다.
- Link 헤더에 Profile relation으로 해당 명세를 링크한다.
- 이제 메시지를 보는 사람은 명세를 찾아갈 수 있으므로 이 문서의 의미를 온전히 해석할 수 있다.
- => 클라이언트가 Link 헤더(RFC 5988)와 profile(RFC 6906)을 이해해야 하고, Content negotiation을 할 수 없다는 단점이 있다.

## HTTP API를 REST API로 바꾸기(HATEOS)

1.  data로

- data에 다양한 방법으로 하이퍼링크를 표현한다.
- 링크를 표현하는 방법을 직접 정의해야 한다는 단점이 있다.
- 기존 API를 고쳐야 한다는 단점이 있다.

```
[
 {
   "link":"https://example.org/todos/1",
   "title":"회사가기"
 }
]

```

2.  HTTP 헤더로

- Link, Location 등의 헤더로 링크를 표현한다.
- 정의된 relation만 활용한다면 표현에 한계가 있다는 단점이 있다.

```
HTTP/1.1 204 No Content
Location:/todos/1
Link:</todos/>; rel="collection"
```
