# Spring Batch에서 TransactionManager 사용하기

## @EnableBatchProcessing

Spring Batch에서 @EnableBatchProcessing 어노테이션을 선언한 클래스에서는

```
JobRepository, JobLauncher, JobExplorer, JobRepository, JobRegistry,
PlaformTransactionManager,
JobBuilderFactory
StepBuilderFactory
```

의 클래스를 빈으로 등록하여 사용할 수 있다.

```
@EnableBatchProcessing
public BatchTest {

    @Autowired
    private final JobBuilderFactory kobBuilderFactory;

}
```

여기서 StepBuilderFactory 클래스는 생성자로 2개의 파라미터 JobRepository와 PlatformTransactionManager를 받는다.

즉, PlatformTransactionManager도 @EnableBatchProcessing에 의해서 Bean으로 등록되어 있다.
이때 그 값은 `DataSourceTransactionManager` 이다.

<br>

## JpaWriter

JpaWriter는 TransactionManager로 JpaTransactionManager를 사용한다.

즉, @EnableBatchProcessing에 의해서 기본으로 사용되는 DataSourceTransactionManager는 jpaWriter에서 문제가 발생할 수 있다.

그렇기 때문에 jpaTransactionManager를 따로 Bean으로 등록하여 설정해주어야 한다.
