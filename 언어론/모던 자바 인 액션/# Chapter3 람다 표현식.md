# Chapter3 람다 표현식

## 3.1 람다란 무엇인가?

- 람다 표현식은 메서드로 전달할 수 있는 익명 함수를 단순화한 것이다.
- 람다식에는 이름이 없지만, 파리미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트를 가질 수 있다.
- 람다의 특징
  - 익명: 보통의 메서드와 달리 이름이 없다.
  - 함수: 메서드처럼 특정 클래스에 종속되지 않으므로 함수라 부른다. 하지만 메서드 처럼 파리미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트를 가질 수 있다.
  - 전달: 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
  - 간결성: 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

```java
// 기존 코드
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
};

// 람다를 이용한 코드
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

- 람다 표현식은 파라미터, 화실표, 바디로 이루어진다.
  - 파라미터: (Apple a1, Apple a2)
    - 구현하는 메서드의 파라미터
  - 화살표: ->
    - 람다의 파리미터 리스트와 바디를 구분
  - 람다 바디: a1.getWeight().compareTo(a2.getWeight())
    - 람다의 반환값에 해당하는 표현식

```java
// String을 받아 int를 반환
(String s) -> s.length();

// int 파라미터 두 개를 받아 void 리턴.
(int x, int y) -> { 
    System.out.println("RESULT:");
    System.out.println(x + y);
}
```

## 3.2 어디에, 어떻게 람다를 사용할끼?

- 람다는 함수형 인터페이스라는 문맥에서 사용한다.

### 3.2.1 함수형 인터페이스

- 함수형 인터페이스란 오직 하나의 추상 메서드만 가지고 있는 인터페이스를 말한다.
  - 디폴트 메서드가 있더라도 추상 메서드가 하나면 함수형 인터페이스다.
- Compartor, Runnable, Predicate 등이 자바에서 기본적으로 제공하는 함수형 인터페이스다.
- 람다를 이용하여 구현할 경우, 람다식을 통해 구현한 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.

```java
public static void process(Runnable r) {
    r.run();
}

Runnable r = () -> System.out.println("Hello World");

process(r);
```

### 3.2.2 함수 디스크립터

- 함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.
- 람다 표현식의 시그니처를 서술하는 메서드를 함수 디스크립터라고 부른다.

```java
public interface Runnable {
    void run(); // 인터페이스의 추상 메서드 시그니처 -> 매개변수는 없고 void를 반환한다.
}

Runnable r = () -> {};  // 매개변수는 없고 void를 반환하는 함수 디스크립터
```

- 컴파일러는 람다 표현식을 함수형 인터페스의 추상 메서드와 같은 형식으로 취급한다.
- `@FunctionalInterface` 가 선언된 인터페이스는 하나의 추상 메서드만 가질 수 있다. 두 개 이상의 추상 메서드가 있다면 컴파일 에러가 발생한다.

## 3.3 람다 활용: 실행 어라운드 패턴

- 실행 어라운드 패턴이란 open -> execute -> close 과정에서 execute를 기준으로 open, close 과정이 둘러쌓여 있는 패턴을 말한다.
  - ex) 파일 open -> 파일 처리 -> 파일 close

## 3.3.1 1단계: 동작 파라미터화를 기억하라.

- 파일을 한 줄 읽어 반환하는 코드를 두 줄 읽거나 자주 사용되는 단어를 반환하는 등 여러 동작으로 변경하려면 동작 파라미터를 사용하면 된다.
- 기존의 설정, 정리 과정은 재사용하고, processFile 메서드만 다른 동작을 수행하도록 명령할 수 있으면 된다.

### 3.3.2 2단계: 함수형 인터페이스를 이용해 동작 전달

- 함수형 인터페이스 자리에 람다를 사용할 수 있다.

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

public String processFile(BufferedReaderProcessor p) throws IOException {
    ...
}
```

### 3.3.3 동작 실행

- BufferedReaderProcessor에 정의된 메서드의 시그니처와 일치하는 람다를 전달할 수 있다.
- processFile 바디 내에서는 전달받은 BufferedReaderProcessor 객체의 process를 호출한다.

```java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br);
    }
}
```

### 3.3.4 람다 전달

- 람다를 이용하여 동작을 processFile 메서드로 전달할 수 있다.

