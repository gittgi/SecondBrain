# JDBC

```table-of-contents
```

##  JDBC란?

- 어플리케이션의 중요 데이터들은 대부분 데이터베이스에 보관
	- 주로 TCP/IP 방식으로 커넥션을 연결
	- SQL을 통해 DB에 데이터 전달 및 조회
- 그러나 데이터베이스마다 커넥션을 연결하는 방법부터 SQL을 전달하고 응답을 받아오는 방법이 달라짐
	- 사용하는 데이터베이스를 변경하고자 할 때, 어플리케이션 서버에 작성한 코드들도 모두 고쳐야하는 문제점
	-  데이터베이스마다 커넥션 연결, SQL 전달 및 응답 방식을 따로 숙지해야함
- 위와 같은 문제점을 해결하기 위해, 자바에서 데이터베이스에 접속할 수 있도록 만든 자바 API가 JDBC 표준 인터페이스


### JDBC의 대표 기능 3가지

- 다음 3가지 기능을 표준 인터페이스로 정의
	- `java.sql.Connection` : 연결
	- `java.sql.Statement` : SQL을 담은 내용
	- `java.sql.ResultSet` : SQL 요청 응답
- DB 벤더 들은 이 인터페이스에 맞춰 구현한 JDBC 드라이버를 라이브러리로 제공하고, 개발자들은 이 드라이버를 통해 쉽게 DB에 접근
	- 다른 DB 사용시, 어플리케이션은 JDBC 인터페이스에만 의존하기 때문에, 해당 DB에 맞춘 라이브러리만 변경하면 됨
	- 개발자 역시 다양한 DB 사용방법을 모두 학습하는 대신, JDBC 표준 인터페이스만 학습하고 적용하면 됨
		- 다만 SQL이나 데이터타입 등의 일부 사용법의 경우에는 데이터베이스마다 아직 차이가 날 수 있기 때문에, SQL 등은 해당 데이터베이스에 맞게 바꿀 필요가 있을 수 있다.
			- 이 부분은 [JPA](../미완성%20문서/JPA.md)를 사용하면 다시 또 상당부분 공통화할 수 있다.

### JDBC 접근 기술

- JDBC 역시 좀 더 편리하게 접근하기 위해서 두가지의 기술 사용
	- SQL Mapper
	- ORM

#### SQL Mapper
- JDBC를 편리하게 사용하도록 도와줌
	- SQL 응답 결과를 객체로 편리하게 반환
	- JDBC 사용시의 반복 코드를 제거
- 다만 개발자가 직접 SQL 작성해야 함
- spring JdbcTemplate, MyBatis 등이 대표적

#### ORM
- 객체와 RDBMS의 테이블을 매핑해주는 기술
- SQL을 직접 작성하는 대신 ORM이 동적으로 SQL을 만들어서 실행
	- 따라서 DB마다 조금씩 차이나는 문법도 ORM이 중간에서 맞춰줌
- 대표적으로 JPA, 하이버네이트, 이클립스링크
	- JPA는 자바의 ORM 표준 인터페이스, 하이버네이트와 이클립스 링크는 이를 구현한 것


## JDBC에서 데이터 연결

- JDBC가 제공하는 `DriverManager.getConnection(..)`를 사용해서 연결
- 라이브러리에 있는 데이터베이스 드라이버를 통해 커넥션을 반환
- 반환된 커넥션은 `java.sql.Connection`을 구현한 구현체
```java
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.SQLException;  
  
import static hello.jdbc.connection.ConnectionConst.*;  
  
@Slf4j  
public class DBConnectionUtil {  
    public static Connection getConnection() {  
        try {  
            Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);  
            return connection;  
        } catch (SQLException e) {  
            throw new IllegalStateException(e);  
        }  
    }
```

### 커넥션 요청 흐름
- `DriverManager.getConnection(..)`호출
- `DriverManager`는 라이브러리에 등록된 드라이버 목록을 인식하고,  각 드라이버에게 커넥션에 필요한 정보를 넘겨서 처리할 수 있는 요청인지 확인
	- 이때 넘기는 정보는 다음과 같다.
		- URL : jdbc:h2:tcp://localhost/~/test 
			- 앞부분의 jdbc:h2을 h2 DB의 드라이버가 인식하고 작동
		- 이름, 비밀번호 등 접속에 필요한 추가 정보
- 처리할 수 있다는 커넥션 구현체를 찾으면 이를 반환


## JDBC 저장/수정/삭제 예제

```java

@Slf4j  
public class MemberRepositoryV0 {  
  
    public Member save(Member member) throws SQLException {  
        String sql = "insert into member(member_id, money) values (?, ?)";  
  
        Connection con = null;  
        PreparedStatement pstmt = null;  
  
  
        try {  
            con = getConnection();  
            pstmt = con.prepareStatement(sql);  
            pstmt.setString(1, member.getMemberId());  
            pstmt.setInt(2, member.getMoney());  
            pstmt.executeUpdate();  
            return member;  
        } catch (SQLException e) {  
            log.error("db error--------------", e);  
            throw e;  
        } finally {  
            close(con, pstmt, null);  
        }
	}

	private void close(Connection con, Statement stmt, ResultSet rs) {  
	  
	    if (rs != null) {  
	        try {  
	            rs.close();  
	        } catch (SQLException e) {  
	            log.error("error", e);  
	        }  
	    }  
	  
	    if (stmt != null) {  
	        try {  
	            stmt.close();  
	        } catch (SQLException e) {  
	            log.error("error", e);  
	        }  
	    }  
	    if (con != null) {  
	        try {  
	            con.close();  
	        } catch (SQLException e) {  
	            log.error("error", e);  
	        }  
	    }  
	}  
	  
	private static Connection getConnection() {  
	    return DBConnectionUtil.getConnection();  
	}
  
}

```

