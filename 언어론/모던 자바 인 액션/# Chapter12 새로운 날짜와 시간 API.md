# Chapter12 새로운 날짜와 시간 API

- 자바1.0 에서는 java.util.Date에서 날짜와 시간 관련 API를 제공하였지만, 여러 문제가 발생했다.
  - 기준 오프셋이 1900년이고, 달 인덱스는 0으로 시작하는 등 불편함이 존재한다.
- 자바1.1 부터는 java.util.Calendar 라는 새로운 날짜와 시간 관련 API를 출시하였지만, 문제는 여전히 존재했다.
- 날짜 관련 처리를 하는 DateFormat 같은 API 역시 문제가 많았다.
- 자바8 에서는 결국 Joda-Time의 많은 기능을 java.time에 추가하게 된다.

## 12.1 LocalDate, LocalTime, Instant, Duration, Period 클래스

- java.time 패키지에서는 LocalDate, LocalTime, Instant, Duration, Period 클래스를 제공한다.
- 해당 클래스들은 모두 불변 객체로 제공되기 때문에 스레드 경합에 안전하다.

### LocalDate, LocalTime, LocalDateTime

- LocalDate는 시간을 제외한 날짜를 표현하는 불변 객체다.
  - LocalDate.of 메서드를 통해 인스턴스를 생성한다.(LocalDate.of(2017, 9, 21))
  - LocalDate.now 메서드를 이용하여 현재 시간을 얻을 수 있다.
- LocalTime은 시간 데이터만 표현하는 불변 객체다.
- LocalDateTime은 LocalDate와 LocalTime을 쌍으로 같은 복합 클래스로 날짜와 시간을 모두 표현하는 불변 객체다.

### Instant

- Instant 클래스는 기계적 관점에서 시간을 표현한다.
  - 기계적 관점으로 표현하기 때문에 사람이 사용하는 형식으로는 표현할 수 없다.
- 기계적 관점에서 시간이란 유닉스 에포크 시간(Unix epoch time, 1970/01/01 00:00:00 UTC) 기준으로 특정 지점까지의 시간을 초로 표현한 것이다.
- Instant는 LocalXXX 과 달리 초와 나노초 정보를 가지고 있다.

### Duration, Period

- 두 시간 단위 비교를 하기 위해 Duration과 Period를 사용할 수 있다.
- Duration은 초, 나노초로 시간 단위를 표현하기 때문에 Instant 끼리 또는 LocalTime 끼리 비교할 수 있다.
- Period는 날짜 관련된 비교를 하기 위해서 사용할 수 있다. 즉, LocalDate 끼리 비교한다.

## 12.2 날짜 조정, 파싱, 포매팅

### 날짜 조정

- `withAttribute` 메서드로 기존의 데이터를 조정할 수 있다.
  - date.withYear(2011); // 년도를 2011로 변경
  - date.withDayofMonth(25); // 해당 달의 날짜를 25일로 변경
- `TemporalAdjusters` 펙토리 메서드를 이용하여 미리 정의된 여러 작업을 할 수 있다.
  - dayOfWeekInMonth, firstDayOfMonth, firstDatOfNextMonth 등

### 날짜 파싱, 포메팅

- java.time.format에서는 `DateTimeFormatter` 클래스를 제공하여 포매터를 만들 수 있게 제공한다.
- DateTimeFormatter 에서는 BASIC_ISO_DATE, ISO_LOCAL_DATE 등의 포메터를 미리 정의해두었다.
- 포메터를 이용하여 string을 날짜 관련 클래스로, 날짜 관련 클래스를 string으로 변환할 수 있다.

```java
LocalDate date = LocalDate.of(2014, 3, 18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);  // 20240318
String s1 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);  // 2024-03-18

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date1 = LocalDate.of(2014, 3, 18);
String formattedDate = date1.format(foramtter);
LocalDate date2 = LocalDate.parse(formattedDate, formatter);
```

## 12.3 다양한 시간대와 캘린더 활용 방법

- 자바에서 기존에는 java.util.TimeZone을 사용했지만, 이를 대체하기 위해 `java.time.ZoneId` 가 등장했다.
- java.time.ZoneId 을 사용하면 서머타임과 같은 사항도 자동으로 처리할 수 있다.

### 시간대 사용하기

- `ZoneRules` 클래스에는 40개 정도의 시간대가 존재한다.
- `ZoneId` 는 특정 시간대 규정을 획득할 수 있다.
  - ZoneId.of("Europe/Rome");
  - ZoneId는 `지역/도시` 형식으로 이루어져있다.
- ZoneId를 기준으로 LocalDate, LocalDateTime, Instant를 이용하여 ZoneDateTime 인스턴스로 변경할 수 있다.
  - ZoneDateTime는 지정한 시간대에 상대적인 시점을 표현한다.
- ZoneId를 이용하여 LocalDateTime을 Instan로 변경할 수도 있다.
  - LocalDateTime.ofInstant(istant, romeZone);

### UTC/Greenwich 기준의 고정 오프셋

- UTC/GMT 를 기준으로 시간대를 표현하는 경우가 있다.
- ZoneOffset을 이용하여 offset을 설정할 수 있다.
  - 기준은 UTC/GMT 로 하며, 이를 이용하여 원하는 offset을 생성할 수 있다.(ZoneOffset.of("-05:00"))
  - 즉, UTC로 부터 얼마나 떨어져 있는가를 의미한다.

### 정리

- LocalDateTime --> 현재 날짜(해당 프로그램이 있는 위치)
- OffsetDateTime --> offset 기준 날짜
  - offset은 단순히 기준(UTC)에서 얼마나 차이가 나는가에 대한 정보만 가지고 있다.
- ZonedDateTime --> 특정 지역 기준 날짜
  - ZonedId는 툭정 Zone에 대한 정보를 가지고 있다.
  - 가장 포괄적

## 마치며

- 자바8 이전 버전에서 제공하는 기존의 java.util.Date 클래스와 관련 클래스에서는 여러 불일치점들과 가변성, 어설픈 오프셋, 기본 값 등 설계 결함이 존재한다.
- 새로운 날짜와 시간 API에서 날짜와 시간 객체는 모두 불변이다.
- 새로운 API는 각각 사람과 기계가 편리하게 날짜와 시간 정보를 관리할 수 있도록 두 가지 표현 방식을 제공한다.
- 날짜와 시간 객체는 절대적인 방법과 상대적인 방법으로 처리할 수 있으며 기존 인스턴스를 변환하지 않도록 처리 결과를 새로운 인스턴스가 생성된다.
- TemporalAdjuster를 이용하면 단순히 값을 바구는 것 이상의 복잡한 동작을 수행할 수 있으며 자신만의 커스텀 날짜 변환 기능을 정의할 수 있다.
- 날짜와 시간 객체를 특정 포맷으로 출력하고 파싱하는 포매터를 정의할 수 있다.
- 특정 지역/장소에 상대적인 시간대 또는 UTC/GMT 기준의 오픗세을 이용해서 시간대를 정의할 수 있으며 이 시간대를 날짜와 시간 객체에 적용해서 지역화할 수 있다.
- ISO-8601 표준 시스템을 준수하지 않는 캘린더 시스템도 사용할 수 있다.