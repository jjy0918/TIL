# 18. Spring Data R2DBC

## 18.1 R2DBC란?

- R2DBC(Reactive Relational Database Connectivity)는 리액티브 프로그래밍을 지원하기 위한 관계형 데이터베이스의 개방형 사양이면서, 드라이버 벤더가 구현하고 클라이언트가 사용하기 위한 SPI(Service Provider Interface)
- R2DBC 이전
  - 몇몇 NoSQL 에서만 리액티브 프로그래밍을 지원.
  - RDB 에서는 JDBC API 자체가 Blocking API이기 때문에 완전한 Non-Blocking I/O 지원이 불가능.
- R2DBC 이후
  - RDB에서도 Non-Blocking I/O 를 지원하기 때문에 리액티브 프로그래밍이 가능해짐.
- R2DBC 지원 RDB
  - H2, MySQL, jaync-sql MySQL, MariaDB, Microsoft SQL Server, Postgres, Oracle 등

## 18.2 Spring Data R2DBC란?

- Spring Data R2DBC란 R2DBC를 좀 더 쉽게 구현할 수 있게 도와주는 Spring Data Family 프로젝트.
  - Spring Data Jpa 와 유사함.
  - 여타 Spring Data Family 프로젝트와 동일하게 데이터 액세스 계층의 보일러플레이트 코드 양을 대폭 줄일 수 있음.
- Spring Data R2DBC는 JPA와 같은 ORM에서 제공하는 캐싱, 지연 로딩 등의 기능들을 제거한 상태에서 심플하게 사용할 수 있음.

## 18.3 Spring Data R2DBC 설정

- Spring Data R2DBC를 사용하기 위해서 Spring Data R2DBC 스타터와 사용하려는 데이터베이스 밴더사에 맞는 R2DBC용 드라이버 추가 필요.

```
dependencies {
    implementation("com.oracle.database.r2dbc:oracle-r2dbc")
    implementation("org.springframework.data:spring-data-r2dbc")
}
```

- R2DBC Repository와 Auditing 기능을 활성화하기 위해서는 애플리케이션 엔트리포인트에 `@EnableR2dbcRepositories`와 `@EnableR2dbcAuditing` 추가가 필요함.
  - `@EnableR2dbcRepositories`: ReactiveCrudRepository, R2dbcRepository 등의 Repository를 상속한 interface를 구현체로 만든 후 Bean으로 등록
  - `@EnableR2dbcAuditing`: Entity의 `@CreatedBy`, `@CreatedDate`, `@LastModifiedBy` 등의 기능을 사용할 수 있도록 허용.

```java
@EnableR2dbcRepositories
@EnableR2dbcAuditing
@SpringBootApplication
public class Chapter18Application {

    public static void main(String[] args) {
        SpringApplication.run(Chapter18Application.class, args);
    }

}
```

## 18.4 Spring Data R2DBC에서의 도메인 엔티티 클래스 매핑

- 여타 Spring Data Family에서 사용하던 것과 동일하게 작성

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Setter
public class Book {
    @Id
    private long bookId;
    private String titleKorean;
    private String titleEnglish;
    private String description;
    private String author;
    private String isbn;
    private String publishDate;

    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column("last_modified_at")
    private LocalDateTime modifiedAt;
}
```

## 18.5 R2DBC Repositories를 이용한 데이터 엑스스

- 여타 Spring Data Family에서 사용하던 것과 동일하게 Repository를 쉽게 구현할 수 있다.
- 다른 점은 리액티브를 지원하기 때문에 `ReactiveCrudRepository`를 상속해야 하며 리턴 타입이 `Mono` 또는 `Flux` 다.


```java
public interface BookRepository extends ReactiveCrudRepository<Book, Long> {
    Mono<Book> findByIsbn(String isbn);
}
```

## 18.6 R2dbcEntityTemplate을 이용한 데이터 액세스

- Spring Data R2DBC에서는 Repository를 이용하는 것이 아니라 SQL 쿼리문을 작성하는 것 같이 메서들 조합하여 실행하는 `R2dbcEntityTemplate`을 제공한다.
- R2dbcEntityTemplate은 JdbcTemplate 처럼 SQL을 직접 작성하는 것이 아니라 JPA에서 사용하는 Query DSL과 유사하다.
- R2dbcEntityTemplate은 SQL 시작 구문인 SELECT, INSERT, UPDATE, DELETE는 select(), insert(), update(), delete() 메소드로 대응되며 이를 `Entrypoint method`라고 한다.
- Entrypoint method 외에 select() 와 함께 사용할 수 있는 `Termination method`가 존재한다.
  - first(), one(), all(), count(), exists()
  - Termination method 는 SQL문을 생성하고 최종적으로 실행한다.
- SQL 시작 구문 외 SQL 연산자에 해당하는 `Criteria method` 가 존재한다.
  - and(), or(), is() 등

```java
@Service("bookServiceV6")
@RequiredArgsConstructor
public class BookService {
    private final @NonNull R2dbcEntityTemplate template;
    private final @NonNull CustomBeanUtils<Book> beanUtils;

    public Mono<Book> saveBook(Book book) {
        return verifyExistIsbn(book.getIsbn())
                .then(template.insert(book));
    }

    public Mono<Book> updateBook(Book book) {
        return findVerifiedBook(book.getBookId())
                .map(findBook -> beanUtils.copyNonNullProperties(book, findBook))
                .flatMap(updatingBook -> template.update(updatingBook));
    }
    ...

    public Mono<List<Book>> findBooks() {
        return template.select(Book.class).all().collectList();
    }

    private Mono<Void> verifyExistIsbn(String isbn) {
        return template.selectOne(query(where("ISBN").is(isbn)), Book.class)
                .flatMap(findBook -> {
                    if (findBook != null) {
                        return Mono.error(new BusinessLogicException(
                                ExceptionCode.BOOK_EXISTS));
                    }
                    return Mono.empty();
                });
    }

    ...
}
```

## 18.7 Spring Data R2DBC에서의 페이지네이션(Pagination) 처리

- Repository에서 페이지네이션은 다른 Spring Data Family와 유사하게 Pageable을 사용하여 처리할 수 있다.
- R2dbcEntityTemplate 에서는 limit(), offset(), sort() 등의 쿼리 빌드 메서드를 조합하여 처리할 수 있다.
- 그 외에 Reactor의 Operator 체인을 이용하여 직접 처리할 수도 있다.

```java
    public Mono<List<Book>> findBooks(@Positive long page, @Positive long size) {

        return template
                .select(Book.class)
                .count()
                .flatMap(total -> {
                    Tuple2<Long, Long> skipAndTake = getSkipAndTake(total, page, size);
                    return template
                            .select(Book.class)
                            .all()
                            .skip(skipAndTake.getT1())
                            .take(skipAndTake.getT2())
                            .collectSortedList((Book b1, Book b2) ->
                                    (int) (b2.getBookId() - b1.getBookId()));
                });
    }
```