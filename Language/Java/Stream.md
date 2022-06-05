# 스트림



## 스트림 개요

- 자바8 부터는 컬렉션의 요소를 하나씩 참조하여 람다식으로 처리할 수 있도록 지원한다.
- 기존의 Iterator와 비슷한 역할을 하지만, 람다식을 이용하여 내부 반복자를 사용하기 때문에 병렬 처리나 중간 처리가 쉽다.
- Stream API는 데이터를 추상화하고, 처리하는데 자주 사용되는 함수들을 정의해두었다.
- 데이터를 추상화 하여 데이터의 종류에 상관 없이 같은 방식으로 데이터를 처리할 수 있다.

## Stream API 의 특징

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

## 스트림의 종류

- 스트림은 기본적으로 BaseStream을 상속 받은 Stream, IntStream, LongStream, DoubleStream이 있다.
  - BaseStream은 인터페이스로, 직접 사용되지는 않는다.
- Stream은 객체 요소를 처리하는 스트림이다.
- IntStream, LongStream, DoubleStream은 int, long, double 요소 처리 스트림이다.
- `Collection`에서는 stream(), parallelStream()을 통해 Stream<T>를 얻을 수 있다.
- `배열`에서는 Arrays.stream(T[]), Stream.of(T[])을 통해 Stream<T>를 얻을 수 있다.
  - Arrays.stream(int[]), Stream.of(int[]) 을 통해 IntStream을 얻을 수 있다.
  - 그 외에도 LongStream, DoubleStream을 배열을 통해 얻을 수 있다.
- `int 범위 스트림`은 IntStream.range(int, int), IntStream.rangeClosed(int, int)에서 얻을 수 있다.
- `long 범위 스트림`은 LongStream.range(long, long), LongStream.rangeClosed(long, long)에서 얻을 수 있다.
- `디렉토리 스트림`은 Files.find(Path, int, BiPredicate, FileVisitOption), File.list(Path)에서 얻을 수 있다.
- `파일 스트림`은 Files.lines(Path, Charset), BufferedReader.lines() 에서 얻을 수 있다.

## 스트림 파이프라인

### 중간 처리와 최종 처리

- 중간 처리 스트림은 처리 후 다시 중간 처리 스트림을 리턴한다.
- 최종 처리 스트림은 기본 타입이거나 OptionalXXX이다.
- 중간 처리 스트림은 기본적으로 lazy하기 때문에 최종 처리 스트림이 있어야 실행된다.
  - `중간 처리 스트림만 작성되는 경우 실행되지 않는다.`
- 스트림의 경우 각 단계가 모두 진행되고 다음 단계가 진행되는 것이 아니라, 한 단계씩 차례로 진행된다.
  - filter -> forEach 순으로 진행되는 경우, 모든 filter후 forEach가 아니라 하나 filter후 하나 forEach 다시 filter 순으로 진행된다.

```java

List<String> myList = Arrays.asList("a1", "a2", "b1", "c2", "c1"); 
myList
 .stream() // 스트림 생성
 .filter(s -> s.startsWith("c")) // 중간 처리
 .map(String::toUpperCase) // 중간 처리
 .sorted() // 중간 처리
 .count(); // 최종 처리

```


## 필터링

- distinct()는 중복을 제거한다.
  - distinct는 Object의 equals 메소드를 이용하여 중복을 판단한다.
- filter(Predicate), filter(IntPredicate), filter(LongPredicate), filter(DoublePredicate)는 조건 필터링을 제공한다.
- distinct와 filter모두 Stream, IntStream, LongStream, DoubleStream을 리턴할 수 있다.


## 매핑

- mapXXX() 메소드는 요소를 대체하는 새로운 스트림을 리턴한다.
  - 객체 A를 대신하는 객체 B 스트림 생성
- flatMapXXX() 메소드는 복수 개의 요소들을 새로운 하나의 스트림으로 만들어준다.
  - List<String>을 하나의 문자들의 스트림으로 만들 수 있다.
- asDoubleStream()은 IntStream, LongStream의 요소를 DoubleStream으로 만들어준다.
- asLongStream()은 IntStream 요소를 LongStrea으로 만들어준다.
- boxed()는 int, long, double 요소를 Integer, Long, Double 요소로 박싱하여 Stream을 만들어준다.

## 정렬

- sorted()를 이용하여 객체의 Comparable 구현 방법에 따라 정렬할 수 있다.
- sorted(Compartor<T>)를 이용하여 Comaparator를 전달하여 정렬할 수 있다.

## 루핑

- peek()은 전체를 순회하여 반복 작업을 하고 중간 처리 메소드이다.
  - 중간 처리 메소드이기 때문에 peek만 선언하고 종료하는 경우, 메소드가 실행되지 않는다.
- forEach()는 전체를 순화하여 반복 작업을 하고 최종 처리 메소드이다.

## 매칭

- allMatch()는 모든 요소가 Predicate를 만족하는 경우에 true이다.
- anyMatch()는 하나의 요소만 Predicate를 만족해도 true이다.
- noneMatch()는 모든 요소가 Predicate에 만족하지 않은 경우에 true이다.

## 집계

- sum, average, count, max, min 등의 집계 함수를 제공한다.

## 커스텀 집계

- 제공하는 집계 함수 외 reduce를 이용하여 집계 함수를 만들 수 있다.
- reduce((a, b) -> a+b) 처럼 sum을 직접 구현할 수 있다.

## 수집

- 스트림 요소들을 컬렉션에 담을 수 있다.
- Collector의 정적 메소드를 이용하여 컬랙션으로 담을 수 있다.
  - toList(): T를 리스트로 저장한다.
  - toSet(): T를 Set으로 저장한다.
  - toCollection(Supplier<Collection<T>>): T를 Supplier가 제공한 Collection에 저장
  - toMap( Function<T, K> keyMapper, Function<T, U> valueMapper ): T를 K로 매핑하여 키로, T를 U로 매핑하여 값으로 저장

### 사용자 정의 컨테이너 수집

- List, Map 등의 컬렉션이 아니라, 사용자 정의 컨테이너 객체에 수집할 수 있다.
- collect(Supplier<R>, BiConsumer<R, ? super T>, BiConsumer<R, R>)
  - 첫 번째 Supplier는 수집될 컨테이너 객체 R을 생성한다.
    - 싱글 스레드의 경우 하나만 생성되지만, 멀티 스레드의 경우 여러개 생성된 후 합쳐진다.
  - 두 번째 Bicunsumer의 경우 컨테이너 객체 R에 요소 T를 수집한다.
  - 세 번째 BiConsumer의 경우 객체 R을 결합한다. 순차 스트림에서는 호출되지 않는다.

### 요소 그룹핑


## 병렬 처리

- parallelStream, parallel 메소드를 이용하여 병렬 스트림을 생성할 수 있다.
- 병렬 스트림은 런타임 시 포크조인 프레임워크가 동작한다.
- 포크 단계에서 전체 데이터를 서브 데이터로 분리한 후 멀티 코에에서 병렬로 처리한다.
- 모든 서브 데이터의 처리가 완료되면, 조인 단계를 거친 후 최종 결과를 산출한다.
