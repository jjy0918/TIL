# 03 Reactor 개요

## Reactor 란?

- Reactor는 Spring Framework 팀이 만든 리액티브 스트림즈 구현체를 말한다.

## 마블 다이어그램

- Reactor에서는 마블 다이어그램을 통해 구조를 이해하기 쉽게 다이어그램으로 그릴 수 있다.

## Cold Sequence와 Hot Sequence

- Hot Sequence에서 Hot은 Hot Deploy가 의미하는 즉시 배포 처럼 즉시 무언가를 하는 것이다. 즉, 다시 처음부터 하지 않는 것을 말한다.
  - 간단하게 말하자면 무언가를 새로 시작하지 않는 다는 것이다.
- 반대로 Cold 는 다시 처음부터 시작하는 것을 말한다.

### Cold Sequence

- Sequence란 Publisher가 emit하는 데이터의 흐름을 말한다.
- Cold Sequence는 Subscriber가 데이터를 구독할 때 `처음부터` 데이터를 처리하는 것을 말한다.
- Publisher 가 데이터를 구축하고 나서 새로운 Subscriber가 subscribe 했을 때 다른 Subscriber가 어느 데이터까지 처리했는지와 상관 없이 무조건 처음부터 새로 처리한다.

### Hot Sequence

- Hot Sequence 는 Subscriber가 데이터를 구독할 때 `현재 처리중인` 데이터 부터 처리하는 것을 말한다.
- Publisher 가 데이터를 구축하고 나서 새로운 Subscriber가 subscribe 했을 때 다른 Subscriber가 어느 데이터까지 처리했는지를 보고, 그 다음 데이터부터 처리한다.
  - Subscribe 시점 이전의 데이터는 emit하지 못한다.
- Mono의 경우에는 `cache()`, Flux의 경우에는 `share()` Operator를 사용하면 HotSequence 로 동작하게 처리할 수 있다.
  - cache의 경우에는 emit된 데이터를 캐시한 뒤 전달한다. 즉, Mono의 경우에는 데이터가 하나라서 처리된 데이터를 캐싱해두었다가 반환만 하게 되는 것이다.

## Backpressure

- Backpressure란 배압 또는 역압이라고 한다. 
- 리액티브 프로그래밍에서 Backpressure는 Subscriber가 처리할 수 있는 만큼의 데이터만을 요청하여 가져오는 것을 의미한다.
- 정리하자면, 데이터 전달량을 보내는쪽이 아니라 받는 쪽에서 정하는 것이다.

### Reactor에서 Backpressure 처리

- Reactor에서는 Subscriber가 request 메서드를 통해 데이터 처리 개수를 요청한다.
- 일반적으로는 BaseSubscriber를 구현하는 것으로 Backpressure 처리를 할 수 있다.
  - BaseSubscriber의 hookOnSubscribe를 통해 구독할 떄 몇개의 데이터를 요청할 지 결정하여 request를 요청한다.
  - BaseSubscriber의 hookOnNext를 통해 다음 데이터는 몇개를 요청할 지 결정하여 request를 요청한다.

### Backpressure 전략

- IGRNORE 전략
  - Backpressure를 적용하지 않는다.
- ERROR 전략
  - Downstream으로 전달할 데이터가 버퍼에 가득찰 경우, Exception을 발생시킨다.
  - `onBackpressureError()` Operator를 사용하면 된다.
- DROP 전략
  - Downstream으로 전달할 데이터가 버퍼에 가득찰 경우, 버퍼 밖에 대기하는 먼저 emit된 데이터부터 Drop 시킨다.
  - `onBackpressureDrop()` Operator를 사용하면 된다.
- LATEST 전략
  - Downstream으로 전달할 데이터가 버퍼에 가득찰 경우, 버퍼 밖에서 대기하는 가장 최근에 emit된 데이터부터 버퍼에 채운다.
  - DROP 전략과 달리 새로운 데이터가 들어오면 가장 최근 데이터를 제외하고 모두 삭제한다.
  - 버퍼밖에서 기다릴 수 없게 되면, 앞에 데이터를 모두 삭제하는 것이다. DROP은 하나씩만 제거한다.
  - `onBackpressureLatest()` Operator를 사용하면 된다.
- BUFFER 전략
  - Downstream으로 전달할 데이터가 버퍼에 가득찰 경우, 버퍼 안에 있는 데이터부터 DROP 시킨다.
  - DROP_LATEST: 버퍼가 가득찰 경우, 가장 나중에 채워진 데이터를 DROP한다.
  - DROP_OLDEST: 버퍼가 가득찰 경우, 가장 먼저 채워진 데이터를 DROP한다.
  - `onBackpressureBuffer()` Operator를 사용하면 된다.