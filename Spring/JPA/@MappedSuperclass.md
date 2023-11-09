# @MappedSuperclass

- 공통적인 매핑 정보가 필요한 경우(예 : createdAt, createdBy 등), 모든 엔티티에 직접 칼럼들을 복사 붙여넣기 하는 대신 객체의 상속을 통해서 관리 가능
- [상속관계 매핑](상속관계%20매핑.md) 도 아니고, 엔티티가 새로 생기는 것도 아니고, 테이블과 매핑하는 것도 아님
- 단순히 상속 받는 자식클래스에게 매핑 정보(칼럼)만 제공
	- 따라서 이 클래스는 조회, 검색(em.find) 불가
	- 직접 생성할 일이 없는 클래스이므로 추상클래스로 만드는 것이 적절

- 참고로 @Entity 가 붙은 클래스가 상속 받을 수 있는 클래스는 마찬가지로 @Entity가 붙은 클래스 이거나([상속관계 매핑](상속관계%20매핑.md)) @MappedSuperclass가 달려있는 클래스만 가능 

```JAVA
// 매핑 정보를 담고 있을 클래스
@MappedSuperclass
public abstract class BaseEntity {

	private String createdBy;
	private LocalDateTime createdAt;
	private String modifiedBy;
	private LocalDateTime modifiedAt;
	
}

// 이제 매핑 정보를 필요한 엔티티에 공통적으로 상속 가능
@Entity
public class Member extends BaseEntity {
	...
}

@Entity
public class Item extends BaseEntity {
	...
}

```



---
