# DataSource

```table-of-contents
```

##  DataSource란?

- DB 커넥션을 얻는 방법은 다양
	- [JDBC](../JAVA/JDBC.md)의 `DriverManager`를 직접 사용
	- `hikariCP`등의 [커넥션 풀](../CS/커넥션%20풀.md) 오픈소스를 사용 등
- 이처럼 어플리케이션이 `DriverManager`나 `HikariCP`등에 직접 의존하게 되면, 이를 변경하는데 많은 수정이 필요할 것이고 또한 둘의 사용법이 조금씩 달라지기 때문에 역시 추가적인 코드 수정이 불가피
- 따라서 자바는 이런 문제를 해결하기 위해 `javax.sql.DataSource`라는 인터페이스를 제공
- DataSource는 커넥션을 획득하는 방법을 추상화한 인터페이스
- DataSource의 핵심 기능은 커넥션 조회
```java
// 핵심 기능만 나타낸 수도 코드
 public interface DataSource {
    Connection getConnection() throws SQLException;
}
```

- 대부분의 커넥션 풀은 DataSource 인터페이스를 구현해 두었기 때문에, 개발자는 DataSource 인터페이스에만 의존하도록 어플리케이션 로직을 작성
	- `DriverManager`는 DataSource 인터페이스를 사용하지는 않지만, 스프링에서 DriverManager도 DataSorce를 통해 쓰 수 있도록 `DriverManagerDataSource`라는 구현체를 제공

## DataSource 예시

```java
// 그냥 DriverManager 사용
@Test  
void driverManager() throws SQLException {  
    Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);  
    Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);  
    log.info("connection = {}, class = {}", con1, con1.getClass());  
    log.info("connection = {}, class = {}", con2, con2.getClass());  
}  
  
@Test  
void dataSourceDriverManager() throws SQLException {  
    DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);  
    useDataSource(dataSource);  
  
}  
  
@Test  
void dataSourceConnectionPool() throws SQLException, InterruptedException {  
    //커넥션 풀링  
    HikariDataSource dataSource = new HikariDataSource();  
    dataSource.setJdbcUrl(URL);  
    dataSource.setUsername(USERNAME);  
    dataSource.setPassword(PASSWORD);  
    dataSource.setMaximumPoolSize(10);  
    dataSource.setPoolName("myPool");  
    useDataSource(dataSource);  
    Thread.sleep(1000); // 커넥션 풀에서 커넥션을 생성하는 작업은 어플리케이션 실행속도에 영향을 주지 않기 위해 별도의 쓰레드에서 작동 -> 별도의 쓰레드이기 때문에 테스트가 먼저 종료되어 버림... 대기시간을 강제로 주는 것으로 로그 확인 할 수 있도록 함 
  
}  
  
private void useDataSource(DataSource dataSource) throws SQLException {  
    Connection con1 = dataSource.getConnection();  
    Connection con2 = dataSource.getConnection();  
  
    log.info("connection = {}, class = {}", con1, con1.getClass());  
    log.info("connection = {}, class = {}", con2, con2.getClass());  
}
```

- DriverManager의 경우에는 생성시 마다 URL, USERNAME, PASSWORD를 넘겨야 하는 반면에, DataSource를 사용하는 방식은 처음 객체를 생성할 때만 필요한 파라미터를 넘겨두고, 커넥션을 획득할 떄는 단순히 `dataSource.getConnection()`만 호출하면 됨
	- 이는 DataSource를 설정하는 부분과 사용하는 부분을 명확히 나눠서, 사용하는 부분에서는 URL, USERNAME, PASSWORD 등에 의존하지 않고 DataSource만 주입받아 사용할 수 있게 할 수 있다. 
	- 한 곳에서 설정한 것으로 여러곳에서 사용할 수 있다.
- useDataSource를 만들때, DataSource만 주입해주면 어떤 방식으로 DataSource를 만들었는지와는 상관없이 똑같이 사용 가능
	- 이는 곧, **DataSource를 외부에서 [의존관계 주입](../CS/디자인%20패턴/의존관계%20주입.md) 하여 사용**할 수 있다는 뜻
	- 리포지토리가 DataSource 인터페이스에만 의존하고 있다면, DriverManagerDataSource에서 HikariDataSource로 변경해서 주입한다 하더라도 문제가 되지 않는다


