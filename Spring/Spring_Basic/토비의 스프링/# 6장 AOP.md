# 6장 AOP

- AOP는 IoC/DI, 서비스 추상화와 더불어 스프링 3대 기반기술 중 하나다.

## 6.1 트랜잭션 코드의 분리

- 지금까지 작성한 UserService 코드는 트랜잭션 경계설정 관련된 코드를 계속해서 넣어주어야 한다.
- 트랜잭션 경계설정 코드와 비즈니스 로직 코드는 뚜렷하게 구분되어 있기 때문에 서로간 주고받는 정보가 없다.
  - 비즈니스 로직에서는 직접 DB를 사용하지 않기 때문에 DB 커넥션 정보가 필요 없다.
  - 즉, 트랜잭션 로직과 비즈니스 로직은 코드 성격도 다르고 주고 받는 데이터도 없는 완벽하게 독립적인 코드라고 할 수 있다.
- 이를 해결하기 위해 UserService를 인터페이스로 만들고, 실제 비즈니스 로직을 가지는 `UserServiceImpl`과 트랜잭션 경계를 담당하는 `UserServiceTx` 로 나눈다.
  - `UserServiceTx`는 트랜잭션을 담당하고, 실제 비즈니스 로직은 `UserServiceImpl`에서 수행한다
- 트랜잭션 코드를 분리하는 것의 장점
  - 비즈니스 로직을 담당하고 있는 부분과 트랜잭션을 담당하는 부분이 나뉘기 때문에 비즈니스 로직에서는 트랜잭션에 관련된 부분을 몰라도 된다.
  - 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있다.

## 6.2 고립된 단위 테스트

- 일반적으로 테스트는 가능한 한 작은 단위로 쪼개서 테스트하는 것이다.
  - 작은 단위 테스트는 실패했을 때 원인 찾기가 쉽다.
  - 또한 테스트 의도나 내용이 분명해지고 만들기가 쉽다.

### 테스트 대상 오브젝트 고립시키기

- 일반적으로 클래스는 다양한 의존 오브젝트를 가지고 있기 때문에 테스트를 하기 위해서 불필요한 작업을 할 수 있다.
- 그렇기 때문에 테스트의 대상이 환경이나 외부 클래스에 종속되거나 영향을 받지 않도록 고립시킬 필요가 있다.
- 테스트 대상 오브젝트를 고립시키기 위해서 테스트 대역을 사용하는 것이 가장 좋다.

### 단위 테스트와 통합 테스트

- 단위 테스트는 테스트 대역을 통해 외부 오브젝트나 외부 리소스를 사용하지 않고 고립시켜서 테스트하는 것을 말한다.
- 통합 테스트는 외부 오브젝트나 외부 리소스를 참여하는 테스트를 말한다.
  - 즉, 통합 테스트란 두 개 이상의 단위가 결합하여 동작하면서 테스트가 수행되는 것이라 할 수 있다.

### 목 프레임워크

- 단위 테스트에서 대상 오브젝트를 고립시키기 위해서 목 오브젝트를 만들어주어야 하지만, 매번 만들어주기 굉장히 번거롭니다.
- 목 오브젝트를 간단하게 만드는 것을 도와주는 목 오브젝트 지원 프레임워크가 존재한다.
  - `Mockito 프레임워크` 등

## 6.3 다이내믹 프록시와 팩토리 빈

### 6.3.1 프록시와 프록시 패턴, 데코레이터 패턴

- 트랜잭션 코드를 `핵심 기능`을 가지고 있는 클래스와 트랜잭션 기능을 가지고 있는 `부가 기능` 클래스로 나눈다.
  - 이 경우, 두 클래스는 서로의 역할만으로 구분되어 있기는 하지만, 부가 기능 클래스는 핵심 기능 클래스를 사용하는 형태가 된다.
  - 즉, 클라이언트가 핵심 기능을 가진 클래스를 직접 사용할 경우 부가 기능을 적용할 수 없게 된다.
  - 클라이언트는 반드시 부가 기능을 거쳐서 핵심 기능을 사용해야 된다.
- 클라이언트와 소통하는 핵심 기능을 가진 것 처럼 위장한 클래스를 `프록시`라고 부른다.
  - 핵심 기능을 감싸고 있는 부가 기능을 구현한 객체를 프록시 객체라 볼 수 있다.
  - 프록시는 타깃과 같은 인터페이스를 구현했고, 타깃을 제어할 수 있는 위치에 있다.
- 프록시를 통해 위임 받아 처리하는 실제 오브젝트를 `타깃` 또는 `실체(real subject)`라고 부른다.
  - 클라이언트 --> 프록시 --> 타깃 형태로 구현된다.

#### 데코레이터 패턴

- `데코레이터 패턴`은 `타깃`에 부가적은 기능을 런타임 시 다이나믹하게 부여하기 위한 패턴이다.
  - 런타임 시 다이나믹하게 부여한다는 것은 코드상으로는 어떤 방법으로 어떠한 타깃과 연결되어 있는지 정해지지 않았다는 것이다.
- 데코레이터 패턴은 말 그대로 여러개의 데코레이터를 가질 수 있다.
  - 즉, 여러개의 프록시를 통해 감싸서 구현될 수 있다는 것이다.
  - 클라이언트 --> 라인넘버 데코레이터 --> 컬러 데코레이터 --> 페이징 데코레이터 --> 소스코드 출력 기능(타겟)
- `데코레이터 패턴`의 구현은 `타깃`과 같은 `인터페이스`를 구현한다.
- 데코레이터 `위임` 대상 역시 인터페이스로 접근하기 때문에 최종 타겟인지, 데코레이터인지 알 수 없다.
  - 다음 위임 대상은 인터페이스로 선언하고, 생성자나 수정자 메소드를 통해 `외부`에서 `런타임 시` `주입` 받을 수 있도록 만들어야 한다.

#### 프록시 패턴

- `프록시 패턴`의 프록시는 타깃의 기능을 확장하거나 추가하지 않고 클라이언트가 타깃에 `접근`하는 방식을 변경해준다.
- 클라이언트에서 타깃 오브젝트의 `객체 생성`을 `당장 하지 않고` `레퍼런스`만 `미리` 필요할 경우 사용된다.
- 또한, `타깃`의 `접근권한` `제어`를 하기 위해 사용한다.
  - 프록시 객체에서 특정 메소드 실행 시 에러 발생 등의 역할을 구현할 수 있다.
