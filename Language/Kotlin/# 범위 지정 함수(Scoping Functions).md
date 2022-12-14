# 범위 지정 함수(Scoping Functions)

- Kotlin 에는 범위 지정 함수 5가지가 존재한다.
- apply, also, let, run, with

## apply

```kotlin
inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this
}
```

- apply는 확장 함수 형태로, 해당 객체를 리턴한다.
- 확장 함수 형태로 구현되기 때문에 객체의 프로퍼티나 메소드 접근 시 `this` 또는 생략이 가능하다.
- 일반적으로 `내부 프로퍼티`를 `변환`할 때 사용된다.
- `확장 함수` 형태로 구현되기 때문에 당연하게 `private` 프로퍼티 및 메소드는 접근할 수 없다.

```kotlin
val person = Person("123", 123).apply {
    this.method()
    this.age = 3
    method()
    age = 4
}
```

## also

```kotlin
inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}
```

- 람다 block 내부에 해당 객체를 전달하여 로직을 수행한 후 해당 객체를 리턴한다.
- 람다 내부에 객체를 전달하기 때문에 객체 접근 시 `it`을 이용할 수 있다.
- 일반적으로 수신 객체를 사용하지 않거나 수신 객체의 속성을 변경하지 않는 경우 사용한다.

### let

```kotlin
inline fun <T, R> T.let(block: (T) -> R): R {
    return block(this)
}
```

- let은 일반적으로 객체가 null이 아닌 경우 어떠한 동작을 하도록 할 때 사용한다.
- let도 block 람다에 객체를 전달하기 때문에 객체 접근 시 `it`을 사용해야 한다.
- 람다의 결과 값을 리턴한다.

```kotlin
Person()?.let {
    validate(it)
}
```

### run

```kotlin
inline fun <T, R> T.run(block: T.() -> R): R {
    return block()
}
```

- run은 확장 함수 형태로 구현되기 때문에 객체 접근 시 `this` 또는 생략할 수 있다.
- 람다의 결과 값을 리턴한다.

### with

```kotlin
inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}
```

- with는 수신 객체를 명시적으로 선언한다 외에는 run과 동일하다.

## 정리

- 객체 접근 차이
  - `it`으로 객체 접근: also, let
  - `this`로 객체 접근: apply, run, with
  - 여러 수신 객체가 겹치는 경우 적절히 선택
- 리턴 값 차이
  - 리턴 값이 객체: apply, alsㅐ
  - 리턴 값이 람다 결과: let, run, with
- 객체가 null이 아닌 경우 실행하는 엘비스 연산자(?.)는 모든 경우 사용할 수 있다.