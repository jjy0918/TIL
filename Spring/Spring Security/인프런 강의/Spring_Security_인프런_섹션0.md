## 인프런 Spring Security 정리(08/03)

<br>

## 섹션0. 스프링 시큐리티 : 폼 인증

<hr>

### 스프링 웹 프로젝트 만들기

Principal

- java.security에 존재하는 인터페이스
- 인증된 사용자가 있다면, Principal 구현체를 Parameter로 받아 사용할 수 있다.
- 현재 페이지에 접근하는 유저에 대한 정보를 획득할 수 있다.
- princiapl이 null인 경우 유저가 없다는 것을 알 수 있다.

인증 페이지

- 인증을 해야 접속이 가능한 페이지가 존재한다.(로그인 필요)
- 인증을 하지 않아도 접속이 가능한 페이지가 존재한다.
- Spring Security 설정을 통해 권한이 필요한 페이지 설정이 가능한다.

<hr>
<br>

### 스프링 시큐리티 연동

Spring Security 기본 설정

- Spring Security dependency 추가
  - spring-boot-starter-security
- 의존성 추가 시 기본 조건은 모든 요청은 인증을 필요로 한다.
- 기본 유저가 생성된다.
  - Username : user
  - Password : 콘솔에 출력된 문자열.
- 해결된 문제
  - 인증을 할 수 있다.
  - 현재 사용자 정보를 알 수 없다.
- 새로운 문제
  - 인증 없이 접근 가능한 URL을 설정하고 싶다.
  - 이 애플리케이션을 사용할 수 있는 유저 계정이 그럼 하나 뿐인가?
  - 비밀번호가 로그에 남는다!

<hr>
<br>

### 스프링 시큐리티 설정하기

Spring Security 설정클래스

- @Configuration과 @EnableWebSecurity 어노테이션 추가
- WebSecurityConfigurerAdapter를 상속 받는다.
- configure(HttpSecurity http) 메소드를 오버라이딩 한다.
  - 이 메소드에서 인증해야 하는, 하지 않는 URL을 설정할 수 있다.
  - http.authorizeRequests().mvcMatchers("url").permitAll()
    => 인증을 하지 않는 URL
  - http.authorizeRequests().mvcMatchers("url").hasRole("ADMIN")
    => Admin 권한이 필요한 URL
  - http.authorizeRequests().anyRequest().authenticated()
    => 나머지 요청은 인증이 필요하다.

```java
http.authorizeRequests()
                .mvcMatchers("/", "/info", "/account/**", "/signup").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated();

http.formLogin();
http.httpBasic();


```

==> configure 메소드 내에서 인증이 필요한 URL을 설정할 수 있다.

<hr>
<br>

### 스프링 시큐리티 커스터마이징 : 인메모리 유저 추가

AuthenticationManagerBuilder

- WebSecurityConfigurerAdapter를 상속 받은 클래스를 생성한다.
- configure(AuthenticationManagerBuilder auth) 메소드를 오버라이딩한다.
- auth.inMemoryAuthentication()
  .withUser("유저아이디").password("비밀번호").roles("역할"); 의 형식을 통해 유저 설정이 가능하다.

<hr>
<br>

### 스프링 시큐리티 커스터마이징 : JPA 연동

UserDetailService

- loadUserByUsername
  - username을 받아서 UserDetails를 리턴해준다.
  - DAO에서 username을 이용하여 가져온다.
  - username이 없는 경우 예외를 던진다.
  - username이 있는 경우 UserDetails 타입으로 변화하여 리턴한다.
- Spring Security에서 UserDetails를 간단하게 만들 수 있도록 User객체를 제공한다.
- Spring Security에서 제공하는 것.
- Authentication 관리 시 DAO를 통해서 유저 정보를 읽어온다.
- 인터페이스 구현 시 어떠한 DAO 구현체를 사용하는지는 상관 없다.
-

```java

public AccountService implements UserDetailsService{

    @Autowired
    AccountRepository accountRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException{
        Account account = accountRepository.findByUsername(username);
        if(account == null){
            throw new UsernameNotFoundException(username);
        }

        return User.builder()
            .username(account.getUsername())
            .password(account.getPassword())
            .roles(account.getRole())
            .build();
    }
}


```

- SpringSecurity는 password에 특별한 패턴을 요구한다.
- {암호화 알고리즘 패턴}비밀번호
- {noop}password 라 하면 비밀번호 암호화 하지 않도록 만들어준다.

- WebSecurityConfigurerAdapter를 구현한 클래스의 configure(AuthenticationManagerBuilder auth) 메소드 오버라이드한 부분에서 DetailService를 등록할 수 있다.
  - bean으로 등록한 경우 하지 않아도 된다.

```java

@Override
protected void configure(AuthenticationManagerBuilder auth) thorws Exception{
    // accountService는 UserDetailsService를 구현한 클래스

    auth.userDetailsService(accountService);
}


```

### 스프링 시큐리티 커스터마이징 : PasswordEncoder

SpringSecurity에는 password 기본 포맷이 존재한다.

- {noop}password

```java

@Bean
public PasswordEncoder passwordEncoder(){
    // 기본은 bcrypt
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}

////
this.password = passwordEncoder.encode(this.password);

```