- 즉, `프록시 패턴`은 오브젝트의 `접근 제어` 역할을 한다고 볼 수 있다.

### 6.3.2 다이나믹 프록시

- `프록시`는 `기존 코드`는 `영향`을 `주지 않고` 타깃의 `기능을 확장`하거나 `접근 제어`할 수 있게 도와준다.
- 프록시는 구조적으로 좋은 방법이지만, 구현 시 번거로움이 존재한다.
  - 프록시를 구현하기 위해 인터페이스 구현 시 위임 부가 기능이 필요 없는 메소드를 일일이 만들어줘야 한다.
  - 부가 기능이 중복될 가능성이 많기 때문에 주의가 필요하다.
- `JDK`의 다이나믹 프록시를 사용하면, 번거로움 중 메소드 구현과 위임 기능 문제를 해결할 수 있다.

#### 리플렉션

- 다이나믹 프록시는 `리플렉션`을 통해 만들 수 있다.
  - 리플렉션은 자바의 코드 자체를 추상화해서 접근하도록 만든 것이다.

```java
public class ReflectionTest {
  @Test
  public void invokeMethod() throws Exception {
    String name = "Spring";

    assertThat(name.length(), is(6));

    Method lengthMethod = String.class.getMethod("length");
    assertThat((Integer)lengthMethod.invoke(name), is(6));
  }
}
```

#### 다이나믹 프록시 적용

- 리플랙션을 통해 `Method` 추출 후 `invoke` 메소드 실행 시 해당 객체의 메소드가 실행됨을 확인했다.
- 이를 이용하여 `InvocationHandler`와 `프록시 팩토리`를 조합하면, 프록시 객체를 간단하게 구현함과 동시에 원하는 메소드를 주입시켜 실행할 수 있다.
- `InvocationHandler`는 `invoke`라는 메소드 하나만 가지고 있는 인터페이스이다.
  - invoke 메소드는 `proxy Object`, `Method`, `매개 변수 배열(Object[])`를 매개변수로 받는다.
  - invoke 메소드의 매개변수 값을 통해 타깃 메소드를 실행(Method.invoke)할 수 있고, 부가 기능 추가 등 프록시의 기능을 추가할 수 있다.
- `프록시 팩토리`는 `타깃 객체`의 정보와 `InvocationHandler`를 받아서 타깃 객체의 프록시를 만들어준다.
  - `Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { Hello.class }, new UppercaseHandler(new HelloTarget()));`

#### 다이나믹 프록시의 확장

- 다이나믹 프록시 방식은 직접 정의하는 프록시보다 훨씬 유연하고 많은 장점이 있다.
  - 타깃 인터페이스의 메소드가 많거나 변경되어도 프록시 생성에는 변화가 없다.
  - 타깃 인터페이스의 메소드 변경 시 단순히 `InvocationHandler`만 변경되면 된다.
- `InvocationHandler`의 Method 값을 통해 리턴 타입이나 특정 메소드에서만 적용될 부가 기능을 구현할 수 있다.
  - 기본적으로 `InvocationHandler`는 단일 메소드에서 모든 요청을 처리하기 때문에 어떤 기능을 적용할지 구분해야 한다.

### 6.3.3 다이나믹 프록시를 이용한 트랜잭션 부가기능

- 기존에 구현했던 트랜잭션 기능(전략 패턴)을 프록시를 통해 간단하게 분리하여 구현할 수 있다.
- 트랜잭션 기능을 구현하기 위해 `InvocationHandler`를 구현하면 된다.

### 6.3.4 다이나믹 프록시를 위한 팩토리 빈

- 기본적으로 빈 주입 시 클래스 이름을 가지고 리플랙션을 이용하여 오브젝트를 만든다.
- 다이나믹 프록시를 통해 만들어지는 객체의 경우에는 클래스 이름을 알 수 없기 때문에 빈으로 등록할 수 없다.
- 스프링에서는 빈 생성 시 `기본 생성자`를 통해 만든다.
- `FactoryBean`을 구현한 객체를 빈으로 등록하고 있다면, `FactoryBean`에 구현된 방법으로 생성된다.
  - 이때 만들어지는 빈의 타입은 FactoryBean의 제네릭 타입 즉, getObject() 메소드 리턴 타입 값이다.
  - FactoryBean 값 자체를 가져오고 싶다면, `&`값을 빈 앞에 붙여주면 된다.

#### 다이나믹 프록시를 만들어주는 팩토리 빈

- FactoryBean을 이용하면 다이나믹 프록시를 빈으로 등록할 수 있다.
- 스프링에서는 `팩토리 빈`과 `타깃`을 빈으로 등록하면 프록시 빈을 등록할 수 있다는 것이다.
- 트랜잭션 기능을 가진 프록시 팩토리빈을 만들면, 여러 핵심 기능에서 공통적으로 적용할 수 있다.

```xml
<bean id="userService" class="springbook.user.service.TxProxyFactoryBean">
  <property name="target" ref="userServiceImpl" />  <!-- 트랜잭션 타깃 설정 -->
  <property name="transactionManager" ref="transactionManager" />
  ...
</bean>
```

### 6.3.5 프록시 팩토리 빈 방식의 장점과 한계

#### 장점

- 프록시 팩토리 빈의 재사용
  - 타깃 오브젝트에 맞는 프로퍼티 정보만 빈으로 등록해주면 하나의 프록시 기능을 여러 핵심 프록시 빈으로 등록할 수 있다.
- 클라이언트의 코드 변경 없이 프록시가 제공해주는 트랜잭션 기능을 이용할 수 있다.

#### 한계

- 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 것이 불가능하다.
  - 트랜잭션과 같이 비슷한 부가기능이 필요한 경우 거의 비슷한 팩토리 빈 설정이 중복될 수 있다.
- 하나의 타깃에 여러 부가기능을 적용하기가 번거롭다.
  - 프록시 적용시 팩토리 빈 설정이 계속 추가된다.
- 같은 기능을 하는 부가 기능 객체가 팩토리 빈 개수만큼 늘어난다.

