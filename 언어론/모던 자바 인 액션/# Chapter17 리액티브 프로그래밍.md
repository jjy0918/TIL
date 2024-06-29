# Chapter 17 리액티브 프로그래밍

## 17.1 리액티브 매니패스토

- 리액티브 매니패스토는 리액티브 애플리케이션과 시스템 개발의 핵심 원칙을 정의한다.
- 반응성(responsive): 리액티브 시스템은 빠르며 일정하고 에상할 수 있는 반응 시간을 제공한다.
- 회복성(resilient): 여러 컴포넌트의 시간과 공간 분리를 통해 장애가 발생해도 시스템은 반응할 수 있다.
- 탄력성(elastic): 무서운 작업 부하가 발생하면 자동으로 관련 컴포넌트에 할당된 자원 수를 늘린다.
- 메시지 주도(Message-driven): 컴포넌트 끼리 경계를 명확하게 정의하고, 메시지를 통해 통신이 이루어진다.

### 17.1.1 애플리케이션 수준의 리액티브

- 애플리케이션 수준의 리액티브에서는 비동기로 작업을 수행하여 멀티 코어 CPU 사용을 극대화시킬 수 있다.
- 이러한 방식은 일반적인 스레드 보다 가벼우며, 개발자 입장에서는 추상 수준을 높일 수 있어 비즈니스 구현에만 집중할 수 있다.
- 리액티브 시스템을 사용하는 RxJava, Akka 등은 별도의 스레드 풀을 사용하기 떄문에 이러한 메인 스레드에서는 I/O 등의 블록킹 작업은 수행하면 안된다.
  - 블록킹 작업은 별도의 스레드를 할당 받아 사용하는 것이 좋다.
  - 스레드가 블록된다는 측면에서는 동일하지만, I/O 등의 경우에는 단순 대기이기 때문에 CPU 사용하는 블록킹 작업과는 다르다.
  - I/O에 의한 대기하는 스레드를 별도로 두면, 기존의 메인 스트림을 처리(CPU 작업)할 수 있게 된다.

### 17.1.2 시스템 수준의 리액티브

- 리액티브 시스템은 여러 애플리케이션이 한 개의 일관적인, 회복할 수 있는 플랫폼을 구성할 수 있게 해주며, 이 중 하나가 실패하더라도 운영이 가능한 아키텍처다.
- 리액티브 시스템은 데이터 스트림 기반으로 연산을 수행하기 떄문에 이벤트 주도라고 할 수 있다.
- 리액티브 시스템은 컴포넌트를 완전히 고립시켜 결합되지 않도록 하기 떄문에 장애와 높은 부하에서도 반응성을 유지할 수 있다.
  - 장애를 고립시키기 떄문에 장애가 다른 컴포넌트로 전파되지 않도록 한다.
- 리액티브 시스템은 위치 투명성 덕분에 시스템을 복제할 수 있으며, 현재 작업 부하에 따라 애플리케이션을 확장할 수 있다.

## 17.2 리액티브 스트림과 플로 API

- 리액티브 프로그래밍은 리액티브 스트림을 사용하는 프로그래밍으로, 무한의 비동기 데이터를 순서대로 블록하지 않는 역압력을 통해 처리한다.
- 자바9 에서는 Flow API를 통하여 리액티브 프로그래밍을 지원한다.

### 17.2.1 Flow 클래스 소개

- 리액티브 스트림 프로젝트의 표준에 따라 자바9 에서는 프로그래밍 발생-구독 모델을 지원한다.
- `Publisher`, `Subscriber`, `Subscription`, `Processor`
- Publisher가 이벤트를 발생하면, Subscriber가 한 개 또는 여러 개 이벤트를 소비하고 Subscription은 이를 관리한다.
- Publisher는 반드시 Subscription의 request 메서드에 정의된 개수 이하의 요소만 Subscriber에 전달해야 한다.
- Subscriber는 몇 개의 요소를 받아 처리할 수 있음을 Publisher에 알려야 한다.
- Publisher와 Subscriber는 정확하게 Subscription을 공유해야 하며 각각이 고유한 역할을 수행해야 한다.
  - onSubsribe와 onNext 메서드에서 Subscriber는 request를 동기적으로 호출할 수 있어야 한다.
  - Subscription.cancel()은 몇 번 호출해도 한 번 호출한 것과 같은 효과를 가져야 한다.
- Processor는 단순 Publisher와 Subscriber를 상속 받는 인터페이스다.
  - 주로 데이터를 변환할 때 사용한다.

## 17.3 리액티브 라이버러리 RxJava 사용하기

