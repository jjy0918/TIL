# Kotlin의 open 키워드

<hr>

## open 키워드란?

코틀린은 자바 기반으로 만들어졌다.

하지만, 자바와 다른 점이 몇 가지 존재한다.

그 중 하나가 open이라는 키워드이다.

기본적으로 자바에서는 final 키워드를 명시해주어야 상속이 불가능해진다.

하지만, 코틀린에서는 기본적으로 final 키워드가 삽입되고, open 키워드를 명시해주어야 상속이 가능해진다.

<hr>

## Spring에서 open 키워드

기본적으로 Spring에서 Bean을 등록하는 방법은 여러가지가 존재한다.

그 중, @Configuration을 선언한 클래스에서 @Bean을 선언한 메소드가 리턴하는 클래스는 Bean으로 등록된다.

```java
@Configuration
public class ConfTest{
    @Bean
    public BeanTest beanTest(){
        return new BeanTest();
    }
}
```

Kotlin 역시 동일한 방식으로 Bean을 등록할 수 있다.

다만, 여기서 주의할 점은 자바처럼 아무런 키워드를 명시하지 않으면 자동으로 final 키워드가 붙는다는 것이다.

Spring에서는 @Configuration과 @Bean을 이용한 Bean 등록 시 cglib를 이용하여 프록시 객체를 생성하고 등록하는 방식을 사용한다.

즉, 위 코드의 ConfTest와 beanTest 메소드에 open 키워드를 명시해주어야 정상적인 Bean 등록이 가능해진다.

또한, 프록시 객체가 생성되야 하기 때문에 리턴하는 BeanTest역시 open 키워드가 필요하다.

```kotlin
@Configuration
open class ConfTest {
    @Bean
    open fun testController() :BeanTest{
        return BeanTest()
    }
}
```

## all-open

Spring에서 Bean으로 등록할 때 모두 open일 하나하나 명시해주어야 한다.

이러한 작업들은 너무 불편하다.
그래서 all-open이라는 플러그인이 존재한다.

이 플러그인을 사용하면, Spring에서 open을 명시해야 하는 클래스와 메소드에 자동으로 open 키워드를 달아준다.

Spring 뿐만 아니라, 그냥 코틀린 코드에서도 옵션을 통해 사용할 수 있다.

```gradle
plugins {
  id "org.jetbrains.kotlin.plugin.spring" version "1.6.10"
}
```
