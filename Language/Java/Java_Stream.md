# Stream

[https://mangkyu.tistory.com/112](https://mangkyu.tistory.com/112)

# Stream API

- JDK8 부터 Stream API를 제공하여 함수형 프로그래밍을 할 수 있게 도와준다.
- Stream API는 데이터를 추상화하고, 처리하는데 자주 사용되는 함수들을 정의해두었다.
- 데이터를 추상화 하여 데이터의 종류에 상관 없이 같은 방식으로 데이터를 처리할 수 있다.

# Stream API 의 특징

- 원본 데이터를 변경하지 않는다.
    - 원본 데이터는 조회에서만 사용하여 별도의 요소들로 Stream을 생성한다.
    - 정렬이나 필터링 등의 작업은 별도의 Stream 요소들에서 처리가 된다.
- 일회성으로 동작한다.
    - Stream은 일회성이기 때문에 한 번 사용이 끝나면 재상용이 불가능하다.
    - Stream을 다시 사용하고 싶다면, 다시 생성해주어야 한다.
    - 닫힌 Stream을 다시 사용한다면, IllegalStateException이 발생하게 된다.
- 내부 반복으로 작업을 처리한다.
    - 내부적으로 반복 문법 메소드를 숨기고 있기 때문에 간결한 코드 작성이 가능하다.

- Stream 연산들은 함수형 인터페이스를 받도록 되어 있다.
    - 람다식은 반환 값으로 함수형 인터페이스를 반환한다.
    - Stream API를 사용하기 위해서는 람다식과 함수형 인터페이스를 알고 있어야 한다.

## 람다식(Lambda Expression)

- 함수를 하나의 식으로 표현한 것.
- 함수를 람다식으로 표현하면, 메소드의 이름이 필요 없기 때문에 익명 함수의 한 종류라고 볼 수 있다.
- 익명 함수는 이름 없는 함수로, 모두 일급 객체이다.
- 람다식을 이용하면, 불필요한 코드를 줄이고 가독성을 높여준다.
- 컴파일러가 문맥을 살펴 타입을 추론한다.

### 람다식의 특징

- 람다식 내에서 사용되는 지역 변수는 final이 붙지 않아도 상수로 간주된다.
- 람다식으로 선언된 변수명은 다른 변수명과 중복될 수 없다.

### 람다식의 장점

- 코드를 간결하게 만들 수 있다.
- 식에 개발자의 의도가 명확히 드러나 가독성이 높아진다.
- 함수를 만드는 과정 없이 한 번에 처리할 수 있어 생산성이 높아진다.
- 병렬 프로그래밍이 용이하다.

### 람다식의 단점

- 람다를 이용하며 만든 무명함수는 재사용이 불가능하다.
- 디버깅이 어렵다.

# Stream API의 연산 종류

## 1. 생성하기

- Stream을 생성하는 단계
- Stream 연산을 위한 Stream 객체를 생성한다.
- 배열, 컬렉션, 임의의 수, 파일 등 모든 것을 Stream으로 생성할 수 있다.
- 연산이 종료되면, Stream이 닫히기 때문에 Stream을 다시 사용하기 위해서는 다시 생성해야 한다.
- Stream 생성 방법
    - Collection
    - 배열
    - 원시 Stream

## 2. 가공하기

- 원본의 데이터를 별도의 데이터로 가공하기 위한 중간 연산
- 연산 결과를 Stream으로 다시 반환하기 때문에 연속해서 중간 연산을 이어갈 수 있다.
- 필터링 - Filter
    - 조건에 맞는 데이터만 정제하여 더 작은 컬렉션을 만들어내는 것.
    - filter 함수의 인자로 Predicate 함수형 인터페이스를 받고 있기 때문에 boolean을 반환하는 람다식을 작성하면 된다.
    - filter(name -> name.contains("a"));
- 데이터 변환 - Map
    - 기존 Stream 요소들을 변환하여 새로운 Stream을 형성하는 연산
    - 저장된 값을 특정한 형태로 변환하는데 주로 사용된다.
    - map 함수의 람다식은 메소드 참조를 이용하여 변경이 가능하다.
- 정렬 - Sorted
    - Stream의 요소들을 정렬하기 위해 사용한다.
    - 파라미터로 Comparator를 넘길 수도 있다.
    - Coparator 인자 없이 호출할 경우 오름차순으로 정렬된다.
- 중복 제거 - Distinct
    - Stream 요소들에 중복된 데이터가 존재하는 경우, 중복 데이터를 제거한다.
    - 중복 데이터를 검사하기 위해 Object의 equals 메소드를 사용한다.
    - 개발자가 정의한 클래스를 사용하기 위해서는 equals와 hashCode를 오버라이드 해야 제대로 적용된다.

## 3. 결과 만들기

- 가공된 데이터로부터 원하는 결과를 만들기 위한 최종 연산
- Stream의 요소들을 소모하며 연산이 수행되기 때문에 1번만 처리 가능하다.

```java

List<String> myList = Arrays.asList("a1", "a2", "b1", "c2", "c1"); 
myList
 .stream() // 생성하기
 .filter(s -> s.startsWith("c")) // 가공하기
 .map(String::toUpperCase) // 가공하기
 .sorted() // 가공하기
 .count(); // 결과만들기

출처: https://mangkyu.tistory.com/112 [MangKyu's Diary]
```

- 최댓값/최솟값/총합/평균/갯수 - Max/Min/Sum/Average/Count
- 데이터 수집 - collect
    - Stream의 요소들을 List, Set, Map 등 다른 종류의 결과로 수집하고 싶은 경우에 collect 함수를 이용할 수 있다.
    - collect 함수는 Collector 타입을 인자로 받아 처리한다.
    - List 등 자주 사용하는 Collector는 Collectors 객체에서 static 메소드로 제공한다.
    - 

```java
List<String> nameList = productList.stream()
 .map(Product::getName)
 .collect(Collectors.toList());

```

- 조건 검사 - Match
    - Stream의 요소들이 특정한 조건을 충족하는지 검사하고 싶을 때 사용한다.
    - match 함수는 함수형 인터페이스 Predicate를 받아서 해당 조건을 만족하는지 검사하게 되고, 검사 결과를 boolean으로 반환한다.
    - anyMatch
        - 1개의 요소라도 해당 조건을 만족하는가
    - allMatch
        - 모든 요소가 해당 조건을 만족하는가
    - nonMatch
        - 모든 요소가 해당 조건을 만족하는가

```java
List<String> names = Arrays.asList("Eric", "Elena", "Java"); 
boolean anyMatch = names.stream()
 .anyMatch(name -> name.contains("a")); 

boolean allMatch = names.stream()
 .allMatch(name -> name.length() > 3); 

boolean noneMatch = names.stream()
 .noneMatch(name -> name.endsWith("s"));

// => 모든 경우가 true를 반환한다.
```

[https://mangkyu.tistory.com/115](https://mangkyu.tistory.com/115)