## 6.4 스프링 프록시 팩토리 빈

### 6.4.1 ProxyFactoryBean

- 자바에서는 JDK에서 제공하는 다이내믹 프록시 외에도 편리하게 프록시를 만들 수 있는 기술들을 지원해준다.
- 스프링에서는 `일관된 방법`으로 프록시를 만들 수 있게 도와주는 `추상화 레이어`를 제공한다.
- 스프링에서 `ProxyFactoryBean`은 `프록시`를 생성하여 `스프링 빈`으로 등록해게 도와준다.
  - ProxyFactoryBean은 `프록시 오브젝트`를 `생성`해주는 `기술`을 `추상화`한 `팩토리 빈`을 제공한다.
  - ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만 담당한다.
- `MethodInterceptor` 인터페이스를 구현하여 ProxyFactoryBean가 생성한 프록시에서 사용할 기능을 명시한다.
  - MethodInterceptor는 InvocationHanlder와 다르게 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지 제공받는다.
  - 즉, MethodInterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다.
  - MethodInterceptor는 독립적이기 때문에 여러 프록시에서 함께 사용이 가능하며, 싱글톤 빈으로 등록할 수 있다.

#### 어드바이스: 타깃이 필요 없는 순수한 부가기능

- `MethodInterceptor`에서는 타깃 오브젝트가 등장하지 않고, `invoke`메소드의 매개변수인 `MethodInvocation`을 통해 타깃 오브젝트의 메소드를 실행할 수 있다.
- 메소드 실행 부분이 독립적으로 존재하기 때문에 부가기능에만 집중할 수 있다.
  - 즉, MethodInterceptor는 부가 기능에만 집중하게 되고, 실제 메소드 로직이 어떻게 동작하는지는 관심이 없다.
  - MethodInterceptor는 일종의 공유 가능한 템플릿처럼 동작한다고 볼 수 있다.
- MethodInterceptor는 공유 가능한 템플릿/콜백 형태라서 JDK 다이나믹 프록시와 달리 싱글톤으로 동작할 수 있다.
- `ProxyFactoryBean`에서는 `MethodInterceptor`를 여러개 등록하여 제공할 수 있다.
- `MethodInterceptor` 처럼 타깃 오브젝트에 여러 부가 기능을 제공하는 오브젝트를 `어드바이스(advice)`라고 부른다.
  - MethodInterceptor는 Advice를 상속하고 있는 서브 오브젝트다.
  - MethodInterceptor는 메소드의 실행을 가로채서 부가기능을 제공하는 것이고 이 외에도 다양한 어드바이스들이 존재한다.
- `ProxyFactoryBean`는 구현해야 할 인터페이스를 알려주지 않아도 내부적으로 자동검출 기능을 통해 알아낼 수 있다.
  - 자동검출 기능을 통해 알아낸 인터페이스를 모두 구현하는 프록시를 만들어준다.
  - 모두가 아니라 일부만 구현하는 프록시를 적용하기 위해서는 인터페이스 정보를 직접 제공해주어야 한다.

#### 포인트컷: 부가기능 적용 대상 메소드 선정 방법

- `MethodInterceptor`는 타깃 오브젝트의 정보를 가지고 있지 않도록 구현했기 때문에 특정 오브젝트의 메소드에서만 적용되도록 하는 것은 옳지 않다.
  - 기존 `TxProxyFactoryBean` 처럼 메소드 이름 패턴을 바탕으로 적용해야할 메소드만 구분하는 것은 안된다.
- 어떠한 메소드에 적용할지를 부가기능 자체에 집중하는 `MethodInterceptor`와 같은 Advice에서 하는 것이 아니라, 별도의 메소드 선정 알고리즘을 가지고 있는 오브젝트를 따로 구현하는 것이 좋다.
  - 이렇게 메소드 선정 알고리즘을 가지고 있는 오브젝트를 `포인트 컷(Pointcut)`이라고 한다.
- 메소드 선정 알고리즘을 가진 오브젝트와 부가기능을 가지고 있는 오브젝트를 분리하여 OCP 원칙을 잘 지킬 수 있는 구조가 된다.
- ProxyFactoryBean이 프록시 생성 --> 다이내믹 프록시 생성 --> 포인트 컷으로부터 대상 선정 --> 선정된 메소드만 어드바이스를 통해 부가기능 추가 후 콜백 리턴
- `어드바이스`와 `포인트컷` 모두 프록시에 DI로 주입되어 사용된다.
  - 어드바이스와 포인트컷을 프록시로부터 독립되어 있기 때문에 부가기능이나 메소드 선정 방식이 바뀌어도 간단하게 변경이 가능해진다.
- 어드바이스처럼 포인트 컷도 싱글톤 빈등로 등록할 수 있다.
- 포인트컷 추가 시 포인트컷과 어드바이스를 담은 `Advisor`를 추가해야 한다.
  - 여러개의 어드바이스 등록되더라도 각각 다른 포인트컷과 조합될 수 있다.
  - 어드바이저 = 포인트컷 + 어드바이스

```java
@Test
public void pointcutAdvisor() {
  ProxyFactoryBean pfBean = new ProxyFactoryBean();
  pfBean.setTarget(new HelloTarget());

  // 메소드 이름을 비교하여 대상을 선정하는 포인트 컷 추가
  NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
  pointcut.setMappedName("sayH*");

  // 포인트컷과 어드바이스를 Advisor로 묶어서 한 번에 추가
  pfBean.addAdvisor(new DefaultPointCutAdvisor(pointcut, new UpperCaseAdvice()));

  Hello proxiedHello = (Hello) pfBean.getObject();

  assert(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
  assert(proxiedHello.sayHi("Toby"), is("HI TOBY"));
  assert(proxiedHello.sayThankYou("Toby"), is("Thank You TOBY")); // 포인트컷에 의해 부가기능 적용 X
}
```

### 6.4.2 ProxyFactoryBean 적용

#### TransactionAdvice

- MethodInterceptor를 통해 타깃 오브젝트에 의존하지 않을 수 있고, 리플렉션을 통해 메소드 호출할 필요가 없어진다.

