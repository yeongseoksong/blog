---
{"dg-publish":true,"permalink":"//jpa/jpa/"}
---


```table-of-contents

style: nestedList

minLevel: 0

maxLevel: 0

includeLinks: true

debugInConsole: false

```
# 1.  Spring Transaction 
```java
@SpringBootTest  
//@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)  
public class SpringBootJpaTest {  
  
    @Autowired  
    EntityManager em;  
    @Autowired  
    EntityManagerFactory emf;  
  
    @Test  
        // 트랜잭션 없이 엔티티 매니저 사용  
    void 트랜잭션_없이(){  
        User  user = new User();  
        user.setId(1L);  
        user.setMsg("테스트");  
        em.persist(user);  
    }  
    @Test  
    void 트랜잭션_명시적으로_호출(){  
        EntityManager em = emf.createEntityManager();  
        EntityTransaction transaction=em.getTransaction();  
        try {  
                transaction.begin();  
                User entity = new User();  
                entity.setMsg("Test Entity");  
                em.persist(entity);  
                transaction.commit(); }  
  
        catch (Exception e) {  
                transaction.rollback();  
            }        finally {  
                em.close();  
        }    }  
    @Test  
    @Transactional    void 트랜잭션_어노테이션_사용() {  
        User user = new User();  
        user.setId(1L);  
        user.setMsg("테스트");  
        em.persist(user);  
    }}
```


# 2. Orm 이란? 

## 2.1 Object-Relational Mapping
---
- **객체 지향 언어**인 자바와 **RDBMS** 의 패러다임은  일치 하지 않는다.
	- 객체는 **참조** 를 통한 **단방향** 연관관계
	- RDBMS 는 **왜래키**를 통한 **양방향** 연관 관계
	- 객체는 **클래스**를 사용
	- RDBM 는 **테이블**을 사용

```java
public class Member{
	Team team; // 단방향	
}
```

```sql
SELECT * 
FROM member m 
LEFT JOIN team t 
ON m.team_id = t.team_id;
```


- 패러 다임이 일치하지 않기 때문에  `Dao` 을 사용해 쿼리문을 작성하고 쿼리 수행 결과를 객체에 매핑하는 작업이 필요했다.
	- 요구사항이 변경 되면 매번 복잡한 쿼리를 변경해야 했으나 객체 지향 코드로 쿼리 작업을 수행할 수 있게되어 유지 보수에 용이해지기도 했다.

- 두 간극을 해결 하기 위해 ORM 을 사용한다. JPA 는 객체의 참조 관계를 RDBMS 의 **외래키 관계로 자동으로 변환**한다.
	- Object - Relational Mapping 해석 그대로 객체와 RDBMS 데이터를 **매핑**한다.


## 2.2  JPA (Java Persistence API)
---
- 자바 진영의 ORM 표준 명세이다. 즉 인터페이스를 의미하며 여러 구현체들이 존재한다.
	- **Hibernate**
	- OpenJpa

![jpa-jdbc over view.png](/images/jpa-jdbc%20over%20view.png)


- 참고로 JPA 는 **JPQL(Java Persistence Query Language)** 을 내부적으로 사용하며 JPQL 는 결국 JDBC 를 사용해 **Native SQL** 로 변환 된다는 점을 기억해야 한다.
- 또, ORM 을 사용하면 어플리케이션이 데이터 베이스의 종류에 종속적이지 않게 된다.
	- Mysql 과 Oracle 만해도  Ansi 표준 sql 만 공유할 뿐이지  구조나 구체적인 sql 문법에 차이가 존재한다.
	- Spring boot 와 ORM 을 사용하면 build.gradle 에 설치된 패키지 정보를 보고 자동으로 특정 DB 문법에 맞는 Native SQL 을 생성해낸다.


```java
@Entity
public class Member{...}
```

```java
String jpql = "SELECT m FROM Member m"; 
TypedQuery<Member> query = entityManager.createQuery(jpql, Member.class); 
List<Member> resultList = query.getResultList();
```

```sql
select * from member
```



# 3. 영속성 관리

## 3.1 EntityMangerFactory
---
![EntityMangerFatory em.png](/images/EntityMangerFatory%20em.png)

- 어플리케이션 하나에는 단 하나의 `EntityMangerFactory` 가 존재한다. Spring 에서 Jpa 를 사용할 경우에 컨테이너에 등록되어 사용할 수 있다.
	- `EntityManagerFatory` 는 생성 비용이 크기 때문이다.
		- **커넥션 풀**을 생성하고 JPA 기반 객체를 생성해야한다. ( **HiKariCP** 사용 )
		  *Spring Boot 를 사용하면 자동으로 `DataSource` 를 application.yml 파일에서 읽어와 생성한다.* 