- `getConnection()` : 위에서 만든 DBConnectionUtil을 통해 커넥션을 획득
- `con.preparedStatement(sql)` : 데이터베이스에 전달할 SQL과 파라미터로 전달할 데이터들을 준비
	- `pstmt.setString(몇번째, member.getMemerId)` : setString을 통해 ?에 값을 지정, 숫자형의 경우 setInt()사용 
	- PreparedStatement는 Statement의 자식 타입으로, [파라미터 바인딩](../미완성%20문서/파라미터%20바인딩.md) 방식을 통해 SQL Injection 예방 할 수 있음
- `pstmt.executeUpdate()` : 준비된 Statement를 커넥션을 통해 실제 데이터베이스에 전달
	- 반환값으로는 영향을 받은 DB row수를 반환

> [!Warning] 리소스 정리
> - 쿼리를 실행한 후에는 리소스 정리 필수
> - Connection -> PreparedStatement 순으로 사용했기 때문에, 정리는 역순으로 해야함
> 	- PreparedStatement를 먼저 종료하고 Connection을 종료
> - 리소스 정리의 경우 (예시에서는 `close()`) 예외와 상관없이 항상 수행되어야 한다. (finally)
> 	- 리소스 정리가 실행되지 않는 경우, 커넥션이 계속 유지되고 이것은 리소스 누수로 이어져 커넥션 부족이라는 장애가 발생할 수 있다.

- 수정, 삭제의 경우에는 쿼리문을 update, delete문을 작성해서`executeUpdate(sql)`로 할 수 있음


## JDBC 조회 예제

```java
	public Member findById(String memberId) throws SQLException {  
	    String sql = "select * from member where member_id = ?";  
	  
	    Connection con = null;  
	    PreparedStatement pstmt = null;  
	    ResultSet rs = null;  
	  
	    try {  
	        con = getConnection();  
	        pstmt = con.prepareStatement(sql);  
	        pstmt.setString(1, memberId);  
	  
	        rs = pstmt.executeQuery();  
	  
	        if (rs.next()) {  
	            Member member = new Member();  
	            member.setMemberId(rs.getString("member_id"));  
	            member.setMoney(rs.getInt("money"));  
	            return member;  
	        } else {  
	            throw new NoSuchElementException("member not found : memberId = " + memberId);  
	        }  
	  
	    } catch (SQLException e) {  
	        log.error("db error-----------", e);  
	        throw e;  
	    } finally {  
	        close(con, pstmt, rs);  
	    }  
	}

```
- `rs = pstmt.executeQuery()` : 데이터를 조회할 때 쓰는 것은 `executeQuery()`, 결과는 `ResultSet`에 담겨 반환됨
- **ResultSet**
	- select 쿼리의 결과가 순서대로 들어감
	- ResultSet 내부에 있는 커서(cursor)를 이동시켜서 다음 데이터를 조회 가능
		- `rs.next()` : 이것을 호출하면 커서가 다음으로 이동 (최초 커서는 데이터를 가리키지 않기 때문에, 최초 한번의 실행으로 첫 데이터를 가리킴)
		- `rs.next()`의 값이 참이면 커서의 이동 위치에 데이터가 있다는 뜻, 거짓이면 더이상 데이터가 없다는 뜻
	- rs.getString(), rs.getInt()로 현재 커서가 가리키는 위치의 데이터를 가져온다
- NoSuchElementException : 조회 결과가 없는 경우, NoSuchElementException이 발생

## JdbcUtils

- 스프링은 JDBC를 편하게 다룰 수 있는 JdbcUtils라는 편의 메서드 제공
- 이를 사용하면 커넥션을 좀 더 편하게 닫을 수 있다.

```java
private void close(Connection con, Statement stmt, ResultSet rs) {
	JdbcUtils.closeResultSet(rs);
	JdbcUtils.closeStatement(stmt);
	JdbcUtils.closeConnection(con);
}

```


## JdbcTemplate

- [레포지토리](../미완성%20문서/@Repository.md)에서 JDBC 를 사용할 때는 반복이 많이 발생
	- 커넥션 조회, 커넥션 동기화
	- `PreparedStatement` 생성 및 파라미터 바인딩
	- 쿼리 실행
	- 결과 바인딩
	- 예외 발생시 스프링 예외 변환기 실행
	- 리소스 종료
- 이런 문제를 해결하기 위해 [템플릿 콜백 패턴](../미완성%20문서/템플릿%20콜백%20패턴.md)을 도입한 JdbcTemplate이라는 템플릿을 제공
	- JdbcTemplate을 적용하면 [트랜잭션](../CS/트랜잭션.md)을 위한 [커넥션 동기화](../Spring/스프링과%20트랜잭션.md)부터 [스프링 예외 변환](../Spring/Exception/데이터%20접근%20예외.md) 까지 자동으로 적용해줌
- 자세한 내용은 [JdbcTemplate](../Spring/데이터%20접근%20기술/JdbcTemplate.md) 참조