```java 
public class TransactionAdvice implements MethodInterceptor {
  PlatformTransactionManager transactionManager;
  
  public void setTransactionManager(PlatformTransactionManager transactionManager) {
    this.transactionManager = transactionManager;
  }

  public Object invoke(MethodInvocation invocation) throws Throwable {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());

    try {
      Object ret = invocation.process();
      this.transactionManager.commit(status);
      return ret;
    } catch (RuntimeException e) {
      this.transactionManager.rollback(status);
      throw e;
    }
  }
}
```

#### 스프링 XML 설정파일

```xml
<!-- advice bean 등록 -->
<bean id="transactionAdvice" class="springbook.user.service.TranscationAdvice">
  <property name="transacionManager" ref="transactionManager" />
</bean>

<!-- pointcut bean 등록 -->
<bean id="transactionPointcut" class="org.springframewor.aop.support.NameMatchMethodPointcut">
  <property name="mappedName" value="upgrade*" />
</bean>

<!-- advisor bean 등록 -->
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
  <property name="admivce" ref="transactionAdvice" />
  <property name="pointcut" ref="transactionPointcut" />
</bean>

<!-- ProxyFactoryBean 등록 -->
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
  <property name="targer" ref="userServiceImpl" />
  <property name="interceptorNames">  <!-- 어드바이스와 어드바이저 동시에 설정가능 -->
    <list>
      <value>transactionAdvisor</value>
    </list>
  </proerty>
</bean>

```

#### 어드바이스와 포인트컷 재사용

- `ProxyFactoryBean`은 스프링의 DI와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용되었다.
- 어드바이스와 포인트컷은 확장 기능을 분리하여 여러 프록시가 공유할 수 있게 되었다.
- 하나의 어드바이스를 다양한 포인트컷과 조합하여 다양한 서비스에 적용할 수 있다.

## 6.5 스프링 AOP

### 6.5.1 자동 프록시 생성

- 위 예시를 통해서 타깃과 메소드를 재사용할 수 있도록 분리하였지만 비슷한 내용의 ProxyFactoryBean 빈 설정 정보를 계속해서 추가해야 한다는 불편함은 남아있다.
- 결국 타깃 객체가 아니라, 프록시 객체를 빈으로 등록하여 사용하기 때문에 자동으로 프록시를 만들고 등록하면 된다.
- 스프링은 OCP의 가장 중요한 요소인 유연한 확장 개념을 적용하고 있다.
- `BeanPostProcessor`를 통해 빈 후 처리를 할 수 있도록 기능을 제공한다.
  - 스프링은 빈 후 처리기가 등록되어 있으면, 빈 오브젝트를 만들 때마다 후처리기에 빈을 보낸다.
- 스프링이 제공하는 빈 후 처리기 중 기본적인 `DefaultAdvisorAutoProxyCreator`를 통해 간단하게 프록시 객체를 생성하여 빈으로 등록할 수 있다.
  - 빈 후 처리기를 빈으로 등록하면, 빈 오브젝트 생성 시 프로퍼티 수정 등의 추가적인 작업을 제공한다.

#### 확장된 포인트 컷

- 포인트컷은 메소드 판단 뿐 아니라 클래스 필터 로직도 가지고 있다.
- 즉, 프록시를 적용할 클래스에 대한 판단과, 그 클래스에서 어떤 메소드에 적용할 지를 결정하는 로직을 담고 있다.
- BeanPostProcessor를 사용한다면, 정확한 포인터컷과 어드바이스가 결합되어 있는 어드바이저가 등록되어 있어야 한다.

### 6.5.2 DefaultAdvisorAutoProxyCreator의 적용

#### 클래스 필터를 적용한 포인트컷 작성

```java
public class NameMatchClassMethodPoincut extends NameMatchMethodPointcut {
  public void setMappedClassName(String mappedClassName) {
    this.setClassFilter(new SimpleClassFilter(mappedClassName));
  }

  static class SimpleClassFilter implements ClassFilter {
    String mappedName;

    private SimpleClassFilter(String mappedName) {
      this.mappedName = mappedName;
    }

    public boolean matches(Class<?> clazz) {
      return PatternMatchUtils.simpleMatch(mappedName, clazz.getSimpleName());
    }
  }

}

```

#### 어드바이저를 이용하는 자동 프록시 생성기 등록

- `DefaultAdvisotrAutoProxyCreator`는 `Advisor 인터페이스`를 구현한 모든 `Bean`을 찾고, 생성되는 `Bean` 에 대해 어드바이저의 포인트컷을 적용하여 프록시 대상을 선정한다.
  - 생성된 Bean이 프록시 대상이라면(포인트컷의 ClassFilter에 의해 선정된 경우), 프록시를 만들어 Bean 오브젝트로 등록한다.
  - 프록시가 Bean으로 등록될 경우, 원래 Bean은 등록되지 안흔ㄴ다.
- `DefaultAdvisotrAutoProxyCreator`에 의해 어드바이저들을 따로 ProxyFactoryBean을 통해 프록시 객체를 만들고 Bean으로 등록할 필요가 없어진다.
  - 즉, `ProxyFactoryBean` 필요가 없고, `Advisor` Bean과 그 Bean에 주입될 `PointCut`, `Advice` Bean만 등록하면 된다.

#### 자동 프록시 생성기를 사용하는 테스트

- 테스트할 때 `@Autowired`를 통해 콘텍스트에서 가져오는 Bean의 경우 프록시가 적용되었다면, 실제 타깃 오브젝트가 아닌 부가 기능이 추가된 프록시 객체를 가져와야 한다.
- 스프링 컨텍스트를 통해 Bean을 가져오기 때문에 이전 테스트에서 사용했던 TestUserService를 강제로 주입하기 위해서는 `TestUserService`도 Bean으로 등록해야 한다.
- `TestUserService`는 실제 사용중인 UserServiceImpl의 기능을 가지고 있어야 하며, 트랜잭션 등 부가 기능들도 적용해야 한다.
  - 이를 위해 Bean 등록 시 `parent` 애트리뷰트를 사용하면 편리하다.
  - 또한, 포인트 컷을 통해 부가 기능을 등록할 수 있도록 해야 하기 때문에 ClassFilter에 맞게 변경이 필요하다.

```xml
<bean id="testUserService" 
  class="springbook.user.service.UserServiceTest$TestUserServiceImpl"
  parent="userService" />
```

