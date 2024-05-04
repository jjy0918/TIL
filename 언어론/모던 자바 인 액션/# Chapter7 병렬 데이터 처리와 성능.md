# Chapter7 병렬 데이터 처리와 성능

- 자바 7 전에는 컬렉션을 병렬로 처리하기 위해서 데이터를 서브 파트로 분할 후 각각 스레드 할당하여 처리했다.
- 각 스레드가 데이터를 접근하기 때문에 레이스 컨디션이 발생하지 않도록 적절한 동기화가 필요했다.
- 자바 7 부터는 `포크/조인 프레임워크` 를 제공하기 때문에 병렬화를 쉽게 제공한다.
- 그리고 자바 8부터는 스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 변경할 수 있다.

## 7.1 병렬 스트림

- 컬렉션에서 `parallelStream`을 호출하면 병렬 스트림을 생성할 수 있다.
- 병렬 스트림은 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림을 말한다.
  - 즉, 병렬 스트림은 여러 청크 스트림으로 분리하고 멀티코어가 각 청크를 처리한다.

### 7.1.1 순차 스트림을 병렬 스트림으로 변환하기

- 순차 스트림에 parallel 메서드를 호출하면 병렬로 처리하도록 변경할 수 있다.

```java
Stream.iterate(1L, i -> i+1)
    .limit(n)
    .parallel() // reduce 연산을 병렬로 처리하도록 선언
    .reduce(0L, Long::sum);
```

- parallel 메서드를 선언하여 스트림을 여러 청크로 분할하게 된다. 즉, reduce 연산을 여러 청크가 분할 수행한다.
- sequential 을 선언하면 병렬 스트림을 순차 스트림으로 변경할 수 있다.
- parallel 과 sequential 이 여러번 사용되었다면, 최종적으로 선언된 값만 적용된다.
- 병렬 스트림의 경우 내부적으로 `ForkJoinPool`을 사용하여 병렬 처리를 한다.
  - ForkJoinPool은 기본적으로 프로세서 수에 상응하는 스레드를 가진다.
- 무조건 parallel을 붙인다고 빨라지는 것은 아니다. 병렬화를 할 때에는 항상 주의해야 한다.
  - parllel을 붙이더라도 박싱/언박싱에 대한 부분 처리를 잘못한다면 오히려 느려질 수 있다.
  - 또한, 병렬 처리에 대한 부분이 제대로 지원되는지 확인해야 한다.
    - 단순히 parallel에 reduce를 붙인다고 모두 reduce가 의도한 것 처럼 병렬로 진행되지 않는다.
    - ex) rangeClosed로 만들어진 스트림은 병렬 reduce가 잘 동작하도록 스트림 청크가 분할된다.
  
### 7.1.3 병렬 스트림의 올바른 사용법

- 병렬 처리에서 가장 큰 문제는 공유된 상태를 바꾸는 부분이다.
- 데이터를 여러 스레드가 접근할 수 있는 레이스 컨디션 문제가 발생한다.

### 7.1.4 병렬 스트림 효과적으로 사용하기

- 무조건 병렬 스트림으로 바꾸는 것이 능사가 아니기 때문에 확신이 서지 않으면 직접 측정하라.
- 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있기 떄문에 박싱을 주의하라.
  - 기본형을 사용하려면 기본 특화형 스트림을 사용하라.
- limit, findFirst 처럼 요소의 순서에 의존하는 연산은 오히려 병렬 스트림 보다 순차 스트림에서 성능이 더 빠르다.
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라.
- 소량의 데이터에서는 병렬 스트림이 도움 되지 않는다.
  - 소량의 데이터는 오히려 병렬화 과정에서 생기는 부가 비용이 이득보다 클 수 있다.
- 스트림을 구성하는 자료구조가 적절한지 확인하라.
- 스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
- 최종 연산의 병합 과정 비용을 살펴보라.

## 7.2 포크/조인 프레임워크

- 포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음 서브태스크 각각의 결과를 합쳐 전체 결과를 만든다.
- 포크/조인 프레임워크에서 서브태스크는 스레드 풀의 작업자 스레드에 분산 할당하는 `ExecutorService` 인터페이스를 구현한다.

