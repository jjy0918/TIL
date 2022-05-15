# Chapter2. 카프카 환경 설정

## 2.1 카프카 환경 설정

### 카프카 힙 메모리 설정

- kafka는 힙 메모리를 `bin/kafka-server-start.sh`에서 설정한다.
- `bin/kafka-server-start.sh`에서 환경 변수 `KAFKA_HEAP_OPS`를 기준으로 힙 메모리를 설정한다.
  - `KAFKA_HEAP_OPS`가 설정되어 있지 않다면 기본 값인 `-Xmx1G -Xms1G`로 설정된다.

### daemon

- `bin/kafka-server-start.sh`를 실행할 때 `-daemon` 옵션을 붙여주면 백그라운드에서 실행하게 할 수 있다.

### server.properties

- `server.properties`를 통해 카프카 브로커 옵션을 설정할 수 있다.
- 이미 실행중인 브로커 설정을 변경하기 위해서는 브로커를 재시작해야 한다.
- `broker.id=0`: 브로커를 식별하기 위한 값.
- `listener=PLAINTEXT://:9092`: 카프카 브로커의 IP와 Port, 설정하지 않을 경우 모든 IP와 Port로 설정된다.
- `advertised.listeners=PLAINTEXT://192.168.0.20`: 카프카 클라이언트, 카프카 커맨드 라인 툴에서 접속하는 IP, Port
- `num.partitions=1`: 토픽 생성 시 기본 파티션 개수
- `log.retention.hours=168`: 데이터가 삭제되기 까지 시간
- `log.segment.bytes=1024`: 카프카 브로커가 저장할 파일의 최대 크기. 최대 크기를 채우면 새로운 파일이 생성된다.
- `zookeper.connect=localhost:2181`: 카프카 브로커와 연동할 주키퍼의 IP, Port

### 주키퍼

- 주키퍼는 카프카의 클러스터 설정 리더 정보, 컨트롤러 정보를 담고 있다.
- 주키퍼 == 브로커들을 통제하는 시스템

## 2.2 카프카 커맨드 라인 툴

### 2.2.1 kafka-topics.sh

- 토픽과 관련된 명령을 실행할 수 있다.

#### 토픽 생성

- 토픽 생성은 생성되지 않은 토픽에 대해 데이터를 요청하거나, 커맨드 라인 툴로 명시적 생성이 가능하다.
- 카프카 클러스터 정보, 토픽 이름은 필수 값이다.
- 파티션 개수, 복제 개수, 토픽 데이터 유지 기간 등 옵션도 설정할 수 있다.

```
kafka-topics.sh \
--create \
--bootstrap-server my-kafka:9092 \
--partitions 3 \
--replication-factor 1 \
--config retention.ms=17280000 \
--topic test.hello.

```

#### 토픽 리스트 조회

- `--list` 옵션을 통해 리스트 조회가 가능하다

```
kafka-topics.sh --bootstrap-server my-kafka:9092 --list
```

#### 토픽 상세 조회

- `--describe` 옵션을 통해 상세 조회가 가능하다.

```
kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic test.hello.
```

#### 토픽 옵션 수정

- `kafka-topcis.sh` + `--alter`: 파티션 개수 수정(파티션 개수는 늘릴 수는 있지만, 줄일 수는 없다.)
- `kafka-configs.sh`: 리텐션 기간(토픽 삭제 정책) 변경, 다이나믹 정책(log.segment.bytes, log.retentions.ms 등)

### 2.2.2 kafka-console-producer.sh

- 메시지 보내는 기능
- 레코드 = 메시지 키 + 메시지 값
- 레코드 값은 UTF-8 기반으로 Byte로 변환된다.
- 메시지 키 값을 입력하지 않은 경우는 null로 처리 되며, 라운드 로빈으로 레코드를 배치한다.
- 메시지 키 값이 존재하는 경우 해시 값을 기준으로 파티션 중 하나를 할당 받는다.

### 2.2.3 kafka-console-consumer.sh

- 메시지를 확인하는 기능
- 메시지 간 순서는 같은 파티션 사이에서만 적용되기 때문에 키 설정이 중요하다.
- 각 컨슈머 그룹은 가져간 토픽의 메시지에 대한 오프셋을 저장한다.

### 2.2.4 kafka-consumer-groups.sh

- `kafka-consumer-groups.sh` 에서 `--list` 옵션을 이용하여 컨슈머 그룹 리스트를 확인할 수 있다.
- `lag`은 컨슈머 그룹이 커밋한 오프셋과 해당 파티션의 가장 최신 오프셋 간의 차이점이다.
  - `lag`을 바탕으로 컨슈머의 처리 상태를 확인하여 최적화할 수 있다.

### 2.2.5 kafka-verifiable-producer, consumer.sh

- kafka-verifiable로 시작하는 스크립트를 이용하여 String 메시지 값 코드 없이 데이터를 주고 받을 수 있다.
- 주로 테스트할 때 사용된다.

### 2.2.6 kafka-delete-records.sh

- 적재된 토픽을 지우는 방법.
- 특정 레코드를 지우는 것이 아니라, 토픽에 존재하는 레코드 중 오래된 오프셋부터 지정한 오프셋까지 삭제된다.