#### 자동생성 프록시 확인

- 빈 후 처리기를 통해 자동으로 프록시를 생성할 경우 실제 예외가 발생하기 전에는 잘 적용이 되었는지 보기 어렵기 때문에 주의가 필요하다.
- 프록시 자동생성기를 적용할 경우 두 가지는 최소한으로 확인해야 한다.
  - 1. 부가기능이 필요한 빈에 부가기능이 잘 적용되었는가.
    - 부가기능이 잘 동작하는지에 대한 테스트가 반드시 있어야 한다.
    - ex) 부가기능이 트랜잭션이라면 예외 발생 시 롤백이 잘 되는가 를 확인해야 한다.
  - 2. 아무 Bean에나 부가기능이 적용된 것이 아닌지 확인해야 한다.
    - 모든 Bean을 확인하기 보다는, 클래스 필터가 제대로 동작하는지만 확인하면 된다.
    - ex) 포인트 컷의 클래스 필터가 이름 기준으로 동작한다면, 이름을 변경할 경우 적용이 안되는지 확인하면 된다.
- 프록시 빈이 등록되었는지에 대해 Bean 타입이 java.lang.reflect.Proxy의 서브클래스인지 확인하는 것도 좋다.
  - JDK의 다이나믹 프록시 방식으로 만들어진 프록시들은 모두 java.lang.reflect.Proxy의 서브클래스이다.

### 6.5.3 포인트컷 표현식을 이용한 포인트 컷

- 이제까지 예시에서 사용한 것 처럼 이름을 통해 클래스나 메소드를 구분하는 포인트 컷도 있지만 패키지, 파라미터, 리턴 값, 어노테이션 등으로도 판단할 수 있다.
- 리플랙션 API를 통해서 여러 포인트 컷을 구현할 수 있지만, 복합하고 번거롭다.
- 스프링은 아주 간단하고 효과적인 방법으로 포인트컷 클래스와 메소드 선정 알고리즘을 작성하게 도와준다.
  - JSP의 EL과 아주비슷한 `포인트컷 표현식(pointcut expression)`이다.

#### 포인트컷 표현식

- 포인트컷 표현식을 지원하는 포인트컷을 사용하기 위해서는 `AspectJExpressionPoincut` 클래스를 사용하면 된다.
- `AspectJExpressionPoincut` 은 클래스와 메소드의 선정 알고리즘을 포인트컷 표현식을 이용해 한 번에 지정할 수 있게 해준다.
- `AspectJExpressionPoincut` 은 AspectJ 프레임워크에서 제공하는 것을 가져와 일부 문법을 확장해서 만든 것이다.

#### 포인트컷 표현식 문법

- AspectJ 포인트컷 표현식은 포인트컷 지시자를 이용해 작성한다.
- 포인트컷 지시자 중 가장 대표적으로 사용되는 것이 `execution()`이다.
- `execution` 지시자
  - execution([접근제한자 패턴] (리턴 값)타입패턴 [패키지, 클래스 타입패턴.](메소드)이름패턴 (타입패턴 | "..", ...) [throws 예외 패턴])
  - 정리하자면, `접근제한다(public, private) / 리턴 타입 / 메소드 이름(패키지, 클래스 포함, .으로 생략 가능) / 매개변수 / 예외` 형태이다.
  - ex) public int springbook.pointcut.Target.minus(int, int) throws java.lang.RuntimeException
- 접근 제한자는 설정할 수도 있지만, 생략할 수도 있다.
- 리턴 타입은 `필수` 값으로 반드시 하나의 타입 이상 지정해야 한다. 모든 타입 설정 시 `*` 을 지정해주면 된다.
- 메소드는 패키지와 클래스 타입등이 포함될 수 있고, 생략될 수도 있다. 하지만 `메소드`는 `필수` 이다.
  - 패키지 이름과 클래스 또는 인터페이스 이름에 *을 사용할 수 있다.
  - .. 를 사용하면 한 번에 여러 패키지를 선택할 수 있다.
  - 모든 메소드를 선택하려면 `*`을 넣으면 된다.
- 파라미터 역시 `필수` 값이지만, 모든 값을 허용하려면 `..`을 넣으면 된다.
  - `...` 은 뒷 부분 파라미터 조건만 생략하는 것이기 때문에 혼동을 주의해야 한다.
- 예외 이름에 대한 타입은 생략 가능하다.

```java
// 예시
execution(int minus(int, int))  // 리턴 타입 int, 메소드 이름 minus, 매개변수는 (int, int)
execution( * minus(int, int)) // 리턴 타입 모든 타입, 메소드 이름 minus, 매개변수는 (int, int)
execution( * minus(..)) // 리턴 타입 모든 타입, 메소드 이름 minus, 매개변수는 모든 값
execution( * *(..)) // 리턴 타입 모든 타입, 메소드 이름 모든 값, 매개변수는 모든 값
```

#### 포인트컷 표현식을 이용하는 포인트컷 적용

- AspectJ에서는 메소드 시그니처를 이용한 `execution()` 뿐 아니라 빈 이름 비교하는 `bean()`이나 어노테이션을 통해 선정하는 `@annotation` 도 있다.

```xml
<bean id="transactionPointcut"
  class="org.springframework.aop.aspectj.AspectJExpressionPointcut">
  <property name="expression" value="execution(* *..*ServiceImpl.upgrade*(..))" />
</bean>
```

#### 타입 패턴과 클래스 이름 패턴

- AspectJ 를 통해 포인트컷을 만드는 경우, 클래스 선정 시 `타입 패턴`을 적용한다.
- 클래스 이름만을 기준으로 하는 것이 아니라, 해당 클래스의 모든 타입을 바탕으로 판단한다.
- 즉, TestUserService라는 클래스가 UserServiceImpl과 UserService를 상속 받는 경우 `*ServiceImpl.upgrade`도 적용되고, `*.UserService.upgrade` 도 적용된다.

### 6.5.4 AOP란 무엇인가?