### 7.2.1 RecursiveTask 활용

- 스레드 풀을 이용하려면 `RecursiveTask<R>`의 서브클래스를 만들어야 한다.
- RecursiveTask 인터페이스는 `R`을 반환하는 추상 메서드 `compute` 를 가지고 있다.
  - compute 는 일반적으로 태스크를 분할할 수 없으면 계산을 수행하고, 분할할 수 있으면 분할 후 재귀적으로 compute 메서드를 수행한다.
  - `device and conquer` 알고리즘의 병렬화 버전이다.

```java
public class ForkJoinSumCalculator extends RecursiveTask<Long> {
    ...
    @Override
    protected Long compute() {
        int length = end - start;

        if(length <= THREASHOLD) {
            return computeSequentially();
        }

        ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
        leftTask.fork();

        ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
        Long rightResult = rightTask.compute();
        Long leftResult = leftTask.join();

        return leftResult + rightResult;
    }
    ...
}
```

- 일반적으로 하나의 소프트웨어에서는 둘 이상의 `ForkJoinPool` 인스턴스 생성하여 사용하지 않는다.
  - 싱글톤으로 ForkJoinPool 인스턴스를 생성하여 사용한다.
- ForkJoinPool 인스턴스의 invoke 메서드에 RecursiveTask 구현체를 전달하면 포크/조인 프레임워크를 통해 작업을 수행할 수 있다.

### 7.2.2 포크/조인 프레임워크를 제대로 사용하는 방법

- join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시키기 때문에 두 서브태스크가 모두 시작된 후 join을 해야 한다.
- RecursiveTask 내에서는 ForkJoinPool의 invoke 메서드를 사용하지 말아야 한다.
  - invoke 대신 compute나 fork 메서드를 직접 호출할 수 있다.
  - 순차 코드에서 병렬 계산을 시작할 때만 invoke를 사용해야 한다.
- 서브태스크에 fork 메서드를 호출하여 ForkJoinPool의 일정을 조절할 수 있다.
  - 모든 태스크를 fork로 실행하는 것 보다는 한쪽은 compute를 통해 작업을 수행하는 것이 효율적이다.
  - 현재 사용중인 스레드를 풀에 넣고 다시 스레드를 할당 받는 불필요한 작업 없이 그대로 재활용할 수 있다.
- 포크/조인 프레임워크가 순차 처리보다 무조건 빠를 거라는 생각은 버려야 한다.
  - 여러 번 프로그램을 실행한 결과를 측정하여 판단하라.
  - 컴파일러는 병렬 보다 순차 작업 최적화를 더 잘한다.

### 7.2.3 작업 훔치기

- 일반적으로 코어 개수 만큼 태스크를 할당해야 최적으로 분할한다고 생각한다.
- 서브 태스크의 모든 결과가 동일하게 끝난다는 가정하에는 맞는 말이지만, 태스크 작업 시간이 모두 다르기 때문에 크게 달라질 수 있다.
- 포크/조인 프레임워크 에서는 `작업 훔치기(work stealing)` 이라는 기법으로 이러한 문제를 해결한다.
  - 작업 훔치기 기법을 통해 모든 스레드를 거의 공정하게 분할한다.
  - 각 스레드는 자신의 큐에 있는 태스크를 가져와 처리하며, 자신의 작업이 끝나면 다른 스레드 큐의 tail에 있는 작업을 훔쳐간다.
- 포크/조인 프레임워크에서는 작업 훔치기를 통해 스레드가 지속적으로 작업을 진행하기 때문에 태스크의 크기를 작게 나누어야 한다.

## 7.3 Spliterator 인터페이스

- 자바 8의 `Spliterator` 인터페이스는 병렬 작업에 특화된 소스 요소 탐색 기능을 제공한다.
- 자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 Spliterator 구현을 제공한다.

```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action); // 탐색 요소가 남아 있는지 확인
    Spliterator<T> trySplit();  // 분할 작업
    long estimateSize();    // 탐색해야 할 요소 수
    int characteristics();  // Spliterator의 특성
}
```

