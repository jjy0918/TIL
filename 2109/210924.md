# Spring Security - 스프링 시큐리티 : 아키텍처

## SecurityContextHolder와 Authentication

Principal > Authentication > SecurityContext > SecurityContextHolder

Principal

- "누구"에 해당하는 정보
- UserDetailsService에서 리턴한 그 객체
- 객체는 UserDetails 타입

Authentication

- Principal을 담고 있는 객체
- Principal과 GrantAuthority를 제공한다.

SecurityContext

- Authentication을 감싸고 있는 객체

SecurityContextHolder

- SecuurityContext를 제공한다.
- 기본적으로 ThreadLocal을 사용한다.
  - ThreadLocal : 한 쓰레드 내에서 공유하는 저장소
  - => Authentication을 한 쓰레드 내에서 공유할 수 있다.

```java

    // SecurityContextHolder는 해당 쓰레드내 저장소에서 가져온다.
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

    // 사용자에 대한 정보
    Obeject principal = authentication.getPrincipal();

    // 사용자의 권한
    Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();

    // 사용자 인증 여부 확인
    // Custom으로 만료 여부 체크 가능
    boolean authenticated = authentication.isAuthenticated();


```

UserDetails

- 애플리케이션이 가지고 있는 유저 정보와 스프링 시큐리티가 사용하는 Authentication 객체 사이의 어댑터

UserDetailsService

- 유저 정보를 UserDetails 타입으로 가져오는 DAO 인터페이스
- 구현은 마음대로 할 수 있다.
- loadUserByUsername 메소드를 오버라이딩 하여 UserDetails를 리턴한다.
- AuthenticationManager에서 인증을 진행한다.

<br>

<hr>

<br>

## AuthenticationManager와 Authentication

AuthenticationManager

- 스프링 시큐리티에서 인증(Authenticaion)을 진행하는 부분
- AuthenticationManager는 authenticate라는 메소드 하나만 가지고 있다.
- 인증이 되었다면, authenticate 메소드에서 Authentication을 리턴한다.
- 인자로 받은 authentication는 사용자가 입력한 인증에 필요한 정보(username, password)로 만든 객체이다.
- 리턴된 Authentication의 Principal에는 UserDetailService의 loadUserByUsername에서 리턴한 UserDetails 구현체가 들어간다.

```java

Authentication authenticate(Authentication authentication) throws AuthenticationException;

```

ProviderManager

- AuthenticationManager의 구현체
- ProviderManager는 Provider 들을 통해 인증을 진행한다.
- Provider는 인증시 UserDetailService를 사용하여 인증을 진행한다.
- => 실제 인증 진행은 AuthenticationManager에서 진행하고, 그 과정에서 UserDetailsService를 사용한다.

UserDetailsService

- loadUserByUsername에서 username을 가지고, User객체(UserDetails 구현체)를 가져온다.
- User객체의 비밀번호와 입력한 비밀번호(Credentail)이 일치하는지는 다른 부분에서 진행한다.

<br>

<hr>

<br>

## ThreadLocal

- java.lang 패키지에서 제공하는 쓰레드 범위 변수.
- 쓰레드 수준의 저장소.
- 같은 쓰레드 내에서만 공유
- 따라서 같은 쓰레드라면 해당 데이터를 메소드 매개변수로 넘겨줄 필요 없음.
- SecurityContextHolder의 기본 전략
