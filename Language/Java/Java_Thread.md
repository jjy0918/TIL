# 스레드

## 스레드 생성

자바에서 스레드는 Thread 클래스를 이용한다.
Thread 클래스는 기본적으로 Runnable 인터페이스 구현체이다.
Thread 클래스는 Runnable의 run 메소드에 있는 동작을 실행한다.
즉, Runnable의 run 메소드를 변경하면 Thread의 동작을 제어할 수 있다는 것이다.

기본적으로, Thread 생성 시 Runnable 구현체를 받는다.
Runnable 구현체를 받아 Thread 내부에 target이라는 이름으로 저장한 후, 실행 시 target.run을 실행한다.
즉, Runnable 구현체를 만들고 그 값을 Thread의 생성자로 넘기면 스레드의 동작을 제어할 수 있다.

```java

Thread t = new Thread(new Runnable() {
    @Override
    public void run() {
        // 스레드에서 수행할 동작
    }
});

```

Runnable은 추상 함수가 하나인 Functional Interface이기 때문에 람다를 적용할 수 있다.

```java

Thread t = new Thread(() -> {
    // 스레드에서 수행할 동작
});

```

## 동기화

멀티 스레드를 다루면서, 당연하게 등장하는 문제는 공유 자원의 동기화 문제이다.
자바에서는 공유 자원에 대한 문제를 해결하기 위해 synchronized 키워드를 이용하면 된다.
synchronized를 붙인 메소드에는 하나의 스레드만 접근할 수 있다.
또한, synchronized는 메소드뿐만 아니라 공유 자원 자체에도 사용할 수 있다.

```java

public class SyncClass {
    public static synchronized void test() {
        System.out.println(Thread.currentThread().getName());
        Thread.sleep(3000);
    }
}

-----

Thread t1 = new Thread(() -> SyncClass.test());
Thread t2 = new Thread(() -> SyncClass.test());

t1.start();
t2.start();

```
- 이 예시의 경우 Thread-0이 먼저 실행된다면, Thread-0이 출력된 후 3초 뒤 Thread-1이 출력된다.
- SyncClass라는 하나의 객체의 test 라는 메소드에는 하나의 스레드만 접근이 가능하다.

당연하게, 객체가 다르다면 다른 메소드 접근이 되기 때문에 동시에 실행이 가능하다.

## 스레드 상태

스레드 상태를 제어하는 메소드는 여러 가지가 존재한다.
아래 예시뿐만 아니라 wait, sleep, notify 등이 존재한다.

```java

Thread t = new Thread();

// 스레드를 종료한다.
t.interrupt();

// 다른 스레드 대기
// join을 실행한 스레드는 join 스레드가 종료될 때까지 대기한다.
// 여기서는 t스레드가 종료될때까지 t.join을 실행한 스레드는 대기한다.
t.join(); 

// 다른 스레드에게 양보
// 스레드는 기본적으로 돌아가며 진행되기 때문에 양보가 가능하다.
t.yield();


```

## 데몬 스레드

데몬 스레드 설정을 통해, 주 스레드와 부 스레드를 설정할 수 있다.
주 스레드가 종료되면, 데몬 스레드로 설정한 부 스레드들은 종료된다.
데몬 스레드를 만들기 위해서는 주 스레드에서 부 스래드의 setDaemon을 true로 설정해주면 된다.

## 스레드 그룹

여러 스레드를 하나의 그룹으로 묶어서 처리할 수 있다.

## 스레드 풀

스레드 풀을 사용하여 무한 스레드 폭증을 막을 수 있다.
스레드 풀을 통해 실제 동작하는 스레드의 개수를 제한하는 것이다.
제한된 개수 만큼만 스레드 큐에서 작업을 받아 수행한다.

ExecutorService의 구현체인 Executors 클래스를 이용하면된다.
Executors 클래스는 static으로 메소드로 ExecutorService의 구현체를 리턴한다.

Executors.newCachedThreadPool()은 초기 0개의 스레드로 시작하지만, 작업 개수가 많아지면 계속 늘어난다(최대 Int).
Executors.newFixedThreadPool(int nThreads)은 nThread 개수 만큼한 제한적으로 스레드를 생성한다.

Executors를 사용하지 않고 싶다면, ThreadPoolExecutor를 통해 값을 지정하여 생성할 수 있다.

### 작업 생성

스레드 풀에서 작업은 일반 Thread와 같이 Runnable을 구현하거나 Callable을 구현하여 진행한다.

Runnable과 Callable의 차이점은 리턴값이 있느냐 없느냐의 차이점이다.
Runaable은 리턴 값이 존재하지 않고, Callable은 리턴 값이 존재한다.

ExceutorService는 Runnable을 파라미터로 받아 실행하는 excute 메소드와 Runaable 또는 Callable을 받는 submit 메소드가 존재한다.

excute 리턴 값은 void 이지만, submit의 경우 Runnable이라고 하더라도 Future을 리턴하여 리턴 값을 저장할 수 있다.
excute(Runnable r) 인 경우 get() 시 null을 리턴하고, excute(Runnable r, T t)인 경우 T타입을 리턴한다.
일반적으로 excute(Runnable r)의 경우 get()의 값이 null인지를 통해 정상 동작했는지 체크한다.

submit 메소드 실행 결과로 받은 Future은 바로 값을 리턴하는 것이 아니라, 스레드 실행 후 값을 리턴하기 때문에 블록킹 후 리턴 된다.
즉, Future의 get()은 블록킹이 된다는 것이다. 그렇기 때문에 Future의 get() 은 주의해서 실행해야 한다.
