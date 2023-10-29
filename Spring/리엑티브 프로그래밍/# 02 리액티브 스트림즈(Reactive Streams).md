# 02 리액티브 스트림즈(Reactive Streams)

## 2.1 리액티브 스트림즈란?

- 리액티브 스트림즈란 리액티브 코드를 작성하기 위한 라이브러리를 의미한다.
  - 즉, 데이터 스트림을 Non-Blocking 이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양을 의미한다.
- 리액티브 스트림의 예시로는 RxJava, Reactor(Spring Webflux), Akka Streams, Java 9 Flow API 등이 있다.

## 2.2 리액티브 스트림즈 구성요소

- 리액티브 스트림즈는 Publsiher, Subscriber, Subscription, Processor가 있다.
- Publisher: 데이털르 생성하고 통지한다.
- Subscriber: 구독한 Publisher로부터 통지된 데이터를 받아 처리하는 역할을 한다.
- Subscription: Publisher에 요청할 데이터의 개수를 지정하고, 데이터의 구독을 취소하는 역할을 한다.
- Processor: Publisher와 Subscriber의 기능을 모두 가지고 있다.

### 동작 방식

1. Subscriber는 Publsher를 구독한다.(subscribe)
2. Publisher는 데이터를 통지(발행, 게시, 방출) 할 준비가 되었음을 Susbscriber에게 알린다.(onSubscribe)
3. Publisher가 데이터를 통지할 준비가 되었다는 알림을 받은 Subscriber는 전달받기 원하는 데이터의 개수를 Publisher에게 요청한다.(Subscription.request)
    - Susbscirber가 Publisher에게 전달받기 원하는 데이터 개수를 별도로 통지하는 이유는 Subscriber와 Publisher가 별도의 스레드에서 동기적으로 처리되기 때문에 데이터 개수를 제어해야 하기 때문이다.
4. Publisher는 Subscriber로부터 요청받은 만큼의 데이터를 통지한다.(onNext)
5. Publisher와 Subscriber 간 데이터 통신, 수신, 요청 과정을 반복하다가 모든 데이터가 통지되면 데이터 전송이 완료되었음을 Subscriber에게 알린다.(onComplete)

## 2.3 코드로 보는 리액티브 스트림즈 컴포넌트

### Publisher

- Publisher는 subscribe 하나의 메소드만 존재하며, 매개변수로 전달 받은 Subscriber를 등록하는데 사용된다.
- publisher에 데이터를 처리할 Subscriber를 등록하는 형태다.

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

### Subscriber

- onSubscribe 메소드를 이용하여 구독 시작 시점에 어떤 처리를 할 지 결정한다.
  - Publisher에게 요청할 데이터의 개수를 지정하거나 구독을 해지한다.
- onNext 메소드를 통해 Publisher가 통지한 데이터를 처리한다.
- onError 메소드를 통해 에락 발생했을 때 처리하는 역할을 한다.
- onComplete 메소드를 통해 데이터 통지가 완료되었을 때 역할을 한다.

