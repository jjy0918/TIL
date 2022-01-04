# 3. Configuring and Running a Job

# 1. Configuring a Job

Job 인터페이스 구현체는 Builder로 추상화한다.

```java
@Bean
public Job footballJob(){
	return this.jobBuilderFactory.get("footballJob")
							.start(platerLoad()
							.next(gameLoad())
							.next(playerSummarization())
							.end()
							.build();
}
```

- 위 예시 Job은 세 개의 Step을 가지고 있다.
- Job 빌더로는 Step 뿐만 아니라, 병렬화(Split), 선언적 flow 제어(Decisiob), flow 정의 외부화(Flow) 같은 요소도 설정할 수 있다.

## 1.1 Restartability

- Job을 실행할 때 특정 JobInstance의 JobExecution이 이미 존재한다면, ‘재시작’으로 간주한다.
- Job이 주단된 지점부터 재시작할 수 없는 경우 개발자가 직접 새 JobInstance를 생성해야 한다.
- restartable 프로퍼티 값을 false로 설정하면, Job을 재실행하지 않고 항상 새 JobInstance를 실행한다.

```java
Job job = new SimpleJob();
job.setRestartable(false);

JobParameters jobParameters = new JobParameters();

// 첫 JobInstance 생성 시에는 에러 발생 x
JobExecution firstExecution = jobRepository.createJobExecution(job, jobParameters);
jobRepository.saveOrUpdate(firstExecution);

try{
	// 두 번째 JobInstance 생성 시 에러 발생
	jobRepository.createJobExecution(job, jobParameters);
	fail();
}catch(JobRestartException e){
	// 실행된다
}
```

- jobBuilder로 세팅 시 재시작 방지

```java
@Bean
public Job footballJob(){
	return this.jobBuilderFactory.get("footballJob")
							.prevnetRestart()
							...
							.build();
}
```

## 1.2 Intercepting Job Execution

- Job 실행 중 job의 라이프사이클 동안 발생한 이벤트를 통지 받아 원하는 코드를 실행할 수 있다.

```java
public interface JobExecutionListener{

	void beforeJob(JobExecution jobExecution);

	// afterJob은 성공 여부와 상관없이 호출된다
	// 성공/실패 여부는 JobExecution을 통해 알 수 있다.
	// jobExecution.getStatus() == BatchStatus.COMPLETED
	void afterJob(JobExecution jobExecution);

}

...

@Bean
public Job footballJob(){
	return this.jobBuilderFactory.get("footballJob")
							.listener(sampleListener())
							...
							.build();
}
```

- @BeforeJob, @AfterJob 어노테이션을 통해서도 설정할 수 있다.

## 1.3 JobParametersValidator

```java
@Bean
public Job footballJob(){
	return this.jobBuilderFactory.get("footballJob")
							.validator(parametersValidator())
							...
							.build();
}
```

# 2. Java Config

- @EnableBatchProcessing 어노테이션은 배치 Job 생성을 위한 기반 설정이라 할 수 있다.
- 이 기반 설정 내에서 주입 받는 여러 빈과 함께 StepScope 인스턴스를 생성한다.
  - JobRepository : “jobRepository”
  - JobLauncher : “jobLauncger”
  - JobRegistry : “jobRegistry”
  - PlatformTransactionManager : “transactionManager”
  - JobBuilderFactory : “jobBuilders”
  - StepBuilderFactory : “stepBuilders”

BatchConfigurer

- 배치 설정을 담당하는 인터페이스
- DefaultBatchConfigurere를 확장하고 필요한 getter를 오버라이딩 하여 사용할 수 있다.

```java
@Bean
public BatchConfigurer batchConfigurer(){
	return new DefaultBatchConfigurer(){
		@Override
		public PlatformTransactionManager getTransactionManager(){
			return new MyTransactionManager();
		}
	};
}
```

# 3. Configuring a JobRepository

- @EnableBatchProcessing을 이용하면 JobRepository를 바로 사용할 수 있따.
- BatchConfigurer 인터페이스를 구현하여 JobRepository 설정을 커스터마이징 할 수 있다.
- JobRepository 설정 중 dataSource와 transactionManager는 필수 값이다.(나머지 값은 디폴트 값이 존재한다.)

```java
@Override
protected JobRepository createJobRepository() throws Exception{
	JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
	factory.setDataSource(dataSource);
	factory.setTransactionManager(transactionManager);
	factory.setIsolationLevelForCreate("ISOLATION_SERIALIZABLE");
	factory.setTablePrefix("BATCH_");
	factory.setMaxVarCharLength(1000);
	return factory.getObejct();
}
```