- RxJava는 넷플릭에서 구현한 리액티브 애플리케이션 구현을 위한 라이브러리다.
- RxJava는 자바9의 Flow가 나오기 전에 사용되었고, Flow가 출시되면서 이를 지원하기 위해 RxJava 2.0 이 개발되었다.
- RxJava에서는 Flow.Publisher를 구현하기 위해 두 클래스를 제공한다.
  - RxJava의 `Observable` 클래스는 기본적으로 역압력을 지원하지 않는 Publisher다.
  - RxJava에서는 역압력을 지원하기 위해 `Flowable` 클래스를 사용한다.
    - Flowable은 Publisher를 구현한다.
  - 마우스 움직임 등 역압력을 적용하지 않아도 되는 부분은 `Observable`을 사용하고, 역압력이 필요하다면 `Flowable`을 사용한다.
  - Flow에서는 request(Long.MAX_VALUE)를 통해 역압력을 제거할 수는 있다.

### 17.3.1 Observable 만들고 사용하기

- Observable, Flowable 클래스는 다양한 종류의 리액티브 스트림을 만들 수 있도록 팩토리 메서드를 제공한다.
- Observable은 Publisher 역할을 한다.
- Observer는 Subscriber 역할을 한다.
  - Subscriber와 다른 점은 다르게 Subscription 대신 Disposable을 받는다.
- Observer는 Flow 보다 조금 더 유연하다.
  - ex) Subscriber의 onSubscriber만 구현해도 되도록 구현되어 있다.

```java
public static Observable<TempInfo> getTemperature(String town) {
    return Observable.create(emitter -> {   // ObservableOnSubscribe 구현
        Observable.interval(1, TimeUnit.SECONDS)    
            .subscribe(i -> {
                if(...)
            })
    })
}

```

- `Observable.create` 는 `ObservableOnSubscribe`를 매개변수로 받음.
- `ObservableOnSubscribe` 는 `void subscribe(@NonNull ObservableEmitter<T> emitter)` 매서드를 가진 인터페이스
- `ObservableEmitter` 는 `onSubscribe` 메서드를 제외한 Observer와 동일
- `Observable`는 역압력을 지원하지 않기 때문에 Subscription을 대신하는 Disposable에는 request 메서드가 필요 없다.
  - 역압력을 지원하기 위해서는 수신하는 Observer에서 request를 이용하여 처리 가능한 개수 만큼만 요청할 수 있어야 한다.
  - Observable은 송신측에서 데이터만 밀어 넣는다.

### 17.3.2 Observable을 변환하고 합치기

- 리액티브 스트림에서는 마블 다이어그램을 이용하여 문서화를 한다.
- map, filter 등을 통해 데이터를 처리할 수 있다.
- maerge를 통해 여러 Observable을 하나의 Observable로 합칠 수 있다.

## 17.4 마치며

- 리액티브 프로그맹의 기초 사상은 이미 20~30년 전에 수립되었지만, 데이터 처리량과 사용자 기대치 덕분에 최근에서야 인기를 얻고 있다.
- 리액티브 소프투에어가 지녀야 할 네 가지 관련 반응을 서술하는 리액티브 매니페스토가 리액티브 프로그밍 사싱을 공식화한다.
- 여러 애플리케이션을 통합하는 리액티브 시스템과 한 개의 애플리케이션을 구현할 때에 각각 다른 접근 방식으로 리액티브 프로그맹 원칙을 적용할 수 있다.
- 리액티브 애플리케이션은 리액티브 스트림이 전달하는 한 개 이상의 이벤트를 비동기로 처리함을 기본으로 전개한다.
- 리액티브 스트림은 비동기적으로 처리되므로 역압력 기법이 기본적으로 탑재되어 있다.
  - 역압력은 발행자가 구독자보다 빠른 속도로 아이템을 발행하므로 발생하는 문제를 방지한다.
- 자바 9 Flow API는 Publisher, Subscriber, Subscription, Processor 네 개의 핵심 인터페이스를 정의한다.
- 대부분의 상황에서는 이들 인터페이스를 직접 구현할 필요가 없으며 실제 이들 인터페이스는 리액티브 패러다임을 구현하는 다양한 라이브러리 공용어 역할을 한다.
- 가장 흔한 리액티브 프로그래밍 도구로 RxJava를 꼽을 수 있으며, RxJava는 Flow API의 기본 기능에 더해 다양한 강력한 연산자를 제공한다.
- 결국, 리액티브에서 `비동기`란 이벤트 들이 `비동기`로 처리될 수 있다는 것을 의미한다.
  - 즉, 해당 이벤트의 프로세스, 작업들은 동기 또는 비동기가 된다.
- 