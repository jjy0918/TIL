# Chapter2 동작 파라미터화 코드 전달하기

- 동적 파라미터화는 아직 결정되지 않는 코드 블록을 동적으로 받아 처리할 수 있도록 하는 것이다,
- 동적 파라미터화를 이용하면 자주 바뀌는 요구사항을 효과적으로 대응할 수 있다.
- 컬렉션 처리 시 `어떤 동작`을 수행할 수 있으며, 관련 작업 끝낸 후 `어떤 다른 동작`을 수행할 수 있다고 명시한 후 어떤 동작들을 동적으로 받는 것이다.
- 자바8에서는 기존에 번거롭게 구현해야 했던 동작 파리미터화를 람다식을 이용하여 간결하게 만들 수 있도록 제공해준다.

## 2.1 변화하는 요구사항에 대응하기

- 변화하는 요구사항 대응 예시

### 2.1.1 첫 번째 시도: 녹색 사과 필터링

```java
enum Color { RED, GREEN }

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if(Green.equals(apple.getColor())) {
            result.add(apple);
        }
    }

    return result;
}
```

- 위 코드는 apple 목록 중 greenApple만 필터링하여 가져오는 로직이다.
- 이때 greenApple이 아니라 redApple을 가져오고 싶다면 위 코드를 복사하여 Green.equals(apple.getColor()) 부분만 변경하는 메서드를 새로 구현해야 한다.
- 지금은 색이 두 개라서 그나마 간단하지만, 색이 늘어난다면 비슷한 코드가 반복되어 복사, 수정 될 것이다.
- 이러한 상황처럼 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화할 수 있다.

### 2.1.2 두 번째 시도: 색을 파라미터화

- 색과 관련된 부분을 파리미터화 할 수 있도록 메서드에 파라미터를 추가하면 유연하게 대응할 수 있다.

```java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if(apple.getColor().equals(color)) {
            result.add(apple);
        }
    }

    return result;
}

List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);

```

- 위 코드처럼 색이 여러개 추가되더라도 간단하게 대응할 수 있다.
- 하지만 색 관련된 부분이 아니라 다른 필터링(무게 등)이 추가된다면 결국 비슷한 코드가 복사, 수정 된다.
  - 다른 필터링도 결국은 if(apple.getColor().equals(color))의 조건만 변경된다.
  - 이는 결국 DRY(don't repeat yourself) 원칙을 어기게 된다.

### 2.1.3 세 번째 시도: 가능한 모든 속성으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple: inventory) {
        if((flag && apple.getColor().equals(color) ||
            (!flag && apple.getWeight() > weight))) {
            result.add(apple);
        }
    }

    return result;
}
```

- 위 코드처럼 모든 필터링 관련 파라미터를 추가할 경우에는 어떤 파라미터가 어떻게 동작하는지 등을 파악하기가 힘들어진다.
  - 또한, 필터링 관련 로직이 늘어날 경우 파라미터가 무한 추가된다.

## 2.2 동작 파라미터화

- 참 또는 거짓을 반환하는 함수를 `프리디케이트`라고 한다.

```java
// 사과의 상태에 따라 참 또는 거짓을 반환하는 프리디케이트 인터페이스
public interface ApplePredicate {
    boolean test(Apple apple);
}

public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return Green.equals(apple.getColor());
    }
}

```

- 조건에 따라 filter 메서드가 다르게 동작한다.
  - 이러한 패턴을 `전략 패턴`이라 한다.
  - 각 전략(알고리즘)에 따라 캡슐화하여 런타임에 전략을 선택하는 것이다.
- 동작 파라미터화는, 이러한 전략을 파라미터로 받아서 내부적으로 전략을 수행하는 것을 말한다.
- 이를 통해 컬렉션을 반복한느 로직과 컬렉션의 각 요소에 적용할 동작을 분리할 수 있다.

### 2.2.1 네 번째 시도: 추상적 조건으로 필터링

```java
public class List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory) {
        if(p.test(apple)) {
            result.add(apple);
        }
    }
}
```

- 필터 로직인 test 메서드를 전달하기 위해서 클래스를 정의하여 객체로 전달해야 한다.
  - 이는 코드를 전달하는 것과 같다.
- 클래스를 별도로 정의하는 것이 아니라, 람다를 이용하여 여러 클래스를 정의하지 않고 전달할 수 있다.

## 2.3 복잡한 과정 간소화

### 2.3.1 익명 클래스

- 자바에서는 클래스 선언와 동시에 인스턴스화할 수 있는 `익명 클래스` 라는 기법을 제공한다.
  - 익명 클래스를 이용하면 불필요한 클래스 선언과 인스턴스화를 줄일 수 있기 때문에 코드의 양을 줄일 수 있다.
- `람다 표현식`을 이용하면 익명 클래스 보다 더욱 간결하게 사용할 수 있다.

### 2.3.2 다섯 번째 시도: 익명 클래스 사용

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple) {
        return RED.equals(apple.getColor());
    }
})
```

- 익명 클래스를 이용하면 코드가 장황해지기 때문에 유지보수가 어려워진다.

### 2.3.3 여섯 번째 시도: 람다 표현식 사용

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

### 2.3.4 일곱 번째 시도: 리스트 형식으로 추상화

```java
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e: list) {
        if(p.test(e)) {
            result.add(e);
        }
    }

    return result;
}

List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

## 2.4 실전 예제

```java
// Comparator
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// Runnable
Thread t = new Thread(() -> System.out.println("Hello World"));

// Callable
Future<String> threadName = executorService.submit(() -> Thread.currentThread().getName());
```

## 2.5 정리

- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있으며 나중에 엔지니어링 비용을 줄일 수 있다.
- 코드 전달 기법을 잘 이용하면 동작을 메서드 인수로 전달할 수 있다.
  - 자바 8 이전에는 코드를 지저분하게 구현해야 했지만, 람다를 이용하여 간단하게 구현할 수 있다.
- 자바 API의 많은 메서드는 정렬, 스레드, GUI 처리 등을 포함한 다양한 동작으로 파라미터화할 수 있다.