## 3.1 Transaction Configuration for the JobRepository

- Transaction 설정을 할 수 있다.
- 디폴트 isolation 레벨은 SERIALIZABLE 이다.
- 그 외에 READ_COMMITED, READ_UNCOMMITTED 등이 있다.

## 3.2 Changing the Table Prefix

- 메타 데이터 테이블의 접두사도 JobRepository의 속성을 통해 바꿀 수 있다.
- 디폴트로 BATCH\_라는 접두사를 사용한다.
- 이 설정은 테이블 프리픽스만 변경한다.(테이블 명이나 컬럼 명은 해당되지 않는다.)

## 3.3 In-Memory Repository

- 도메인 오브젝트를 굳이 데이터베이스에 저장하지 않고 싶을 때 사용한다.
- 인메모리 레포지토리는 휘발성이어서 JVM을 새로 띄우는 식의 재실행은 불가능하다.
- 같은 파라미터로 두 개의 Job 인스턴스를 동시에 실행시킬 수 없으며, 멀티 쓰레드 Job이나 프로그램 내에서 파티셔닝한 Step에는 적합하지 않다.
- 인메모리 레포지토리도 트랜잭션 매니저를 등록해야 한다.

```java
@Override
protected JobRepository createJobRepository() throws Exception{
	MapJobRepositoryFactoryBean factory = new MapJobRepositoryFactoryBean();
	factory.setTransactionManager(transactionManager);
	return factory.getObejct();
}
```

## 3.4 Non-standard Database Types in a Repository

# 4. Configuring a JobLauncher

- @EnableBatchProcessing을 사용하면 JobRegistry가 제공된다.
- JobLauncher 인터페이스의 가장 기본적인 구현체는 SimpleJobLauncher이다.
- JobRepository 의존성만 있으면 실행할 수 있다.
- JobExecution에 의해 생성된 JobExecution은 job의 excute 메소드를 실행할 때 넘겨주고, 마지막에 caller에게 JobExecution을 리턴한다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled.png>)
- 비동기 처리 시 SimpleJobLauncher가 요청 즉시 caller에게 결과를 리턴해줘야 한다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled%201.png>)
- 이 경우에는 SimpleJobLauncher에 TaskExecutor를 설정하면 된다.

```java
@Bean
public JobLauncher jobLauncher(){
	SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
	jobLauncher.setJobRepository(jobRepository());
	jobLauncher.setTaskExecutor(new SimpleAsyncTaskExecutor());
	jobLauncher.afterPropertiesSet();
	return jobLauncher;
}
```

# 5. Running a Job

- 배치 Job을 실행하기 위해서는 실행할 Job과 JobLauncher가 필요하다.
- 커멘드라인에서 Job을 실행하게 되다면, 각 Job은 JVM이 따로 초기화되어 각 Job은 다른 JobLauncher에서 실행된다.
- 웹 컨테이너에서 HttpRequest로 Job을 실행한다면 여러 요청을 비동기로 실행하기 위해 JobLauncher 하나만 사용한다.

## 5.1 Running Jobs from the Command Line

- 엔터프라이즈 스케쥴러로 Job을 실행한다면, 기본적으로 Command Line으로 실행하게 된다.

### The CommandLIneJobRunner

- 스크립트로 Job을 실행하면, JVM으로 실행하기 때문에 진입점으로 메인 메소드를 가진 클래스가 필요하다.
- 스프링 배치는 CommandLineJobRunner 구현체를 이용해 이를 처리할 수 있게 지원한다.
- 작업 수행
  1. 적절한 ApplicationContext 로딩
  2. 커멘드라인에서 받은 인자를 JobPrameters로 변환
  3. 인자에 따라 적절한 Job선택
  4. 어플리케이션 컨텍스트에 있는 JobLauncher로 Job 실행
- 필수 인자
  - jobPath : ApplicationContext를 생성할 때 필요한 XML 파일 위치, 이 파일은 job 실행에 필요한 모든 것을 포함한다.
  - jobName : 실행할 job의 이름
  - 이 인자들은 path, name 순으로 전달해야 한다.
  - 이 두 인자 다음에 나오는 모든 인자는 JobParameter로 간주하며, name=value 포맷으로 사용한다.

```java
<bash& java CommandLIneJobRunner io.springEndOfJobConfiguration endOfDay
schedule.date(date)=2007/05/05

io.springEndOfJobConfiguration => Job을 포함한 설정 클래스의 풀네임
endOfDay => Job의 이름
schedule.date(date)=2007/05/05 => JobParameter로 변환된다.
```