- AOP: 애스팩트 지향 프로그래밍
- 객체지향적인 관점에서는 부가기능 모듈화는 힘들다.
- 그래서 객지향적인 관점이 아니라, `Aspect`라는 관점으로 부가기능을 다루기로 했다.
- Aspect란 말 그대로 핵심기능을 담고 있지는 않지만, 애플리케이션을 구성하는 중요한 한 가지 요소이며 핵심기능에 부가되어 의미를 갖는 모듈을 의미한다.
- Aspect는 부가 기능인 `Advice`와 어디에 적용할 지 결정하는 `Pointcut` 을 가지고 있다.
  - Advisor는 아주 단순한 형태의 Aspect라고 볼 수 있다.
- Aspect로 분리함에 따라 핵심기능은 순수하게 그 기능을 가지게 도록힙적으로 살펴볼 수 있게 된다.
- 핵심적인 기능에서 부가적인 기능을 분리하여 Aspect라는 모듈로 만들어서 설계하고 개발하는 방법을 `관점 지향 프로그래밍(Aspect Oriented Programming, AOP)` 이라고 한다.
- AOP 는 OOP와 다른 패러다임이라기 보다는, OOP를 돕는 보조적인 개념이라고 보면 된다.

### 6.5.5 AOP 적용 기술

#### 프록시를 이용한 AOP

- 스프링에서는 프록시를 이용한 AOP를 제공한다.
  - 프록시를 만들어서 DI로 연결된 빈 사이에 적용하여 부가기능을 제공한다.
  - 스프링 AOP는 타깃 오브젝트 호출 전후로 독립적으로 개발된 부가기능을 다이나믹하게 적용하기 위해 프록시를 사용한다.
- 그렇기 때문에 스프링 AOP는 JDK와 스프링 컨테이너만 있으면 적용할 수 있다.

#### 바이트코드 생성과 조작을 통한 AOP

- 스프링의 AOP 처럼 프록시 객체를 생성하여 AOP를 적용하는 것이 아니라, `AspectJ` 처럼 바이트 코드를 조작하여 적용할 수 있다.
- `AspectJ`는 타깃 오브젝트를 `뜯어 고쳐서` 부가 기능을 `직접` 넣어주는 방법을 사용한다.
- 부가 기능을 넣기 위해 코드를 수정하는 것이 아니라, `컴파일된 파일 자체`를 수정하거나 클래스가 `JVM에 로딩되는 시점`에 가로채서 바이트 코드를 조작한다.
- 바이트 코드를 통해 AOP를 적용하는 것은 프록시 적용하는 것 보다 장점이 있다.
  - 1. 바이트 코드를 조작하기 때문에 스프링과 같은 컨테이너의 도움 없이도 AOP를 적용할 수 있다.
    - 즉, 다양한 환경에서 AOP를 손쉽게 사용할 수 있다.
  - 2. 프록시 방식보다 훨씬 강력하고 유연하게 AOP가 가능하다.
    - 프록시 방식의 경우 부가 기능는 단순히 `메소드`에서만 가능하다.
    - 바이트 코드 조작을 통한 AOP는 메소드뿐 아니라 `오브젝트 생성`, `필드 값 조회`, `스태틱 초기화` 등에도 부가기능을 넣을 수 있다.
- 바이트 코드 조작을 통한 AOP는 JVM 실행 옵션 변경이나 별도의 컴파일러, 클래스 로더 등을 사용해야 한다는 단점이 존재한다.

### 6.5.6 AOP 용어

- 타깃
  - 부가기능을 부여할 대상
  - 핵심기능을 담은 클래스일 수도 있지만, 다른 부가기능을 제공하는 프록시 오브젝트일 수도 있다.
- 어드바이스
  - 타깃에게 제공할 부기기능을 담은 모듈
  - 메소드 호출 전반적으로 참여할 수도 있고, 예외가 발생했을 때만 동작할 수도 있는 등 여러 어드바이스가 존재한다.
- 조인 포인트
  - 어드바이스가 적용될 수 있는 `위치`
  - 스프링 AOP에서 조인 포인트는 `메소드`의 실행 단계뿐이다.
- 포인트컷
  - 어드바이스를 적용할 조인 포인트를 선별하는 작업이나 기능을하는 모듈
  - 스프링 AOP에서 조인 포인트는 메소드뿐이기 때문에 포인트컷은 메소드 선정 기능을 가지고 있다.
  - 포인트컷 표현식은 execution으로 시작고, 메소드의 시그니처를 비교하는 방법을 주로 이용한다.
- 프록시
  - 프록시는 클라이언트와 타깃 사이에 투명하게 존재하면서, 부가기능을 제공하는 오브젝트다.
  - DI를 통해 클라이언트에게 주입되며, 클라이언트의 메소드 호출을 대신 받아서 타깃에게 위임하여 부가기능을 부여한다.
  - 스프링 AOP는 프록시를 이용해 AOP를 지원한다.
- 어드바이저
  - 어드바이저는 포인트컷과 어드바이스를 하나씩 갖고 있는 오브젝트를 말한다.
  - 스프링은 자동 프록시 생성기가 어드바이저를 AOP 작업의 정보로 활용한다.
  - 스프링의 AOP에서만 사용되는 용어다.
- 애스팩트
  - AOP의 고본 모듈로 한 개 이상의 포인트컷과 어드바이스의 조합으로 이루어지며, 보통은 싱글톤 형태의 오브젝트이다.

## 6.6 트랜잭션 속성

### 6.6.1 트랜잭션 정의

#### 트랜잭션 전파

- 트랜잭션 전파(Transaction Propagation)란 트랜잭션 경계에서 이미 진행 중인 트랜잭션이 있는 경우 혹은 없을 때 어떻게 동작할 것인가를 결정하는 방식을 의미한다.
- PROPAGATION_REQUIRED
  - 가장 많이 사용되는 전파 속성이다.
  - 진행 중인 트랜잭션이 있으면 참여하고, 없다면 새로 시작한다.
  - DefaultTransactionDefinition의 전파 속성이다.
- PROPAGATION_REQUIRES_NEW
  - 항상 새로운 트랜잭션을 시작한다.
  - 독립적인 트랜잭션이 보장되어야 하는 코드에 적용할 수 있다.
- PROPAGATION_NOT_SUPPORTED
  - 트랜잭션 없이 동작하도록 만든다.
  - 진행중인 트랜잭션이 있어도 무시한다.

#### 격리 수준

