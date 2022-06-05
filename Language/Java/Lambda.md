# 람다식

## 람다식이란?

- 자바8부터 람다식을 지원한다.
- 람다식을 이용하여 함수형 프로그래밍이 가능하도록 도와준다.
- 람다식은 코드가 간결해지고, 컬렉션의 요소를 필터링하거나 매핑하여 원하는 결과를 쉽게 집계할 수 있게 도와준다.
- 람다식은 런타임 시 익명 구현 객체를 생성한다.

## 람다식 문법

- 람다식은 `(타입 매개변수, ...) -> { 실행문; }` 형태로 구현할 수 있다.
- 매개변수가 하나라면, 괄호를 생략할 수 있지만, 매개변수가 없는 경우 괄호는 반드시 있어야 한다.
- 실행문이 하나라면 중괄호도 생략할 수 있다. 실행문이 리턴문만 있다면, return도 생략할 수 있다.

## 타겟 타입과 함수적 인터페이스

- 람다식은 인터페이스 변수에 대입된다.
- 인터페이스는 구현될 수 없기 떄문에, 람다식은 인터페이스를 구현한 객체를 생성하는 것이다.
- 람다식은 기본적으로 하나의 추상 메소드가 선언된 인터페이스에서만 사용할 수 있다.
  - 이러한 인터페잇르ㅡㄹ 함수적 인터페이스(funtional interface)라고 한다.
  - `@FunctionalInterface` 어노테이션을 선언하면, 두 개 이상의 추상 메소드가 선언되어 있는지 체크해준다.

## 클래스 멤버과 로컬 변수 사용

- 람다식에서는 클래스 멤버와 로컬 변수를 사용할 수 있다.
- 클래스 멤버는 제약 사항 없이 사용할 수 있지만, 로컬 변수의 경우 final을 붙인 경우에만 사용할 수 있다.
  - 람다식의 결과는 클래스이고, 결국 스레드에서도 사용할 수 있다.
  - 클래스의 멤버 변수는 힙 영역에 생성되기 떄문에 다른 스레드에서 접근할 수 있지만, 멤버 변수는 스택 영역이기 때문에 접근이 불가능하다.
  - 즉, 스택 영역의 값은 접근이 불가능하기 때문에 값의 일관성을 보장할 수 없게 된다.
  - 그렇기 때문에 final로 선언된 값을 사용하거나, 힙 영역에서 접근할수 있는 값만 사용이 가능한 것이다.

## 표준 API의 함수적 인터페이스

- 자바8부터는 java.util.function에 함수적 인터페이스 표준 API를 제공한다.
- 함수적 인터페이스를 이용하여 메소드나 생성자에 람다식을 사용할 수 있도록 한다.

### Consumer

- Consumer 인터페이스는 리턴 값이 없고 매개 변수만 있는 accept 메소드를 가지고 있다.
- `Consumer<T>`는 `void accept(T t)` 메소드를 가지고 있고, 객체 T를 받아 소비한다.
- `BiCounsumer<T, U`는 `void accpet(T t, U u)` 메소드를 가지고 있고, 객체 T와 U를 받아 소비한다.
- 그 외에도 DoubleConsumer, IntConsumer, LongConsumer, ObjDoubleConsumer등이 존재한다.

```java
Consumer<String> c = (text) -> System.out.println(text);
c.accept("테스트입니다.");
```

### Supplier

- Supplier 인터페이스는 매개변수가 없고 리턴 값만 있는 getXXX 메소드를 가지고 있다.
- `Supplier<T>`는 `T get()` 메소드를 가지고 있고, 객체 T를 리턴한다.
- `BooleanSupplier`는 `boolean getAsBoolean()` 메소드를 가지고 있고, boolean 값을 리턴한다.
- `DoubleSupplier`는 `double getAsDouble()` 메소드를 가지고 있고, double 값을 리턴한다.
- 그 외에도 IntSupplier, LongSupplier이 존재한다.

```java
IntSupplier s = () -> (int)Math.random() * 100;
int num = s.getAsInt();
```

### Function

- Function 인터페이스는 매개변수와 리턴값이 있는 appliyXXX 메소드를 가진다.
- `Function<T, R>`은 `R appliy(T t)` 메소드를 가지고 있고, 객체 T를 받아 객체 R을 리턴한다.
- `BiFunction<T, U, R>`은 `R appli(T t, U u)` 메소드를 가지고 있고, 객체 T, U를 받아 R을 리턴한다.
- `DoubleFunction<R>`은 `R apply(double value)` 메소드를 가지고 있고, double을 받아 R을 리턴한다.
- 그 외에도 IntFunction, IntToDoubleFunction, IntToLongFunction등이 있다.