### ExitCodes

- 스케줄러를 사용하는 경우 리턴 코드로 배치 잡의 성공/실패 여부를 판단해야 한다.
- 스프링 배치에선 ExitStatus로 캡슐화 하여 리턴 코드를 반환한다.
- CommandLineJobRunner가 ExitCodeMapper 인터페이스를 사용해 string 값을 number 로 바꿔준다.
- string으로 된 종료 코드를 받아서 숫자로 리턴하는게 ExitCodeMapper의 핵심이다.
- JobRunner에는 디폴트 구현체인 SimpleJvmExitCodeMapper가 있고, 성공 시에는 0, 일반적인 에러는 1, 컨텍스트 내 Job을 못찾는 등은 2를 리턴한다.
- 직접 구현한 ExitCodeMapper를 사용하려면 루트 레벨에 구현체를 선언하고 runner에 의해 로드되는 ApplicationContext에 포함시켜야 한다.

## 5.2 Running Jobs from within a Web Container

- 웹 애플리케이션을 지원해야 하는 등에는 비동기로 Job을 실행해야 한다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled%202.png>)
- 컨트롤러는 Job을 실행하고, 이때 요청 즉시 JobExecution을 리턴하게 설정한 비동기 JobLauncher를 사용한다.
- 비동기 JobLauncher를 실행하기 때문에 Job은 여전히 실행 중인데도 논블로킹 처리로 컨트롤러는 즉시 응답할 수 있다.

```java
@Controller
public class JobLauncherController{

	@Autowired
	JobLauncher jobLauncher;

	@Autowired
	Job job;

	@RequestMapping("/jobLauncher.html")
	public void handle() throws Exception{
		jobLauncher.run(job, new JobParameters());
	}

}
```

# 6. Advanced Meta-Data Usage

- JobLauncher와 JobRepository 인터페이스는 간단한 job 실행과 배치 도메인 오브젝트에 대한 기본 CRUD를 지원한다.
- JobLauncher는 JobExecution 오브젝트를 생성하고, 실행하기 위해 JobRepository를 사용한다.
- JobExplorer와 JobOperator 인터페이스는 메타 데이터를 질의하고 관리하기 위한 추가적인 기능을 제공한다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled%203.png>)

## 6.1 Querying the Repository

- JobExplorer 인터페이스는 레포지토리에 기존 execution을 요청하는 기능이다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled%204.png>)
- JobExplorer는 JobRepository의 리드온리 버전이다.
- JobRepository와 동일하게 팩토리 빈을 통해 손쉽게 설정할 수 있다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled%205.png>)

## 6.2 JobRegistry

- JobRegistry는 필수는 아니지만, 컨텍스트 내에 있는 Job을 추적하고 싶을 때 유용하다.
- 여러 곳에서 job을 생성하는 환경이라면, 어플리케이션 컨텍스트에서 job을 수집할 때도 사용할 수 있다.
- 커스텀 JobRegistry 구현체 또한 등록된 job의 이름이나 프로퍼티를 관리할 때 유용할 수 있다.
- 기본으로 제공하는 구현체는 job 이름을 job 인스턴스에 매핑한 간단한 map 기반이다.

## 6.3 JobOperator

- JobOperator를 이용하여 중단, 재시작, job 요약 등의 모니터링이 가능해진다.
  ![Untitled](<220105-Configuring_and_Running_a_Job(완성)/Untitled%206.png>)
- JobOperator는 JobLauncher, JobRepository, JobExplorer, JobRegistry 등의 인터페이스를 사용하는 인터페이스가 많아 의존성이 높다

## 6.4 JobParametersIncreamenter

- startNextInstance 메소드는 항상 새 Job 인스턴스를 실행시킨다.
- JobExecution에서 심각한 이슈가 발생하여 job을 처음부터 다시 시작해야 하는 경우 유용하다.
- JobLauncher는 이전 파라미터 셋과는 다른 값으로 새 JobInstance를 실행시키려면 새로운 JobParameters 오브젝트가 필요하다.
- startNextInstance 메소드는 Job에 상응하는 JobParametersIncreamenter를 통해 새 인스턴스를 만들도록 강제한다.
- JobParametersIncreamenter는 가지고 있는 JobPrameters에서 필요한 값을 증가시켜 다음에 사용될 JobParameters 오브젝트를 리턴한다.
- increamenter는 빌더에서 제공하는 incrementer 메소드로 설정한다.