- **Thread Safe** 해 여러 스레드가 접근해 **EntityManger  생성**을 처리할 수 있다.



## 3.2 EntityManger
---
![영속성 컨텔스트와 엔티티 매니저.png](/images/%EC%98%81%EC%86%8D%EC%84%B1%20%EC%BB%A8%ED%85%94%EC%8A%A4%ED%8A%B8%EC%99%80%20%EC%97%94%ED%8B%B0%ED%8B%B0%20%EB%A7%A4%EB%8B%88%EC%A0%80.png)
- `Entity`의 생명 주기를 관리하는 역할을 수행
- `EntityMangerFactory` 에 의해 생성되며 실제로 **커넥션 풀을 사용**해 DB 에 접근하는 객체이다.
  *DB 접근이 실제로 필요할 때만 커넥션을 사용한다.*
- **영속성 컨텍스트 직접 관리한다.**
  *이를 통해 엔티티를 저장, 수정, 삭제, 조회 등의 작업을 수행*
- `EntityManger` 는 Thread Safe 하지 않아 여러 스레드에서 공유할 수 없다.
- Spring 에서 JPA 사용 시에 **기본 정책**은 `EntityManger` 는 `@Transactional` 이 시작될 때 생성된다. 
	- 정확히는 처음에 Proxy 를 주입받고 `@Transactional` 시작 시점에 실제  `EntityManger` 가 초기화된다.
	- **영속성 컨텍스트가 함께 생성된다.**
	- OSIV 를 사용하면 View 진입 전에 영속성 컨텍스트를 활성화할 수 있다.

```java
@PersistenceUnit private EntityManagerFactory entityManagerFactory;

@PersistenceContext private EntityManager entityManager;

```

```java
transaction = em.getTransaction(); 
try { 
	transaction.begin(); 
	MyEntity entity = new MyEntity(); 
	entity.setName("Test Entity"); 
	em.persist(entity); 
	transaction.commit(); } 
catch (Exception e) { 
		transaction.rollback(); } 
finally { 
	em.close(); 
	emf.close(); }
```

| `EntityManagerFactory` | `EntityManager`     |
| ---------------------- | ------------------- |
| 애플리케이션 전역에서 공유 가능      | 요청마다 생성해야 함         |
| 스레드 안전(thread-safe)    | 스레드 안전하지 않음         |
| 리소스가 많고 무겁다            | 가벼운 객체로 트랜잭션 단위로 사용 |
| 데이터베이스 연결 풀 관리         | **영속성 컨텍스트 관리**     |

## 3.3 영속성 컨텍스트
---
- 영속성 컨텐스트란 `엔티티를 영구 저장하는 환경`이라는 뜻이다. 
- 애플리케이션과 데이터베이스 사이에서 객체를 보관하는 **가상의 데이터베이스** 같은 역할을 한다. **( 1차 캐시 )**
-  엔티티 매니저를 통해 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다



```java
@Autowired
EntityManger em;
...
em.persist(member); // 엔티티를 영속성 컨텍스트에 저장한다.
```



> 엔티티 매니저가 생성 될 때 영속성 컨텍스트가 함께 생성 된다고 했다. 즉  Spring 에서 Jpa 를 사용하면 기본적으론 @Transactional 을 만날때  영속성 컨텍스트 또한 생성된다.

### 3.3.1 엔티티 생명 주기

![entity lifecycle.png](/images/entity%20lifecycle.png)
- **비영속; new/tranisent** :영속성 컨텍스트와 전혀 관계 없는 상태
  - `new Member()` 로 순수한 객체 상태일 때
- **영속; managed** : 영속성 컨텍스트에 저장된 상태
	- `em.find(), em.persist(E)` 
- **준영속; detached** :영속성 컨텍스트에 저장되었다가 분리된 상태
	- `em.detach(E), em.close(), em.clear()`
- **삭제; removed** : 삭제된 상태
	- `em.remove(E)`

###  3.3.2 1차 캐시, 쓰기 지연
#### 1차 캐시

| @ID | Entity    |
| --- | --------- |
| 1L  | member(A) |
| 2L  | member(B) |
*영속성 컨텍스트에 엔티티는 위와같이 Pk (ID) 를 기준으로 저장된다. HashMap<String, Object> 를 생각하면 편하다. *