- 모든 DB 트랜잭션은 격리수준(Isolation Level)을 가지고 있어야 한다.
- 여러 트랜잭션이 동시에 진행되면서 문제가 발생하지 않도록 제어하기 위해서 격리 수준을 설정한다.
- 격리 수준은 기본적으로 DB에 설정되어 있지만, JDBC 드라이버나 DataSource 등에서 재설정할 수 있다.
- 트랜잭션 단위로도 격리 수준을 설정할 수 있다.
- DefaultTransactionDefinition 에서는 기본적으로 ISOLATION_DEFAULT를 사용한다.
  - DataSource에 설정되어 있는 기본 설정을 따른다.

#### 제한 시간

- 트랜잭션 수행 시간을 제한할 수 있다.
- DefaultTransactionDefinition 에서는 기본적으로 제한 시간을 설정하지 않는다.
- 제한 시간은 트랜잭션을 시작하는 PROPAGATION_REQUIRED나 PROPAGATION_REQUIRES_NEW 와 함께 사용해야 의미가 있다.

#### 읽기 전용

- 읽기 전용으로 설정해두면 트랜잭션 내에서 데이터를 조작하는 시도를 막이줄 수 있다.

### 6.6.2 트랜잭션 인터셉터와 트랜잭션 속성

- 트랜잭션 정의를 수정하기 위해서는 경계 설정 기능을 가진 TransactionAdvice를 기능별로 재정의 하는 것 보다는 `TransactionInterceptor` 를 사용하는 것이 좋다.
- `TransactionInterceptor` 는 이미 스프링에서 편리하게 트랜잭션 경계설정 어드바이스로 사용할 수 있도록 만들어 둔 것이다.
- `TransactionInterceptor` 는 기본적으로 TransactionAdvice와 동일하게 동작하면서 메소드 이름 패턴을 이용하여 다르게 지정할 수 있는 방법을 추가로 제공해준다.
- `TransactionInterceptor` 는 `PlatformTransactionManager` 와 `Properties` 타입의 두 가지 프로퍼티를 갖는다.
  - `Properties` 타입의 프로퍼티는 `트랜잭션 속성`을 정의한 `TransactionAttribute` 타입의 프로퍼티다.
  - `Properties` 는 `transactionAttributes`라는 이름을 가지며,  (메소드 패턴, 트랜잭션 속성)을 (key, value) 형태의 Map으로 전달 받는다.
- `TransactionInterceptor` 프로퍼티는 기본적으로 4가지 속성(전파, 격리 수준, 제한 시간, 읽기 전용) 항목과 롤백 여부를 판단하는 `rollbackOn` 메소드를 가지고 있다.
  -  rollbackOn 메소드는 어떤 예외가 발생할 때 롤백할 지를 결정한다.
- `TransactionInterceptor` 는 기본적으로 예외에 따라 두 가지 처리 방식이 존재한다.
  - `런타임 예외` 발생 시 트랜잭션은 `롤백`된다.
  - `체크 예외` 발생 시 예외 상황으로 판단하지 않아서 `커밋` 한다.
  - 스프링에서는 기본적으로 체크 예외는 비즈니시 로직 상 의미가 있다고 판단한다.
  - `rollbackOn` 에서는 이러한 원칙 외 다른 예외처리를 가능하도록 한다.

#### 메소드 이름 패턴일 이용한 트랜잭션 속성 지정

- transactionAttributes의 트랜잭션 속성
  - PROPAGATION_NAME: 트랜잭션 전파 방식으로 필수 항목이다.
  - ISOLATION_NAME: 격리 수준으로 생략 가능하다.
  - readOnly: 읽지 전용 항목으로 생략 가능하다.
  - timeout_NNNN: 제한시간으로 생략 가능하다.
  - -Exception1: 체크 예외 중 롤백 대상으로 추가할 것을 넣는다.
  - +Exception2: 런타임 예외지만, 롤백시키지 않을 예외들을 넣는다.
- 메소드 이름 패턴이 하나 이상과 일치한 경우, 가장 정확히 일치하는 패턴이 적용된다.

```xml
<bean id="transactionAdvice" class="org.springframework.transaction.interceptor.TransactionInterceptor">
  <property name="transactionManager" ref="transactionManager" />
  <property name="transactionAttributes">
    <props>
      <prop key="get*">PROPAGATION_REQUIRED,readOnly,timeout_30</prop>
      <prop key="upgrade*">PROPAGATION_REQUIRES_NEW,ISOLATION_SERIEALIABLE</prop>
      <prop key="*">PROPAGATION_REQUIRED</prop>
    </props>
  </property>
</bean>
```

- 트랜잭션의 경우 기본적으로 자주 사용되기 때문에 `tx` 스키마의 태그를 이용해 간단하게 정의할 수 있다.
  - 네임스페이스 및 스키마 위치 지정 필요.

## 6.7 애노테이션 트랜잭션 속성과 포인트컷

- 일반적으로는 포인트컷 표현식과 트랜잭션 속성을 이용해 트랜잭션을 일괄적으로 적용하는 방식은 잘 들어맞는다.
- 모든 트랜잭션 로직을 일괄적으로 처리하기에는 클래스나 메소드에 따라 제각각 속성이 다르거나 튜닝된 트랜잭션을 적용해야 하는 경우가 있다.
  - 모든 경우에 대해 포인트컷과 어드바이스를 새로 추가해줘야 하면 지저분해지고 복잡해진다.
- 스프링에서는 트랜잭션 속성 제어를 설정파일ㄹ이 아니라, 어노테이션을 이용하여 타깃에 직접 지정하는 방법을 제공한다.

### 6.7.1 트랜잭션 어노테이션

- 스프링 3.0 부터는 자바 5에서 등장한 어노테이션을 이용하여 다양한 기능과 설정을 제공하며, `@Transactional` 도 그 중 하나다.
- `@Transactional` 은 클래스, 어노테이션, 메소드에서 사용할 수 있다.
- 스프링에서는 `@Transactional` 이 부여된 모든 오브젝트를 자동으로 타깃 오브젝트로 인식한다.
  - 이때 `TransactionAttributeSourcePointCut` 을 포인트컷으로 사용하지만, 별도의 표현식이나 선정 방식이 있는 것은 아니다.

#### @Transactional