```java
String oneLine = processFile((BufferedReader br) -> br.readLine());
String oneLine = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 3.4 함수형 인터페이스 사용

- 함수형 인터페이스는 오직 하나의 추상 메서드를 지정한다.
- 함수형 인터페이스의 추상 메서드는 람다 표현식의 시그니처를 묘사한다.
- 함수형 인터페이스의 추상 메서드 시그니처는 함수 디스크립터라고 한다.
- 자바에서는 기본적으로 제공하는 함수형 인터페이스가 있다.
  - Comparable, Runnable, Callable, jata.util.function 패키지 등
  - java에서 기본 제공하는 함수형 인터페이스는 checked Exception을 던지지 않기 때문에 checked Exception 은 내부에서 try~catch로 처리해야 한다.

### 3.4.1 Predicate

- `java.util.function.Predicate<T>` 인터페이스는 `test` 라는 추상 메서드를 정의하며, test는 T를 매개변수로 받아 boolean을 리턴한다.

### 3.4.2 Consumer

- `Consumer<T>` 인터페이스의 `accept` 메서드는 T를 매개변수로 받아 void를 리턴한다.
- 어떤 동작을 수행하고 싶을 때 사용한다.

### 3.4.3 Function

- `Function<T, R>` 인터페이스의 `apply` 메서드는 T를 받아 R을 반환한다.
- 입력을 출력으로 매핑하는 람다를 정의할 때 사용한다.

### 3.4.4 기본형 특화

- 자바에서는 기본형 타입과 참조형 타입으로 구분된다.
- 제네릭 파라미터는 참조형 타입만 사용할 수 있기 때문에 기본형 타입은 박싱이 필요하다.
  - 자바에서는 기본적으로 기본형 <-> 참조형 타입(Wrapper) 변환을 자동으로 해주는 `오토 박싱`을 제공한다.
  - 기본형 -> 참조형: 박싱
  - 참조형 -> 기본형: 언박싱
- 박싱과 언박싱이 일어나게되면, 불필요한 비용이 소모되기 떄문에 이러한 비용을 최소화하기 위한 `기본형 특화 함수형 인터페이스`를 제공한다.
  - IntPredicate, DoublePredicate, LongBinaryOperator 등

## 3.5 형식 검사, 형식 추론, 제약

### 3.5.1 형식 검사

- 람다가 사용되는 context를 이용하면 람다의 형식을 추론할 수 있다.
- 어떤 context에서 기대되는 람다 표현식의 형식을 대상 형식이라고 한다.
- 람다가 표현하는 대상 형식(Predicate<T>의 test 메서드)과 람다 형식의 시그니처가 일치하는 확인한다.

### 3.5.2 같은 람다, 다른 함수형 인터페이스

- 대상 형식이라는 특징 때문에 같은 람다 표현식이라 하더라도 다른 함수형 인터페이스로 사용될 수 있다.
- (Integer i) -> True 람다가 Predicate<Integer> 일수도, IntPredicate 일수도, 직접 구현한 인터페이스 일수도 있다.
- 람다 바디에 일반 표현식이 있으면, void를 반환하는 함수 디스크립터와 호환된다.
  - s -> list.add(s); 람다 표현식이 있을 때 Predicate에서 사용 가능하며, void 반환하는 Consumer에서도 사용 가능하다.

### 3.5.3 형식 추론

- 컴파일러는 대상 형식을 통해 함수형 인터페이스를 추론할 수 있으며, 이를 통해 람다 시그니처를 추론하게 된다.
- 결과적으로 컴파일러는 람다 표현식을 통해 시그니처 확인이 가능하여 파라미터 형식을 생략할 수 있다.
- 같은 시그너처를 가지고 있는 메서드가 오버로딩 되어 있다면 명시해주어야 한다.

```java
public String test() {
    t1(() -> 1);    // 컴파일 에러
    t1((Callable<Integer>)() -> 1);    // 정상 동작
    t1((Callable<Supplier>)() -> 1);    // 정상 동작
    
    return "OK";
}
public void t1(Callable<Integer> t) {
    
}
public void t1(Supplier<Integer> t) {
    
}
```

### 3.5.4 지역 변수 사용

- 람다 표현식은 익명 함수처럼 자유 변수(파라미터가 아닌 외부에 정의된 변수)를 활용할 수 있다.
- 람다가 자유 변수를 사용하기 위해서 `람다 캡처링(capturing lambda)`이라는 동작을 한다.
- 람다 캡처링에서 `인스턴스 변수`와 `정적 변수`는 자유롭게 가능하지만, `지역 변수`는 final 이거나 실질적 final 이어야 한다.
  - 실질적 fianl이란 해당 변수가 한 번만 할당되어야 한다는 것이다.
- 지역 변수만 final 이어야 하는 이유는 실제로 람다 캡처링된 변수 접근 시 지역 변수가 아니라 지역 변수의 복사본에 접근하기 떄문이다.
  - 지역 변수는 스택에 할당이 되기 때문에 해당 영역이 끝나면 사라지게 되기 때문에 복사하여 사용한다.
  - 복사 값을 사용하여 값이 새로 할당되면 잘못된 값을 사용할 수 있기 때문에 final이거나 실질적 final 이어야 한다.

## 3.6 메서드 참조

- 메서드 참조를 이용하면 기존 메서드 정의를 재활용하여 람다처럼 전달할 수 있다.

### 3.6.1 요약

- 메서드 참조는 특정 메소드를 호출하는 람다의 축약형이라고 할 수 있다.
- 명시적으로 메서드 명을 참조함으로써 가독성을 높일 수 있다.

#### 메서드 참조 만드는 방법

- 정적 메서드 참조
  - (args) -> ClassName.staticMethod(args)
  - ClassName::staticMethod
  - 람다의 매개변수가 정적 메서드의 매개변수로 들어간다.
- 다양한 형식의 인스턴스 메서드 참조
  - (arg0, rest) -> arg0.instanceMethod(rest)
  - ClassName::instanceMethod
  - 람다의 첫 번째 매개변수가 인스턴스가 되고, 나머지 매개변수가 인스턴스 메서드의 매개변수로 들어간다.
- 기존 객체의 인스턴스 메서드 참조
  - (args) -> expr.instanceMethod(args)
  - expr::instanceMethod
  - 람다의 매개변수가 인스턴스의 매개변수로 들어간다.
- 컴파일러는 람다 표현식 검사와 동일하게 메서드 참조가 주어진 경우 해당 메서드 시그니처가 함수형 인턴스와 호환하는지 확인한다.

### 3.6.2 생성자 참조

- 생성자도 생성자 참조를 이용하여 간단하게 사용할 수 있다.
- 생성자 참조는 정적 메서드 참조와 만드는 방법이 비슷하다.
  - ex) Supplier<Apple> c1 = () -> new Apple(); ==> Supplier<Apple> c1 = Apple::new; 
  - ex) Function<Integer, Apple> c2 = (weight) -> new Apple(weight); ==> Function<Integer, Apple> c2 = Apple::new;

## 3.8 람다 표현식을 조합할 수 있는 유용한 메서드

- 자바8 API의 몇몇 함수형 인터페이스는 다양한 유틸리티 메서드를 포함한다.
- 유틸리티 메서드를 통해 람다식을 조합하여 다양한 람다식을 만들 수 있다.
- 유틸리티 메서드는 디폴트 메서드를 사용한다.

### 3.8.1 Comparator 조합

```java
// 기본
Compartor<Apple> c = Comparator.comparing(Apple::getWeight);

// 역정렬
inventory.sort(comparaing(Apple::getWeight).reversed());    // 무게순으로 내림차순

//Comparator 연결 --> 두 개가 같을 경우 비교
inventory.sort(comparaing(Apple::getWeight)
    .reversed() // 무게순으로 내림차순
    .thenComparing(Apple::getCountry)); // 두 사과의 무게가 같으면 국가별로 정렬    
```

### 3.8.2 Predicate 조합

```java
Predicate<Apple> redApple = (Apple::isRed);
Predicate<Apple> notRedApple = redApple.negate();   // 기존 Predicate 반전 객체
Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150); // 빨간 사과이면서 150 초과인 사과 조회
Predicate<Apple> redAndHeavyAppleOrGreen = redApple
    .and(apple -> apple.getWeight() > 150)
    .or(apple -> Green.equals(a.getColor()));   // 빨간색이면서 150 초과이거나 초록색 사과
```

### 3.8.3 Function 조합

- andThen은 순차적으로 실행한 결과를 전달한다.
  - f.andThen(g) --> g(f(x))
- compose는 역으로 실행한 결과를 전달한다.
  - f.compose(g) --> f(g(x))