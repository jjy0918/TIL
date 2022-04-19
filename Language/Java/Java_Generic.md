# Generic

## Generic을 사용해야 하는 이유

1. Generic을 통해 컴파일 시 강한 타입 체크를 할 수 있다.

2. 타입 캐스팅을 제거할 수 있다.

## 제네릭 타입

제네릭 타입이란 타입을 파라미터로 가지는 클래스와 인터페이스를 말한다.

```java

// T: 타입 파라미터

public class ClassName<T> { ... }

public interface InterfaceName<T> { ... }

```

일반적으로, 타입 파라미터는 대문자 알파벳 한 글자로 표현한다.

실제 코드에서는 타입 파라미터에 구체적인 타입을 지정해야한다.

```java

ClassName<String> c;

```

## 멀티 타입 파라미터

제네릭에서 타입 파라미터는 하나가 아니라 여러개 가질 수 있다.

```java

class Product<T, K> {

...

}

Product<String, Car> product = new Product<>();

```

Java7부터는 다이어몬드 연산자(<>) 를 제공하여 자바 컴파일러가 타입 파라미터를 유추하여 자동으로 설정해준다.

## 제네릭 메소드

제네릭은 클래스나 인터페이스 말고 메소드에서도 사용할 수 있다.

<타입 파라미터..> 리턴타입 함수이름(매개변수)

```java

public GenericTest {

public <I, S> genericMethod(I i, S s){

...

}

}

```

## 제한된 타입 파라미터

일반적으로 타입 파라미터에는 어떠한 타입도 들어갈 수 있다.

하지만, 일부 기능에서는 제한된 타입만 받아야 하는 경우가 있다. 그때 사용하는 것이 제한된 타입 파라미터이다.

타입 파라미터에 extends 키워드를 이용하여, 해당 클래스나 인터페이스를 구현한 타입만 사용할 수 있도록 제한할 수 있다.

```java

// Integer 타입이나 Integer 타입을 구현한 클래스와 인터페이스만 올 수 있다.

public class Test<T extends Integer> {

...

}

```

## 와일드 카드 타입

제네릭에서는 매개값이나 리턴 타입으로 사용 시 와일드 카드 타입을 이용하여 제한을 할 수 있다.

- `<?>`: 제한 없음
- `<? extends T>`: T타입이거나 T타입의 자식들만 올 수 있다.
- `<? super T>`: T타입이거나 T타입의 부모들만 타입만 올 수 있다.

## 제네릭 상속

제네릭 클래스도 상속을 할 수 있다.

상속을 받는 클래스는 부모 클래스의 제네릭 파라미터를 그대로 받고, 추가도 할 수 있다.

```java

public P<T>{

public void test(T t) { ... }

}

public C<T> extends P<T>{

public void test(T t){ ... }

}

```