```java
em.find(Member.class,1L);
```

- 이후에 저장한 `Member` 를 찾는 로직이 존재하면, Jpa 는 영속성 컨텍스트에서 해당 객체를 반환한다. => **동일성 조건**을 만족한다.

#### 쓰기 지연
- `insert, delete`  쿼리들은 JPA 내부에 저장소에 저장해 두었다가 트랜잭션이 종료되는 시점에 쿼리를 실제 DB 에 전달한다.
- 영속성 컨텍스트에 존재하는 **쓰기 저장소**의 내용을 실제 DB에 저장하는 행위를 **플러시** 라 한다. 
```java
em.flush(); // DB 와 PersistenContext 를 동기화 !
```

- **플러시**가 실행되는 케이스는 다음과 같다.
	- 위와 같이 **명시적으로 호출**
	- **트랜잭션 종료 시**에 자동 호출
	- **JPQL 쿼리**시 자동 호출
	  *JPQL 은 쓰기 저장소를 거치 않고 SQL 로 변환 후에 DB 로 직접 접근하여 값을 가져온다. 이러한 특성 덕에 기존의 쿼리를 실제 DB 에 반영하지 않으면 데이터의 불일치가 생길 수 있기 때문이다. *


### 3.3.3 변경 감지 

| @Id | 엔티티     | 스냅샷           |
| --- | ------- | ------------- |
| 1L  | memberA | memberA 최초 상태 |
| 2L  | memberB | memberA 최초 상태 |

- 영속화된 엔티티가 수정이 됐을 때 JPQ 는 이러한 변경점들을 자동으로 DB 에 반영해줄 수 있는데, 이를  **변경 감지** 라 한다.
- **변경 감지**는 최초에 엔티티가 영속화 됐을 때의 **스냅 샷**을 남겨 두었다가  `em.flush()` 가 호출 되는 시점에 스냅 샷과 현재 영속화된 엔티티를 비교해 `update` 쿼리를 DB 에 보내준다.


### 3.3.4  영속성 컨텍스트 생명 주기 

![영속성 컨텍스트 생존 범위.png](/images/%EC%98%81%EC%86%8D%EC%84%B1%20%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%20%EC%83%9D%EC%A1%B4%20%EB%B2%94%EC%9C%84.png)
- 트랜잭션 과 영속성 컨텍스트가 같은 생명 주기를 공유하는 트랜잭션 범위의 영속성 컨텍스트 전략
	- `open-in-view : false`
- **실행 흐름**
	- `@service` 에서 `@Transactional` 을 실행하면 AOP 가 먼저 실행되어 트랜잭션을 시작한다.
	- 이때부터 영속성 컨텍스트가 존재해 엔티티들을 영속화한다.
	- 서비스가 종료되고 트랜잭션이 종료될 때, 영속성 컨텍스트 또한 종료된다.
	- 영속성 컨텍스트가 종료됐기때문에 `@Controller` 로 반환되는 엔티티는 준영속 상태의 엔티티이다.
-  **트랜잭션이 같으면 같은 영속성 컨텍스트를 사용한다**.
	- `@service` 에서 여러 `@repository` 를 사용하고 엔티티 매니저를 주입 받아 사용하는데, 같은 트랜잭션이므로 영속성 컨텍스트 또한 공유한다.
- **트랜잭션이 다르면 다른 영속성 컨텍스트를 사용한다.**
	- 멀티 스레드 환경에서 같은 엔티티 매니저를 사용하더라도 **트랜잭션이 다른 경우 접근 가능한 영속성 컨텍스트**가 다르다.
	- *스프링 컨테이너는 스레드마다 다른 트랜잭션을 할당한다.*
### 3.3.4 OSIV
**TestController.java**
```java
  
//open-in-view: true 로 변경 혹은 삭제 후 실행  
@GetMapping("/osiv-test")  
public List<Comment> create(){  
    Post post = postService.findById(1L);  
    post.getComments().size();  
    return  post.getComments();  
}
```
- `open-in-view: false` 로 위의 로직을 수행하면 `LazyInitializationException` 이 발생한다.
	- 영속성 컨텍스트가 존재하지 않는 범위에서(entity 가 준영속 상태일 때) 프록시를 초기화 하려고 했기 때문이다.