```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

### Subscription

- Susbcription은 Subscriber가 구독한 데이터 개수를 요청하거나 데이터 요청의 취소 등의 역할을 한다.
- request 메서드를 통해 데이터 개수를 요청할 수 있다.
- cancel 메서드를 통해 구독을 해지할 수 있다.

```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```

### 정리

- Publisher는 subscribe 메서드를 통해 데이터가 필요한 Subscriber를 저장한다.
- Publisher는 데이터 전달이 가능해질 때 Subscirber의 onSubscribe 메소드를 실행하며, 이때 매개변수로 Subsctiption을 전달한다.
- Subscribers는 onSubscribe 메소드를 통해 전달 받은 Subscription을 통해 데이터를 요청할 수 있다.
  - Subscription의 request 메소드를 통해 처리 가능한 데이터 개수를 전달하는 것이다.
- Subscriber가 Subscription의 request를 실행할 경우 매개변수로 전달한 개수 만큼 onNext를 실행한다.
  - 즉, Subscriber가 처리 가능한 개수 만큼한 onNext가 실행된다.
- 데이터의 주체와 상관 없이 결국은 Subscriber는 Subscription의 request를 통해 처리 가능한 데이터 개수를 전달하고, Subscription은 그 개수 만큼 Subscriber의 onNext를 실행한다.
- 결국은 데이터를 전달하는 것은 Subscription이 된다. Subscription의 request 메소드를 통해 데이터를 전달할 것인가를 결정하게 되는 것이고, 이때 일반적으로 onNext가 실행된다.
  - 그래서 일반적으로는 Publisher가 Subscription 생성 시 Subscriber와 데이터를 같이 전달한다.
  - Publisher는 데이터를 직접 전달하는 주체가 아니라, 데이터 발행을 관리자라고 보는게 맞다.
- Publisher가 Subscription에게 몇개의 데이터 전달해달라는 요청(reuqest)이 왔을 때 어떤 데이터를 누구에게 전달할 지 지정해준다.

## 2.4 리액티브 스트림즈 관련 용어 정리

- Signal: Publisher와 Subscriber 간 주고 받는 상호작용
- Demand: Subscriber가 Publisher에게 요청하는 데이터
- Emit: Publisher가 Subscriber에게 데이터를 전달하는 것.
- UpStream/DownStream: 데이터의 흐름
- Sequence: Publisher가 emit 하는 데이터의 연속적인 흐름. Sequence는 다양한 Operator로 구성된다.
- Operator: 리액티브 프로그래밍에서 사용하는 연산자. just, filter, map 등이 있다.
- Source: 최초의 데이터

## 2.5 리액티브 스트림즈의 구현 규칙

### Publisher

- Publisher가 Subscriber에게 보내는 onNext signal의 총 개수는 항상 해당 Subscriber의 구독을 통해 요청된 데이터의 총 개수보다 더 작거나 같아야 한다.
- Publisher는 요청된 것보다 적은 수의 onNext signal을 보내고 onComplete 또는 onError를 호출하여 구독을 종료할 수 있다.
- Publisher의 데이터 처리가 실패하면 onError signal을 보내야 한다.
- Publisher의 데이터 처리가 성공적으로 종료되면 onComplete signal을 보내야 한다.
- Publisher가 Subscriber에게 onError 또는 onComplete signal을 보내는 경우 해당 Subscriber의 구독은 취소된 것으로 간주되어야 한다.
- 일단 종료 상태 signal을 받으면(onError, onComplete) 더 이상 signal이 발생되지 않아야 한다.
- 구독이 취소되면 Subscriber는 결국 signal을 받는 것을 중지해야 한다.

### Subscriber

- Subscriber는 Publisher로부터 onNext siganl을 수신하기 위해 Subscription.request(n)를 통해 Demand signal을 Publsher에게 보내야 한다.
- Subscriber.onComplete() 및 Subscriber.onError(Throwable t)는 Subscription 또는 Publisher의 메서드를 호출해서는 안 된다.
- Subscriber.onComplete() 및 Subscriber.onError(Thorwable t)는 signal을 수신한 후 구독이 취소된 것으로 간주해야 한다.
- 구독이 더 이상 필요하지 않은 경우 Subscriber는 Subscription.cancel()을 호출해야 한다.
- Subscriber.onSubscribe()는 지정된 Subscriber에 대해 최대 한 번만 호출되어야 한다.

### Subscription

- 구독은 Subscriber가 onNext 또는 onSubscribe 내에서 동기적으로 Subscription.request 를 호출하도록 허용해야 한다.
- 구독이 취소된 후 추가적으로 호출되는 Subscription.request(long n)는 효력이 없어야 한다.
- 구독이 취소된 후 추가적으로 호출되는 Subsctiption.cancel()은 효력이 없어야 한다.
- 구독이 취소되지 않은 동안 Subscription.request(long n)의 매개변수가 0보다 작거나 같으면 IllegalArgumentException과 함께 onError signal을 보내야 한다.
- 구독이 취소되지 않은 동안 Subscription.cancel()은 Publisher가 Subscriber에게 보내는 signal을 결국 중지하도록 요청해야 한다.
- 구독이 취소되지 않은 동안 Subscription.cancel()은 Publsiher가 해당 구독자에게 대한 참조를 결국 삭제하도록 요청해야 한다.
- Subscription.cancel(), Subscription.request() 호출에 대한 응답으로 예외를 던지는 것을 허용하지 않는다.
- 구독은 무제한 수의 request 호출을 지원해야 하고 최도 2^63-1 개의 Demand를 지원해야 한다.