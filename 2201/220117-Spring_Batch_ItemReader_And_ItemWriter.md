# 5. ItemReaders and ItemWriters

# 5.1 ItemReader

- 데이터 입력을 읽는 부분.
- 플랫(flat) 파일 : 필드가 고정된 위치에 있거나, 특정한 특수문자(쉼표 등)로 필드를 구분하는 파일
- XML : 파싱, 매핑, 검증에 사용되는 기술과는 독립적으로 XML을 처리한다. 입력 데이터 유효성은 XSD 스키마로 검증한다.
- Database : 객체에 매핑되는 resultset을 얻어온다.
    - 디폴트로 SQL ItemReader 구현체는 RowMapper를 호출하여 오브젝트를 리턴한다.
    - 재시작에 대비해 현재 row를 추적하고, 기본적인 통계를 저장하고, 트랜잭션을 제공한다.

```java
public interface ItemReader<T> {

    T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;

}
```

- read 메소드는 아이템이 존재하면 아이템 하나를 리턴하거나, 없으면 null을 리턴한다.
    - 아이템이 없어도 예외가 발생하지 않는다.
    - 결과가 0개 라면, 예외가 아니라 null을 리턴한다는 것이다.
- 아이템 하나라는 것은 하나의 row가 될 수도 있고, xml의 엘리먼트 하나가 될 수도 있다.
- ItemReader 구현체는 앞으로만 읽어야 한다. 즉, 역행하면 안된다.(forward only)
    - 트랜잭션 처리가 있다면, read 메소드는 롤백 후 다시 호출해도 같은 아이템을 리턴해야 한다.

# 5.2 ItemWriter

- ItemReader와 비슷하지만, 읽는 것이 아니라 쓴다.

```java
public interface ItemWriter<T> {

    void write(List<? extends T> items) throws Exception;

}
```

- 리소스가 열려있다면, 전달 받은 아이템 리스트를 write 한다.
- 일반적으로는 청크 단위로 묶어서 결과물을 만들기 때문에 ItemReader와 달리 아이템 하나가 아니라 아이템 리스트를 받는다.
- write 메소드가 결과를 반환하기 전 flush 처리를 수행한다.

# 5.3 ItemProcessor

- write 작업 전 비즈니스 로직을 작업하는 부분이다.
- composite 패턴을 적용한 ItemWriter나 ItemReader에서도 비즈니스 로직을 작성할 수 있다.
    - 이러한 패턴을 사용하는 경우는, 다른 참조 데이터를 읽거나 write, read 메소드를 직접 제어하고 싶을 때 사용한다.

```java
public class CompositeItemWriter<T> implements ItemWriter<T> {

    ItemWriter<T> itemWriter;

    public CompositeItemWriter(ItemWriter<T> itemWriter) {
        this.itemWriter = itemWriter;
    }

    public void write(List<? extends T> items) throws Exception {
        //Add business logic here
       itemWriter.write(items);
    }

    public void setDelegate(ItemWriter<T> itemWriter){
        this.itemWriter = itemWriter;
    }
}
```

일반적인 `변환` 의 경우 ItemProcessor를 사용한다.

- 객체 하나를 받아 다른 객체로 변환해서 반환한다.
- 새로 반환되는 객체는 같은 타입일 수도, 아닐 수도 있다.
- 반환되는 과정 전 비즈니스 로직을 수행한다.

```java
public interface ItemProcessor<I, O> {

    O process(I item) throws Exception;
}
```

## 5.3.1 Chainig ItemProcessors

- 하나의 ItemProcessor를 통한 변환이 아니라, 여러 변환이 필요한 경우 사용된다.
- composite 패턴이 적용된 CompositeItemProcessor를 사용한다.