## 스프링 부트의 자동 리소스 등록

### 데이터 소스 자동 등록
- 스프링 부트 이전에는 DataSource와 [트랜잭션 매니저](스프링과%20트랜잭션.md)를 직접 [스프링 빈](스프링%20빈.md)으로 등록해서 사용
```java
 @Bean
DataSource dataSource() {
	return new DriverManagerDataSource(URL, USERNAME, PASSWORD);
}

@Bean
PlatformTransactionManager transactionManager() {
	return new DataSourceTransactionManager(dataSource());
}

```

- 그러나 [스프링부트](../미완성%20문서/SpringBoot.md)는 DataSource를 자동으로 등록
	- 자동 등록된 스프링 빈 이름 : `dataSource`
	- 직접 DataSource를 등록하는 경우, 자동으로 등록하지 않는다
- DataSource를 자동 등록하기 위해 필요한 정보들은 application.properties(yml)에서 찾아서 등록
```properties
# MySql  
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver  
spring.datasource.url=jdbc:mysql://localhost:3306/book_store?characterEncoding=UTF-8&serverTimezone=UTC  
spring.datasource.username=root
spring.datasource.password=
```

```yml

spring:  
  datasource:  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    url: jdbc:mysql://127.0.0.1:3306/book_store  
    username: book_reg  
    password: book_reg
    
```

- 스프링 부트가 기본으로 생성하는 DataSource는 [커넥션 풀](../CS/커넥션%20풀.md)방식의 `HikariDataSource`
	- 커넥션 풀 관련 설정도 application.porperties를 통해 지정
- `spring.datasource.url`속성이 없으면 메모리 DB 로 내장 데이터베이스 생성을 시도

### 트랜잭션 매니저 자동 등록
- 스프링 부트는 적절한 [[스프링과 트랜잭션|트랜잭션 매니저](PlatformTransactionManager)를 자동으로 스프링 빈에 등록
	- 자동 등록된 스프링 빈 이름 : `transactionManager`
	- 직접 등록하는 경우, 자동등록 되지 않는다.
- 어떤 트랜잭션 매니저를 선택할지는 현재 등록된 라이브러리를 보고 판단
	- [JDBC](../JAVA/JDBC.md) -> `DataSourceTransactionManager`
	- [JPA](../미완성%20문서/JPA.md) -> `JpaTransactionManager`

### 자동 등록된 빈 사용 예시

```java
// 수동 등록
@TestConfiguration  
static class TestConfig {  
    @Bean  
    DataSource dataSource() {  
        HikariDataSource dataSource = new HikariDataSource();  
        dataSource.setJdbcUrl(URL);  
        dataSource.setUsername(USERNAME);  
        dataSource.setPassword(PASSWORD);  
        return dataSource;  
    }  
  
    @Bean  
    PlatformTransactionManager transactionManager() {  
        return new DataSourceTransactionManager(dataSource());  
    }  
  
    @Bean  
    MemberRepositoryV3 memberRepositoryV3() {  
        return new MemberRepositoryV3(dataSource());  
    }  
  
    @Bean  
    MemberServiceV3_3 memberServiceV3_3() {  
        return new MemberServiceV3_3(memberRepositoryV3());  
    }  
}

// 자동 등록
@TestConfiguration  
static class TestConfig {  
  
    private final DataSource dataSource;  
  
    public TestConfig(DataSource dataSource) {  
        this.dataSource = dataSource;  
    }  
  
    @Bean  
    MemberRepositoryV3 memberRepositoryV3() {  
        return new MemberRepositoryV3(dataSource);  
    }  
  
    @Bean  
    MemberServiceV3_3 memberServiceV3_3() {  
        return new MemberServiceV3_3(memberRepositoryV3());  
    }  
}
```
- DataSource와 트랜잭션 매니저가 자동으로 등록되어 있기 때문에, 직접 생성해서 넣지 않는다.
- application.properties를 기반으로 생성되어 [스프링 컨테이너](스프링%20컨테이너.md)에 보관되어 있는 DataSource를 [의존관계 주입](../CS/디자인%20패턴/의존관계%20주입.md) 및 [의존관계 자동 주입](의존관계%20자동%20주입.md) 받을 수 있다.