```java
Function<Person, String> function = p -> p.getName();
p.apply(person)
```

### Operator

- Operator는 Function과 동일하게 매개변수와 리턴값이 있지만, Function과 다르게 매개변수와 리턴값의 타입이 동일하다.
- `UnaryOperator<T>`는 `T apply(T t)` 메소드를 가지고 있고, T를 받아 T를 리턴한다.
- `BinaryOperator<T>`는 `T apply(T t1, T t2)` 메소드를 가지고 있고, T와 T를 받아 T를 리턴한다.
- `IntUnaryOperator`는 `int applyAsInt(int i)` 메소드를 가지고 있고, int를 받아 int를 리턴한다.
- 그 외에도 IntBinaryOperator, DoubleBinaryOperator, DoubleUnaryOperator 등이 있다.

### Predicate

- Predicate는 매개변수와 boolean 리턴값이 있는 testXXX 메소드를 가진다.
-`Predicate<T>`는 `boolean test(T t)` 객체 T를 받아 boolean을 리턴한다,
- 그 외에도 BiPredicate, DoublePredicate, IntPredicate 등이 있다.

### andThen()과 compose()

- Consumer, Function, Operator 인터페이스는 andThen()과 compose()를 디폴트 메소드로 가지고 있다.
- andThen()과 compose()를 이용해, 리턴 결과를 매개 변수로 자연스럽게 넘길 수 있다.
- interfaceA.andThen(interfaceB)인 경우 interfaceA 처리 후 결과를 interfaceB에 넘기고 처리한 결과를 리턴한다.
- interfaceA.compose(interfaceB)인 경우 interfaceB 처리 후 결과를 interfaceA에 넘기고 처리한 결과를 리턴한다.
- Consumer, BiConsumer, DoubleConsumer, IntConsumer, LongConsumer는 andThen만 지원한다.
- Function은 andThen, compose 모두 지원한다.
- BiFunction은 andThen만 지원한다.
- BinaryOperator는 andThen만 지원한다.
- DoubleUnaryOperator, IntUnaryOperator, LongUnaryOperator는 andThen, compose 모두 지원한다.

```java
Function<Integer, Integer> plus = num -> num + 2;
Function<Integer, Integer>  multiple = num -> num * 10;

Function<Integer, Integer> andThen = plus.andThen(multiple);
Function<Integer, Integer> compose = plus.compose(multiple);

// (5 + 2) * 10
System.out.println(andThen.apply(5));

// (5 * 10) + 2
System.out.println(compose.apply(5));
```

### and(), or(), negate(), isEquals()

- Prediacte 종류의 인터페이스는 and, or, negate 디폴트 메서드를 가지고 있다.
  - and는 &&을 의미한다.
  - or은 ||을 의미한다.
  - negate는 !을 의미한다.
- isEquals는 Predicate의 Static 메소드로 제공되고, Predicate 구현체를 리턴한다.
  - 리턴된 구현체는 test 메소드를 이용하여 Object.equals로 비교한 결과를 리턴한다.

### minBy(), maxBy()

- BinaryOperator 인터페이스는 minBy, maxBy 정적 메소드를 제공한다.
- 이 메소드들은 매개값으로 Comparator를 받고 BinaryOperator<T>를 리턴한다.

## 메소드 참조

- 메소드 참조를 통해 람다식에서 불필요한 매개 변수를 제거할 수 있다.
- 기존에 정의되어 있는 매소드와 매개변수, 리턴 값이 같은 경우 해당 메소드를 그대로 대입하는 것이다.
  - Math의 max 메소드의 경우 두 매개 변수를 받고, 그 중 큰 값을 리턴한다.
  - (l, r) -> Math.max(l, r);
  - Math::max;

### 외부 메소드 참조

- 정적 메소드의 경우 `클래스::메소드` 형태로 작성한다.
- 인스턴스 메소드의 경우 `참조변수::메소드` 형태로 작성한다.

### 매개변수의 메소드 참조

- 매개변수의 매소드를 사용하는 경우에도 메소드 참조를 이용하면 간단하게 서술할 수 있다.
- (a, b) -> { a.function(b) } 의 경우
- a클래스::function 으로 작성할 수 있다.

### 생성자 참조

- 생성자 역시 메소드 참조를 통해 간단하게 사용할 수 있다.
- (a, b) -> { return new 클래스(a, b) } 의 경우
- 클래스:new 로 작성할 수 있다.