```java
// foo -> bar
public class FooProcessor implements ItemProcessor<Foo,Bar>{
    public Bar process(Foo foo) throws Exception {
        //Perform simple transformation, convert a Foo to a Bar
        return new Bar(foo);
    }
}

// bar -> foobar
public class BarProcessor implements ItemProcessor<Bar,Foobar>{
    public Foobar process(Bar bar) throws Exception {
        return new Foobar(bar);
    }
}

@Bean
public CompositeItemProcessor compositeProcessor() {
	List<ItemProcessor> delegates = new ArrayList<>(2);
	delegates.add(new FooProcessor());
	delegates.add(new BarProcessor());

	CompositeItemProcessor processor = new CompositeItemProcessor();

	processor.setDelegates(delegates);

	return processor;
}

// 각 processor를 등록하는게 아니라, chaining된 compsiteItemProcessor를 등록한다.
@Bean
public Step step1() {
	return this.stepBuilderFactory.get("step1")
				.<String, String>chunk(2)
				.reader(fooReader())
				.processor(compositeProcessor())
				.writer(foobarWriter())
				.build();
}
```

## 5.3.2 Filtering Records

- ItemProcessor 과정 중 필터링을 적용할 수 있다.
- skip은 데이터가 유효하지 않는 경우 발생하고, filtering은 데이터를 write 하지 않겠다는 것이다.
- 필터링한 데이터가 발생하는 경우 null을 리턴하면 된다.
- null이 리턴되는 경우 프레임워크는 자동으로 ItemWriter에 전달되는 아이템 리스트에서 제외한다.
- null을 리턴하는게 아니라, 예외를 발생시키는 경우 skip이 발생된다.

## 5.3.3 Fault Tolerance

- 청크가 롤백되면, read 시 캐시된 아이템을 다시 처리하는 경우 사용된다.
- falut torlerance가 설정된 step이라면, 모든 ItemReader는 idempotence를 보장해야 한다.
- 일반적으로 ItemProcessor의 입력 데이터는 바꾸지 않고, 결과로 사용될 인스턴스만 바꾸는 식으로 구현한다.

# 5.4 ItemStream

- read과정을 수행하기 전 리소스를 open해야 하고, write 과정 후에는 리소스를 close 해야 한다.
- ItemStream은 이러한 과정을 수행함과 동시에 상태를 저장하는 매커니즘을 가지고 있다.

```java
public interface ItemStream {

    void open(ExecutionContext executionContext) throws ItemStreamException;

    void update(ExecutionContext executionContext) throws ItemStreamException;

    void close() throws ItemStreamException;
}
```

- ItemReader의 read가 실행되기 전 open을 통해 리소스에 접근할 수 있다. 이는 ItemWriter의 wirte도 마찬가지이다.
- ExecutionContext에 데이터가 있다면, 초기 상태(이미 실행했던 것)가 아니기 때문에 ItemReader와 ItemWriter를 실행할 때 함께 사용된다.
- close는 열려 있는 모든 리소스를 안전하게 닫기 위해 호출된다.
- update는 현재까지 진행된 모든 상태를 ExecutionContext에 저장할 때 사용한다.
    - 커밋 전 데이터베이스에 현재 상태를 저장하려면 커밋 전에 호출하면 된다.
- ItemStream으로 Step을 구현하는 경우, StepExecution마다 ExecutuonContext를 생성하여 각 실행 상태를 저장한다.
    
    그리고 같은 JobInstance가 실행되면 이 값을 넘겨준다.
    

# 5.5 The Delegate Pattern and Registering with the Step

- CompositeItemWriter는 delegation 패턴 중 하나로 구성되어 있다.
- 위임 받는 객체 자체가 StepListener 같은 콜백 인터페이스를 구현하는 경우도 있다.
- Step에서 Delegate 패턴을 사용하게 된다면, 대부분 수동으로 Step에 등록해야 한다.
- ItemStream 이나 StepListner 인터페이스를 Step과 직접 연결하는 reader, writer, processor로 구현 시 자동 등록된다.
    - 다만, Step은 어떤 객체가 위임됐는지 모르기 때문에 직접 연결해야 한다.

```java
@Bean
public Step step1() {
	return this.stepBuilderFactory.get("step1")
				.<String, String>chunk(2)
				.reader(fooReader())
				.processor(fooProcessor())
				.writer(compositeItemWriter())
				.stream(barWriter())
				.build();
}

@Bean
public CustomCompositeItemWriter compositeItemWriter() {

	CustomCompositeItemWriter writer = new CustomCompositeItemWriter();

	writer.setDelegate(barWriter());

	return writer;
}

@Bean
public BarWriter barWriter() {
	return new BarWriter();
}
```