- `open-in-view: true` 로 설정하면 정상적으로 결과가 수행되는 것을 알 수 있는데 이는 **OSIV** 를 사용했기 때문에 그렇다.
- **OSIV** 는 3.3.4 와 다르게 영속성 컨텍스트를 **인터셉터, 필터** 에서부터 미리 켜둔다.
	- 서비스 계층에서 `@Transactional` 을 만나면 영속성 컨텍스트를 활성화하는 기존 전략과 다른점이다.
	- 이로써, `Controller` 계층에서도 영속성 컨텍스트를 사용할 수 있게된다.

**Controller 에서는 Transaction 이 존재하지 않는다.**
-  `Service` 계층에서 `@Transactional` 을 만나 `tx.begin,... commit(),em.flush()` 가 `AOP` 에 의해 주입된다고 앞서 설명했다.
- 즉, `Controller` 계층에서는 트랜잭션이 종료된 이후에 값이 반환되게 된다.

**트랜잭션 없이 영속성 컨텍스트 사용**
- 영속성 컨텍스트의 모든 변경은 **트랜잭션 안**에서 이루어져야 한다.
- **단, 지연 로딩을 포함하여 단순히 엔티티를 읽는 경우에는 트랜잭션이 없어도 가능하다**

| 영속성 컨텍스트 | 트랜잭션 | 동작     |
| -------- | ---- | ------ |
| O        | X    | 조회     |
| O        | O    | 조회, 수정 |

> `controller` 계층에서 지연로딩된 엔티티를 초기화할 수 있는 것은 개발의 편리성을 증대 시켜준다. 그러나 트랜잭션 커넥션을 불필요하게 많이 점유하게 된다는 단점이 있다.
> 이를 해결하기 위해 `jpql fetch join` 을 사용하거나 초기화는 모두 `service` 계층에서 수행한 후에 `controller` 계층으로 넘겨 주어야한다.
> 더 나아가 `jpql fetch join` 을 좀더 편리하게 사용하기 위해서 `query dsl`을 사용하는 것이다. 


# 4. 엔티티 매핑 
#### @Id
**User.java**

```java

@Entity  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@Table(name = "user")  
@Builder  
@AllArgsConstructor  
@ToString  
@Setter  
@Getter   
public class User {  
  
    @Id  
    @Column(name ="user_id")  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    private String name;  
  
  
    public static User init(String name){  
        User user = new User();  
        user.setName(name);  
        return user;  
    }  

}
```

**Post.java**
```java
@Builder  
@Entity  
@Table(name = "post")  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@AllArgsConstructor  
@ToString(exclude = "user")  
@Setter  
@Getter  
public class Post {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "post_id", nullable = false)  
    private Long id;    
  
  
    public static Post init(User user,String content){  
        return Post.builder()  
                .user(user)  
                .content(content)  
                .status(PostStatus.PENDING)  
                .build();  
    }    
}
```

- `@Id` : 테이블의 식별자를 지정해 준다. 이때 
	- `@Id`  적용 가능 타입
		- 기본형
		- wrapper
		- String
		- Date, BigDemical, BigInteger
 -  `@GeneratedValue(strategy = GenerationType.IDENTITY)`  **기본 키 생성을 DB에 위임하는 전략**이다.
   *MySql과 같이 Sequence를 제공하지 않고, `AutoIncrement` 기능을 제공*


#### @MappedSuperClass

```java
@MappedSuperclass  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@Getter  
@Setter  
@EntityListeners(AuditingEntityListener.class)  
public class BaseEntity {  
  
    @CreatedDate  
    @Column(updatable = false)  
    private LocalDateTime createdAt;  
  
    @LastModifiedDate  
    private LocalDateTime updatedAt;  
    }
```

```java

public class User extends BaseEntity{  
}
```

**Post.java**
```java
public class Post extends BaseEntity{  
}

```

- `@MappedSuperclass` 는 실제 테이블과 매핑 되지 않고 **정보를 자식 테이블에 상속할 목적**으로 사용한다.


### @Enumerated
**PostStatus.java**

```java
public enum PostStatus {  
    PENDING("대기 중"){  
        @Override  
        public boolean isViewable() {  
            return false;  
        }    },    APPROVED("승인됨"){  
        public boolean isViewable() {  
            return true;  
        }    },    BLOCKED("차단됨"){  
        public boolean isViewable() {  
            return false;  
        }    };  
  
    private final String description;  
  
    PostStatus(String description) {  
        this.description = description;  
    }  
    public String getDescription() {  
        return this.description;  
    }  
    public abstract boolean isViewable();  
}
```

**Post.java**

```java
   @Enumerated(value = EnumType.STRING)  
    PostStatus status;  
```

