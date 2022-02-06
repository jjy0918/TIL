# 코틀린에서 Spring Bean 등록 시 주의사항

## @Configuration & @Bean

Spring에서 일반적으로 Bean을 등록하는 방법은 `@Configuration & @Bean`을 이용하거나
`@Component`, `@Controller`, `@Service`, `@Repository`를 이용해야 한다.

코틀린에서 `@Configuration & @Bean`을 이용하여Bean을 등록할 때 주의할 점이 있다.

`@Configuration`을 선언한 클래스는 open이여야 한다.
또한, `@Bean`을 선언한 메소드 역시 open이어야 한다.

그 이유는 Configuration을 통해 Bean을 등록하는 방식때문이다.

<br>

### CGLIB 프록싱

일반적으로 @Configuration을 이용하여 Bean을 등록하는 방식은 CGLIB를 통해 등록이 된다.
@Configuration을 상속 받는 프록시 객체가 Bean으로 등록되기 때문이다.

하지만, 코틀린에서는 기본적을 class나 fun 선언 시 final 키워드가 자동으로 붙는다.

즉, final 키워드가 선언된 클래스와 메소드는 상속이 불가능해지기 때문에 프록시 객체가 생성되지 않고, Bean으로 등록이 불가능해진다는 것이다.

=> 결론적으로, 코틀린에서 `@Configuration & @Bean`을 이용하기 위해서는 open 키워드를 적어야 한다.

## @Transactional & AOP

JPA에서 `@Transactional`이 적용되는 방식 역시 프록시 객체를 생성하게 된다.

즉, `@Transactional`이 적용된 메소드 역시 코틀린에서는 open 키워드를 적어야 한다.

이 방식은 AOP에서도 동일하게 적용이 가능하다.

`@Transactional` 역시 AOP 처럼 프록시 객체를 만들고, 해당 Bean의 메소드가 실행하려고 하면, 프록시 객체가 실행된다.

=> 결론적으로, 코틀린에서 `@Transactional`이나 AOP를 이용하기 위해서는 해당 클래스와 메소드에 open 키워드를 적어야 한다.
