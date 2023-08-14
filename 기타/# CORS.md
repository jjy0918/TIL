# CORS

## CORS란

- CORS란 원 출처와 다른 출처로 요청할 때 `브라우저`에서 확인하는 로직을 의미한다.
- 서버1에서 처음 실행 중일 때 서버2의 API를 요청하는 경우 출처가 달라지기 때문에 브라우저에서 검증하는 로직이라고 할 수 있다.
  - 요청하려는 리소스의 출처가 다신과 다를 때
- 기본적으로 브라우저는 동일 출처를 원칙으로 한다.

## CORS 동작 원리

- CORS 로직은 간단하게, 출처가 다른 요청이 발생했을 때 `다른 출처의 서버`에게 나의 `원 출처(ORIGIN)`와 요청 `Header`가 허용되는지 물어본 후 요청하는 것이다.
- 출처가 다른 모든 요청에 대해 CORS 로직이 동작하는 것은 아니다. 요청이 `SimpleRequest` 인 경우 CORS 로직과 상관 없이 요청이 동작하게 된다.

### Simple Request

- 위에 설명처럼 Simple Request의 경우에는 CORS는 아래 설명하는 preflight를 하지 않고 CORS인지 판단하는 로직.
- Simple Request에서는 `Access-Control-Allow-Origin` 으로 Origin만 판단한다.
- Simple Request의 응답에서는 특정 헤더만 읽는다.
  - Cache-Control
  - Content-Language
  - Content-Type
  - Expires
  - Last-Modified
  - Pragma
  - 이 헤더 외에는 읽지 않고, 브라우저가 읽기 위해서는 `Access-Control-Expose-Headers` 에 해당 해더를 포함하여 전달해야 한다.
- 허용 가능 HTTP Method
  - GET
  - HEAD
  - POST
- 허용 가능 Header
  - User-Agent(브라우저) 에서 자동으로 설정하는 값
    - Connection
    - User-Agent
  - 이미 정의되어 사용자가 설정하지 못하는 값
    - Accept-Charset
    - Accept-Encoding
    - Access-Control-Request-Headers
    - Access-Control-Request-Method
    - Connection
    - Content-Length
    - Cookie
    - Cookie2
    - Date
    - DNT
    - Expect
    - Host
    - Keep-Alive
    - Origin
    - Referer
    - Set-Cookie
    - TE
    - Trailer
    - Transfer-Encoding
    - Upgrade
    - Via
    - proxy- 로 시작하는 값
    - sec- 로 시작하는 값
  - 사용자 지정 헤더
    - Accept
    - Accpet-Language
    - Content-Language
    - Content-Type
      - application/x-www-form-urlencoded
      - multipart/form-data
      - text/plain

### Preflight Request

- Simple Request에 허용되는 값이 아니면, 브라우저는 CORS 로직을 발동한다.
- CORS 로직은 preflight 요청을 다른 출처의 서버로 요청한 후 허용되는 출처(Origin)인지, 허용하는 헤더인지 판단하여 실제 요청을 실행한다.
- preflight는 HTTP `OPTIONS` 메소드를 통해 요청한다.
- OPTIONS 요청
  - `Origin` 헤더를 통해 Origin 값을 전달하여 허용 가능한 원 출처인지 전달한다.
  - `Access-Control-Request-Method` 헤더를 통해 요청하려는 HTTP 메서드를 전달한다.
  - `Access-Control-Request-Headers` 헤더를 통해 요청하려는 Header를 전달한다.
    - 이때 전달하는 헤더는 이미 정의되어 사용자가 설정하지 못하는 값 외에 모두 해당된다.
    - 즉, `Connection`, `User-Agent` 와 같은 값들은 브라우저만 설정할 수 있기 때문에 해당이 되지 않고 `Accept`, `Content-Type` 등 커스텀하게 설정할 수 있는 값들은 포함된다.
- OPTIONS 응답
  - `Access-Control-Allow-Origin` 을 통해 허용 가능한 Origin을 전달한다.
  - `Access-Control-Allow-Methods` 을 통해 허용 가능한 HTTP 메서드를 전달한다.
  - `Access-Control-Allow-Headers` 을 통해 허용 가능한 Header를 전달한다.
    - 허용 헤더 값은 기본적으로 사용자가 요청에 포함시켜야 할 수 있는 값만 넣으면 된다.
    - Access-Control-Request-Headers 에 서술되어 있는 것 처럼 `Connection`, `User-Agent` 등 브라우저만 설정할 수 있는 값은 넣어도 의미 없다.
- 브라우저는 preflight 응답을 바탕으로 정상 출처에 대한 요청인지 판단한 후 정상적이라 판단되면 실제 요청을 실행한다.

### 자격 증명 요청

- 기본적으로 브라우저는 요청 시 `credentialed requests` 를 포함하지 않은 상태로 전달된다.
- 즉, HTTP `Cookies와` HTTP `Authentication` 정보를 포함하지 않는다.
- 이를 허용하기 위해서는 `Access-Control-Allow-Credentials` 값을 `true`로 전달해야 한다.
- 이것은 simple Request도 동일하다.
  - 쿠키 or Authentication을 포함하여 요청한 경우에는 서버 응답이 정상적이라 하더라도 브라우저는 그 값을 사용하지 않고 무시한다.
- 주의할 점으로 자격 증명 요청 시에는 서버 응답 시 `Access-Control-Allow-Origin`의 값을 `*`로 내려주면 `안된다.`

### 기타

- `Access-Control-Expose-Headers`
  - 이 헤더는 브라우저가 접근할 수 있는 `화이트리스트` 헤더다.
  - 말 그대로 접근할 수 있는, 노출될 수 있는 헤더들을 말한다.
  - 브라우저는 여기서 선언되지 않은 헤더 값은 읽지 않는다.
- `Access-Control-Max-Age`
  - preflight 응답 결과를 캐싱할 수 있는 시간을 나타낸다.