- `@Enumerated` : enum 타입을 매핑할 수 있게 해준다.  아래 값들을 `value` 로 지정해 줄 수 있다.
	- `EnumType.STRING`  
	- `EnumType.ORDINAL



# 5. 연관관계 매핑

- 테이블은 **조인**을 통해 양방향 연관관계를 맺는다. 객체의 경우는 단방향만 가능하다.
  *객체에서 참조는 단방향만 가능하지만, 단방향 두개의 합해 양방향처럼 표현한다.*
	- `@OneToMany`
	- `@ManyToOne`
	- `@OneToOne`

**User.java**

```java

@Entity  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@Table(name = "user")  
@Builder  
@AllArgsConstructor  
@ToString  
@Setter  
@Getter   
public class User {  
  
    @Id  
    @Column(name ="user_id")  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    private String name;  
  
	@OneToMany( fetch = FetchType.LAZY,mappedBy = "user", cascade =CascadeType.ALL,orphanRemoval = true)  
	@Builder.Default  
	private List<Post> posts= new ArrayList<>();
}
```

**Post.java**
```java
@Builder  
@Entity  
@Table(name = "post")  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@AllArgsConstructor  
@ToString(exclude = "user")  
@Setter  
@Getter  
public class Post {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "post_id", nullable = false)  
    private Long id;    

	@JsonIgnore  
	@ManyToOne(fetch = FetchType.LAZY)  
	@JoinColumn(name = "user_id" ,nullable = false)  
	private User user;
  
    public static Post init(User user,String content){  
        return Post.builder()  
                .user(user)  
                .content(content)  
                .status(PostStatus.PENDING)  
                .build();  
    }    
}
```

## 5.1 연관관계 주인
---
![1-n 관계.png](/images/1-n%20%EA%B4%80%EA%B3%84.png)
- **JPA에서 FK를 관리하는 엔티티를 '연관관계의 주인' 이라고 부른다.**
- 주인이 아닌 필드에 `@mappedBy` 를 사용한다.
- 주인인 필드는 외래키와 매핑 되어 키를 관리할 수 있다.
	- DB 에 해당하는 테이블에 외래키 칼럼이 존재하게 된다.
- `mappedBy` 칼럼은 외래키를 읽기만 할 수 있다.
- **연관관계의 주인만 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있다. 주인이 아닌 반대편은 읽기만 가능하고 외래 키를 변경하지 못한다**.
	- 예를들어 `member.posts.remove(post)` 를 하더라도 실제 DB 에 존재하는 post 가 사라지는 것은 아니다.
	- `mappedBy` 를 갖는 엔티티는 읽기만 가능하며, 객체 그래프를 탐색하기 위해 존재한다.

> **연관관계 주인은 테이블에서 외래키가 존재하는 곳으로 정한다. 1:N 관계에서 N 인 테이블**  

## 5.2 연관관계 편의 메소드

**post.java**
```java
public void setUser(User user){
	if(getUser()!=null)
		user.getPosts().remove(this);
	this.user= user;
	user.getPosts().add(this);
}
```

**postTest.java**

```java
@Test  
@Transactional  
void 연관관계_편의_메소드(){  
    User user1 = userFactory.create();  
    User user2 = userFactory.create();  
    Post post = postFactory.create();  
  
    post.setUser(user1);  
    em.persist(user1);  
    em.persist(user2);  
    em.persist(post);  
  
    em.flush();  
    em.clear();  
   
    User findUser1 = em.find(User.class, user1.getId());  
    Post post1 = findUser1.getPosts().get(0);  
    post1.setUser(user2);  
    em.flush();  



    Assertions.assertThat(findUser1.getPosts().size()).isEqualTo(0);  
    Assertions.assertThat(user2.getPosts().size()).isEqualTo(1);  
}
```

- `findUser1` 엔티티에 기존에 넣어주었던 `post` 엔티티가 존재할 것이다.
- 이때, `post` 엔티티의 연관 관계를 `user2` 로 변경해주면 **변경 감지** 에 의해  `em.flush()` 가 호출 될 때 업데이트 쿼리가 실행된다.
- 그러나 연관 메소드가 없으면, `findUser1` 엔티티의 `posts` 에는 여전히 `post` 엔티티가 남아있게된다.
- **실제 DB 에는 연관 관계 변경 (외래키 변경) 이 일어났지만, 자바 메모리 상에서는 변경이 일어나지 않았다.
- 이러한 문제 때문에 **연관 관계 편의 메소드**를 정의하여 자바 메모리 상의 연관 관계 또한 일치 시켜준다.