- Spliterator 인터페이스의 `characteristics` 는 Spliterator의 특성을 나타낸다.
  - ORDERED, DISTINCT, SORTED, SIZE, NON-NULL, IMMUTABLE, CONCURRENT, SUBSIZED
  - 각 값은 미리 정의된 int 값을 가지고 있으며, characteristics 의 리턴 값은 해당 특성의 조합을 통해 표현한다.
  - ex) `return ORDERED | SIZED | IMMUTABLE | SUBSIZED;`
  - https://docs.oracle.com/javase/8/docs/api/constant-values.html#java.util.Spliterator.CONCURRENT

### 커스텀 Spliterator 구현하기

- 문자열의 단어 수를 계산하는 프로그램을 구성해보자.
- 일반적으로 반복문을 통해 구현할 수 있으며, 순차 스트림을 통해 구현할 수도 있다.
- 단순히 순차 스트림에 parallel을 붙인다고 하더라도 구현 방식에 따라 결과가 의도한 것과 달라질 수 있다.
- 병렬 스트림 구성 시 태스크를 분할할 때 어떻게 분할하느냐에 따라 예외 상황이 발생할 수 있다.
- 병렬 스트림으로 구성 시에는 문자열의 임의의 위치 분할이 아니라 단어가 끝나는 위치에서만 분할해야 한다.

```java
class WordCounterSpliterator implements Spliterator<Character> {
    private final String string;
    private int currentCharIndex = 0;

    public WordCounterSpliterator(String string) {
        this.string = string
    }

    @Override
    public boolean tryAdvance(Consumer<? super Charater> action) {
        action.accept(string.charAt(currentCharIndex++));    // 현재 문자 소비
        return currentCharIndex < string.lenghth();
    }

    @Override
    public Spliterator<Character> trySplit() {
        int currentSize = string.length() - currentCharIndex;
        if(currentCharIndex < 10) {  // 파싱할 문자열을 순차 처리할 수 있을 만큼 충분히 작은지 판단.
            return null;   // 순차 처리 가능한 경우 null 반환
        }

        // 절반 위치에서 시작
        for(int splitPos = currentSize / 2 + currentCharIndex; splitPos < string.length(); splitPos++) {
            if(Character.isWhitespace(string.charAt(splitPos))) {   // 공백 문자가 나올 때 까지 이동
                // 공백 문자가 나올 경우 문자를 분할하여 위임
                Spliterator<Character> spliterator = new WordCounterSpliterator(string.substring(currentCharIndex, splitPos));
                currentCharIndex = splitPos;
                return spliterator;
            }
        }
    }

    @Override
    public long estimateSize() {    // 탐색할 요소의 개수
        return string.length() - currentChar;
    }

    @Override
    public int characteristict() {
        return ORDERED + SIZED + SUBSIZED + NON-NULL + IMMUTABLE:
    }
}


Spliterator<Character> spliterator = new WordCounterSpliterator(SENTENCE);
Stream<Character> stream = StreamSupport.stream(spliterator, true);

```

## 7.4 마치며

- 내부 반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고도 스트림을 병렬로 처리할 수 있다.
- 간단하게 스트림을 병렬로 처리할 수 있지만 항상 병렬 처리가 빠른 것은 아니다.
  - 직접 성능을 측정하며 확인해야 한다.
- 병렬 스트림으로 데이터 집합을 실행할 때 특히 처리해야 할 데이터가 아주 많거나 각 요소를 처리하는 데 오랜 시간이 걸릴 때 성능을 높일 수 있다.
- 가능하면 기본형 특화 스트림을 사용한느 등 올바른 자료구조 선택이 병렬 처리보다 성능적으로 더 큰 영향을 미칠 수 있다.
- 포크/조인 프레임워크에서는 병렬화할 수 있는 태스크를 작은 태스크로 분할한 다음 분할된 태스크를 각각의 스레드로 실행하며 서브태스크 각각의 결과를 합쳐서 최종 결과를 생산한다.
- Spliterator는 탐색하려는 데이터를 포함하는 스트림을 어떻게 병렬화할 것인지 정의한다.
