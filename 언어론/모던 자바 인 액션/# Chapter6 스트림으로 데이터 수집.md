# Chapter6 스트림으로 데이터 수집

- 스트림은 `중간 연산`과 `최종 연산`으로 구분할 수 있다.
- `중간 연산`은 스트림을 다른 스트림으로 변환시켜준다.(스트림을 소비하지 않는다.)
- `최종 연산`은 스트림을 소비하여 최종 결과를 도출시켜준다.
  - 최종 연산은 스트림 파이프라인을 최적화하며 계싼 과정을 짧게 생략하기도 한다.

## 6.1 컬렉터란 무엇인가?

- `Collector` 인터페이스는 스트림의 요소를 어떤 식으로 도출할지 결정한다.
- 스트림에서 collect를 호출하면 스트림 요소에 리듀싱 연산을 수행하여 최종 결과를 반환하게 된다.
- 기본적으로 제공되는 Collcetors의 메서드는 는 크게 세 가지로 구분할 수 있다.
  - 스트림 요소를 하나의 값으로 리듀스하고 요약
  - 요소 그룹화
  - 요소 분할

## 6.2 리듀싱과 요약

- 스트림에서 숫자 필드의 합계나 평균등을 반환하는 연산에서 리듀싱이 자주 사용되며 이를 `요역`이라 한다.
- averagingXXX, summingXXX, summarizingXXX 등을 통해 기본적으로 제공하는 요약 기능을 사용할 수 있다.
- `joining`을 이용하여 스트림의 각 객체의 toString을 이용하여 하나의 문자열로 연결하여 반환할 수 있다.
- `reducing` 메서드를 이용하여 커스텀하게 리듀싱을 할 수 있다.
  - reducing은 세 가지 매개변수를 가진다.
  - 첫 번째 인수는 리듀싱 연산의 시작값을 의미한다.(스트림 원소가 없으면 첫 번째 인수를 반환한다.)
  - 두 번째 인수는 변환할 때 사용된다.
  - 세 번째 인수는 두 항목을 하나의 값으로 변경할 때 사용된다(리듀싱)
  - 첫 번째, 두 번째 인수를 생략하게 될 경우 초기 값은 스트림의 첫 원소가 되며 원소 그대로 사용된다.(이때는 Optional이 반환된다)
- reduce와 collect는 비슷해보이지만, 역할이 다르다.
  - reduce는 두 값을 통해 하나의 결과로 도출하는 불변형 연산이다.
  - collect는 결과의 누적 결과 컨테이너를 변경하도록 설계된 메서드이다.

## 6.3 그룹화

- `Collectors.groupingBy`를 이용하여 쉽게 그룹화할 수 있다.
- groupingBy를 통해 그룹핑한 값들의 필터링이나 매핑을 적용할 수 있다.
  - groupingBy(Dish::getType, filtering(dish -> dish.getCalories() > 500, toList()))
  - groupingBy(Dish::getType, mapping(Dish::getName, toList()))
- groupingBy는 두 번째 기준을 정의하는 groupingBy를 받을 수 있다.
  - 즉, n개의 그룹핑 기능을 제공한다.
  - `Map<Type, Map<Name, Map<Id, List<Intgeer>>>>`
- groupingBy의 두 번째 원소는 Collector를 받기 떄문에 groupingBy외에도 다른 Collector를 전달할 수 있다.
  - ex) groupingBy(Dish::getType, counting()) --> 각 타입별 개수

## 6.4 분할

- `Collectors.partitioningBy`를 이용하면 predicate의 true, false로 분류할 수 있다.

## 6.5 Collector 인터페이스

- Collector 인터페이스에는 5가지 메서드가 존재하며, 해당 메서드를 통해 동작된다.
- supplier
  - supplier 메서드는 수집과정에서 빈 누적자 인스턴스를 만든다.
- accumulator
  - accumulator 는 결괴 컨테이너에 요소를 추가한다.
  - 리듀싱 연산을 수행하는 함수다.
- finisher
  - finisher는 최종 반환값을 결과 컨테이너로 적용한다.
- combiner
  - combiner는 두 결과 컨테이너를 병합한다.
  - 즉, 마지막으로 리듀싱 연산에서 사용할 함수를 반환한다.
  - 리듀싱을 병렬로 수행할 때 사용된다.
  - 스트림을 분할하여 처리하고 병합한다.
- Characteristics
  - Characteristics는 스트림을 병렬로 리듀스할 것인지, 병렬로 한다면 어떤 최적화를 할 것인지에 대한 힌트 제공을 위해 사용된다.
  - UNORDERED: 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
  - CONCURRENT: 다중 스레드에서 accumlator 함수를 동시에 호출할 수 있으며, 병렬 리듀싱을 수행할 수 있다.
  - IDENTITY_FINISH: finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 생략할 수 있다.