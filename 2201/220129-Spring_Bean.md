# Spring Bean

## Bean 등록

### 1. Component, Controller, Service, Respository이용

- 클래스 위에 @Component, @Controller, @Service, @Repository등의 어노테이션을 이용하여 등록한다.

```java
@Component
public class TestBean{
   public void print(){
       System.out.println("빈 등록 테스트");
   }
}

```

### 2. Configuration과 Bean 어노테이션 이용

- Configuration 어노테이션을 선언한 클래스에 Bean 어노테이션을 이용하여 Bean을 등록한다.
- @Bean이 선언된 메소드의 리턴 값이 빈으로 등록된다.

```java
@Configuraion
public class ConfTest{

    @Bean
    public TestBean testBean(){
        return new TestBean();
    }
}
```

- 여기서 중요한 점은, 해당 메소드를 호출하는 경우 컨텍스트에 등록된 빈이 리턴된다는 것이다.

## Bean 주입 받기

### 1. 생성자 주입

```java
@Component
public class TestComponent{
    private final TestBean testBean;

    @Autowired
    public TestComponent(TestBean testBean){
        this.testBean = testBean;
    }
}

```

- 생성자를 통해 주입 받는 것.
- 생성자 호출 시 객체가 단 한 번만 주입 받는 것을 보장한다.
- 잘못된 값을 주입받는 경우가 없어진다.
- `스프링에서 권장하는 방식이다.`
- 생성자가 1개인 경우 Autowired를 생략할 수 있다.

### 2. 필드 주입

```java
@Component
public class TestComponent{
   @Autowired
   private final TestBean testBean;

}
```

- 필드에 @Autowired를 이용하여 주입한다.
- 외부에서 변경이 불가능하다는 단점이 존재한다.
- 제한적인 환경 때문에 사용을 지양한다.

### 3. Setter 주입

```java
@Component
public class TestComponent{

   private TestBean testBean;

   @Autowired
   public void setTestBean(TestBean testBean){
       this.testBean = testBean;
   }

}
```

- 필드의 값을 setter를 이용하여 주입받는 방식.
- 외부에서 쉽게 변경이 가능하기 때문에, 변경 가능성이 있는 경우 사용된다.
- 주입할 대상이 없는 경우 오류가 발생된다.

## Configuration에 대한 궁금증

### @Configuration과 @Component 차이점

- 일반적으로 @Configuration은 @Bean을 이용하여 빈을 등록하는 경우에 사용된다.
- @Component는 클래스를 빈으로 등록할 때 사용한다.
- 그렇다면 @Configuration과 @Bean은 다른 것인가??
  => 그렇지 않다. @Configuration을 살펴보면 결국 @Component를 선언하고 있다.
- 즉, @Configuration을 선언한 클래스도 빈으로 등록된다는 것이다.

### @Configuration의 @Bean 메소드의 리턴 값은 무엇을 리턴할까?

- @Bean이 선언된 메소드는 새로운 객체를 생성하여 리턴하는 것 처럼 보인다.
- 처음 스프링이 실행되면, 객체를 리턴하여 컨테이너에 빈으로 등록하게 된다.
- 그러면 빈으로 등록된 Configuration의 리턴 값은 무엇을 리턴할까? => 등록된 빈을 리턴한다.

### @Configuration이 아닌 @Compoent에 @Bean을 사용할 수 있을까?

- 이용 가능하다.
- @Bean을 통해 빈 등록이 가능하고, 가져오는 것도 정상적으로 된다.
