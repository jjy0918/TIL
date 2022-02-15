# 7. Repeat

## 7.1 Repeat Template

스프링 배치에서는 반복처리 기능을 제공한다.

`RepeatOperation` 인터페이스를 통해 프레임워크에 얼마나 반복해야 하는지 알려준다.

`RepeatOperations` 의 가장 간단한 구현체는 `RepeatTemplate`가 있다.

`RepeatOperations` 의 인터페이스는 다음과 같다.

```java
public interface RepeatOperations{
	RepeatStatus iterate(RepeatCallback callback) throws RepeatException;
}
```

`RepeatCallback` 은 반복할 비즈니스 로직을 구현하는 인터페이스이다.

```java
public interacae RepeatCallback{
	RepeatStatus doInIteration(RepeatContext context) throws Exception;
}
```

`RepeatCallback` 의 `doInIteration` 를 종료할 때까지 반복한다.

`RepeatOperations`의 리턴 값인 `RepeatStatus` 는 `RepeatStatus.CONTINUABLE` 과 `RepeatStatus.FINISHED`를 포함한 열거형이다.

이 열거형은 더 할 일이 남아 있는지를 나타낸다.

`RepeatStatus.FINISH` 를 전달하면, 더 이상 처리할 작업이 없다는 것이다.

구현 예시는 다음과 같다.

```java
RepeatTemplate template = new RepeatTemplate(); // RepeatOperations의 구현체인 RepeatTemplate

template.setCompletionPolicy(new SimpleCompletionPolicy(2));

template.iterate(new RepeatCallback() {
	public RepeatStatus doInIteration(RepeatContext context) {
		return RepeatStatus.CONTINUABLE;
	}
});
```

### 7.1.1 RepeatContext

`RepeatCallback` 은 파라미터로 `RepeatContext` 를 받는다.

`RepeatContext` 는 실행이 반복되는 동안 일시적으로 데이터를 저장할 수 있도록 도와준다.

당연히 `RepeatOperations`의 `iterate` 가 종료되면, `RepeatContext` 의 데이터는 사라진다.

반복 호출을 중첩한다면, `RepeatContext` 는 부모 컨텍스트를 가진다.

그렇기 때문에 `iterate` 를 호출하는 동안 공유하고 싶은 데이터를 부모 컨텍스트에 저장할 수 있다.

### 7.1.2 RepeatStatus

`RepeatStatus` 는 스프링 배치의 처리 상태를 전달할 수 있는 열거형이다.

`RepeatStatus` 값은 안에 있는 `and()` 메소드를 이용하여 AND 연산으로 조합을 할 수 있다.

## 7.2 Completion Policies

`RepeatTemplate` 의 `iterate` 메소드 반복 중단 여부는  `RepeatContext` 의 `CompletionPolicy` 가 결정한다.

`RepeatTemplate` 는 현재 policy를 기준으로 `RepeatContest` 를 생성하여 반복 시 `RepeatCallback` 에 전달한다.

`RepeatCallback` 의 `doInIteration` 메소드 완료 시 `RepeatTemplate` 는 `CompletionPolicy` 를 호출하여 상태를 업데이트한다.

스프링 배치에서는 `CompletionPolicy` 구현체인 `SimpleCompletionPolicy` 를 제공한다.

`SImpleCompletionPolicy` 는 고정된 횟수 만큼 반복한다.(`RepeatStatus.FINISHED` 리턴 시 조기 종료된다.)

## 7.3 Exception Handling

`RepeatTemplate` 가 `RepeatCallback` 을 실행하면서, 예외가 발생하는 경우 `ExceptionHadler` 를 참고하여 처리한다.

스프링 배치에서는 `ExceptionHandler` 를 구현한 `SimpleLimitExceptionHadler` 와 `RethrowOnThresholdExceptionHadler` 를 제공한다.

`SimpleLimitExceptionHadler` 는 limit, exception 타입 프로퍼티가 있어서 exception이 발생할때마다 타입을 비교한다.

주어진 exception을 비롯한 하위 클래스가 발생하면, 그 횟수를 카운팅하고 limit에 도달하면 예외를 던진다.

설정한 exception 외 다른 타입의 exception은 전부 던진다.

`SimpleLimitExceptionHandler` 의 `useParent` 설정을 true로 변경하게 되면, 반복 처리 안에 있는 형제 컨텍스트와 limit을 공유한다.

(기본 값은 false 이다.)

```java
public interface ExceptionHandler {
	void handlerException(RepeatContext context, Throwable throwable) throws Throwable;
}
```

## 7.4 Listeners

스프링 배치의 `RepeatListener` 인터페이스를 이용하여 반복 처리하는 횡단 관심사를 처리할 수 있다.

`RepeatTemplate` 에 `RepeatListener` 를 등록하면, `RepeatContext`, `RepeatStatus` 와 함께 콜백을 받을 수 있다.

`RepeatListner` 의 `open`, `close` 는 반복 전후로 콜백을 받고, `before`, `after`, `onError` 는 `RepeatCallback`을 사용할 때마다 반복된다.

`RepeatListner` 가 둘 이상인 경우 리스트 안에 저장하는 순서가 정해져 있다.

`open`, `before` 메소드는 동일한 순서로 호출되고, `before`, `onError`, `close` 는 역순으로 호출된다.

```java
public interface RepeatListener {
    void before(RepeatContext context);
    void after(RepeatContext context, RepeatStatus result);
    void open(RepeatContext context);
    void onError(RepeatContext context, Throwable e);
    void close(RepeatContext context);
}
```

## 7.5 Parallel Processing

`RepeatOperations` 구현체의 콜백을 병렬로 처리할 수 있다.

스프링의 `TaskExecutor` 를 사용하여 `RepeatCallback` 을 실행하는 `TaskExecutorRepeatTemplate` 사용하여 병렬 처리를 구현할 수 있다.

`TaskExecutor` 의 디폴트는 `SynchronousTaskExecutor` 를 사용하도록 설정되어 있고, 동일한 쓰레드로 전체 반복 처리를 실행하게 되면 문제가 발생할 수 있다.

## 7.6 Declarative Iteration

스프링 배치에서 AOP 인터셉터를 지원한다.

`RepeatOperationsInterceptor` 는 메소드 호출을 가로채서 `RepeatTemplate` 에서 제공하고 있는 `CompletionPolicy` 에 따라 반복 처리를 한다.

가로챈 메소드도 리턴값이 존재하는데, void 반환 시 항상 `RepeatStatus.CONTINUABLE` 을 리턴한다. 

즉, `CompletionPolicy` 가 종료 시점을 알려주지 않으면 무한 루프에 빠질 수 있다.

void가 아니라 null을 리턴하는 경우 `RepeatStatus.FINISHED` 를 리턴한다.

반복을 종료하기 위해서는 null을 리턴하거나 `RepeatTemplate` 의 `ExceptionHadler` 가 다시 던진 exception을 던져야 한다.

```java
@Bean
public MyService myService() {
	ProxyFactory factory = new ProxyFactory(RepeatOperations.class.getClassLoader());
	factory.setInterfaces(MyService.class);
	factory.setTarget(new MyService());

	MyService service = (MyService) factory.getProxy();
	JdkRegexpMethodPointcut pointcut = new JdkRegexpMethodPointcut();
	pointcut.setPatterns(".*processMessage.*");

	RepeatOperationsInterceptor interceptor = new RepeatOperationsInterceptor();

	((Advised) service).addAdvisor(new DefaultPointcutAdvisor(pointcut, interceptor));

	return service;
}
```