```java
package org.springframework.transaction.annotation; // Spring의 annotaion이다.

@Target({ElementType.TYPE, ElementType.METHOD}) // 어노테이션을 사용할 대상 지정(TYPE --> 클래스, 인터페이스 / METHOD --> 메소드)
@Retention(RetentionPolicy.RUNTIME) // 어노테이션이 언제까지 유지되는지에 대한 내용
@Inherited  // 상속을 통해서 어노테이션 정보를 얻을 수 있게 한다.
@Documented
public @interface Transactional {
	@AliasFor("transactionManager")
	String value() default "";
	@AliasFor("value")
	String transactionManager() default "";
	String[] label() default {};
	Propagation propagation() default Propagation.REQUIRED;
	Isolation isolation() default Isolation.DEFAULT;
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
	String timeoutString() default "";
	boolean readOnly() default false;
	Class<? extends Throwable>[] rollbackFor() default {};
	String[] rollbackForClassName() default {};
	Class<? extends Throwable>[] noRollbackFor() default {};
	String[] noRollbackForClassName() default {};
}

```

#### 트랜잭션 속성을 이용하는 포인트컷

- `@Transactional` 을 통한 트랜잭션에서 `TransactionInterceptor` 는 메소드 이름 패턴이 아니라 `AnnotationTransactionAttributeSource` 를 사용하여 어노테이션에 선언된 트랜잭션 속성을 가져ㅑ온다.
- `@Transactional` 을 사용하면 메소드마다 다르게 설정할 수 있기 때문에 유여한 설정이 가능해진다.
- `@Transactional` 이 선언된 오브젝트라면, 포인트컷 대상이 된다.
- 정리하자면, `@Transactional`을 통해 포인트컷 대상과 트랜잭션 속성 정보를 한 번에 설정할 수 있다.

#### 대체 정책

- 스프링에서는 `@Transactional` 적용 시 4단계의 대체(fallback) 정책을 제공한다.
  - 메소드에서 Transaction 속성 확인 시 `타깃 메소드` -> `타깃 클래스` -> `선언 메소드` -> `선언 타입(클래스, 인터페이스)` 순서로 `@Transactional`이 적용되었는지 확인한다.

```java
[1]
public interface Serivce {
  [2]
  void method1();

  [3]
  void method2();
}

[4]
public class ServiceImpl implements Service {
  [5]
  public void method1() {

  }

  [6]
  public void method2() {

  }
}
```

- 위 코드에서 SerivceImpl의 method1 에 Transaction 속성 확인 시 [5] -> [4] -> [2] -> [1] 순으로 `@Transactional` 을 탐색한다.
- 클래스 내에서 동일한 트랜잭션 속성을 사용한다면 [4] 또는 [1] 에 선언하면 된다.
  - 클래스에 트랜잭션을 적용했기 때문에, 모든 메소드가 트랜잭션 대상이 됨을 주의해야 한다.
  - [4] 에 선언하게 되면 실제 타깃 클래스(ServiceImpl)의 모든 메소드에서만 트랜잭션이 적용된다.
  - [1] 에 선언하게 되면 ServiceImpl 뿐 아니라 Serivce의 모든 하위 클래스 모두 트랜잭션이 적용되기 때문에 주의해야 한다.
- 클래스 내에서 동일한 트랜잭션 속성을 사용하지만, method1에만 특정 트랜잭션 속성을 사용한다면, [4] 선언 후 특정 속성만 [5] 에 선언하면 된다.
- `@Transactional`은 일반적으로 클래스 단위에 선언한 후 세부 메소드에만 다시 선언하는 것이 좋다.
- `@Transactional` interface에 선언하는 것도 좋지만, 프록시 방식을 사용하지 않는 경우가 있기 때문에 타깃 클래스에 선언하는 것이 좋다.
  - Spring에서도 타깃 클래스에 선언하는 것을 권장한다.
  - https://stackoverflow.com/questions/3120143/where-should-i-put-transactional-annotation-at-an-interface-definition-or-at-a

#### 트랜잭션 어노테이션 사용을 위한 설정

- 스프링에서는 `@Transactional`을 이용한 트랜잭션 속성을 사용하기 위한 방법을 하나의 태그에 담아뒀다.

```xml
<tx:annotation-driven />
```

### 6.7.2 트랜잭션 어노테이션 적용

- 꼭 세밀한 설정이 필요할 때만 `@Transactional` 을 통해 트랜잭션을 적용하는 것은 아니다.
- 트랜잭션 설정이 직관적이고 간단하기 떄문에 `@Transactional` 을 자주 사용한다.
- `@Transactional` 은 무분별하게 사용하거나 자칫 빼먹을 수 있기 때문에 주의가 필요하다.

- xml을 이용한 설정
```xml
<tx:attributes>
  <tx:method name="get*" read-only="true"/>
  <tx:method name="*" />
</tx:attributes>
```

- `@Transactional`을 이용한 설정
```JAVA
@Transactional
public interface UserService {
  void add(User user);
  void deleteAll();
  void upgrade(User user);
  void upgradeLevels();

  @Transactional(readOnly=true)
  User get(String id);

  @Transactional(readOnly=true)
  List<User> getAll();
}
```

## 6.8 트랜잭션 지원 테스트

- 테스트할 때 하나의 트랜잭션으로 묶고 싶을 때에도 `@Transactional`을 이용하면 간단하게 적용할 수 있다.
- 테스트에서의 `@Transactional`은 테스트가 끝나면 자동으로 롤백시킨다.
  - `@Rollback` 어노테이션의 옵션을 false 로 설정하면, 에러가 발생하지 않는 이상 롤백하지 않는다.
  - `@Raollbak`은 메소드 단위로만 설정할 수 있다.
  - `@TransactionConfiguration` 을 이용하면, 선언된 클래스 내 모든 메소드에 일괄 적용할 수 있다.
- `@NotTransactional` 을 선언하면 트랜잭션을 시작하지 않는다.
  - 다만, 스프링 3.0 부터는 사라졌기 때문에 사용하지 않기를 권한다.
  - 트랜잭션을 시작하지 않고 싶다면, @Transactional(propagation=Propagation.NEVER)을 사용하는게 더 좋다.
- DB를 사용하는 통합 테스트는 가능한 한 롤백 테스트로 만드는게 좋다.
