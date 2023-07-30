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