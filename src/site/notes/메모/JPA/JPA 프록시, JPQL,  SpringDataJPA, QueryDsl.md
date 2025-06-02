---
{"dg-publish":true,"permalink":"//jpa/jpa-jpql-spring-data-jpa-query-dsl/"}
---

```table-of-contents

style: nestedList

minLevel: 0

maxLevel: 0

includeLinks: true

debugInConsole: false

```

# 6.  프록시
## 6.1 지연 로딩
---
- `@-ToMany`  의 경우 해당 엔티티가 조회 될 때 엔티티들이 자동으로 **지연 로딩으로 실행된다.**
- 앞서 실제 실행되는 JQPL 을 보면  엔티티를 조회할 때  해당 엔티티만 조회한 이후에 연관 엔티티는 실제 사용될 때 추가적으로 조회 쿼리가 수행되었었다.
- 그 이유가 바로 JPA 의 **지연 로딩** 덕분이다.

> @-ToOne 은 설정을 해주지 않으면 즉시 로딩으로 실행된다. **연관된 엔티티를 하나 조회 하는 것은 성능상 문제가 되지 않는 반면 여러개 (1:n) 관계 에서는 성능에 문제가 될 수 있기 때문이다.** 


## 6.2 즉시 로딩
---
- **jpa** 는 즉시 로딩 시에 연관 객체를 모두 DB 로 부터 가져온다.
- **이때, jpa 는 select 문이 2번 수행되지 않고 join 쿼리를 실행해 쿼리를 최적화 한다.**

| 로딩 종류     | 함수                    | 비고                   |
| --------- | --------------------- | -------------------- |
| **즉시 로딩** | `FectchType.EAGER`    | 연관 객체를 함께 조회한다.      |
| **지연 로딩** | `FectchType.LAZY`<br> | 연관 객체를 사용 시점에 조회 한다. |


# 6.3  프록시
---
- JPA 는 지연 로딩을 **프록시** 를 통해 구현한다.
- `em.getReference` 를 사용하면, JPA는 데이터 베이스를 조회하지 않고 프록시 객체를 반환한다. 프록시 객체는 실제 객체가 사용되는 시점 까지 DB 조회를 미룬다.
- 프록시는 실제 클래스를 상속받아 만들고 프록시 객체는 실제 객체에 대한 참조(target)를 보관한다.
- **프록시 객체의 메소드를 호출 하면 타겟의 메소드를 호출 한다.**

```java

class MemberProxy extends Member{

	Member target= null;
	
	public String foo (){
		if(target==null){
			//1. 초기화 요청
			//2. DB 조회
			//3. 실제 엔티티 생성 및 보관
			this.target= ...
		}
		return target.foo();
	}
}
```

- 영속성 컨텍스트에 이미 엔티티가 존재하면 프록시가 아닌 실제 엔티티를 반환한다.
- **준영속 상태의 프록시를 초기화 하려하면 에러 발생** (LazyInitializationException)가 발생

### 6.3.1 프록시 초기화

```java
@Test  
public void foo(){  
    MemberProxy reference = em.getReference(MemberProxy.class, 1L);  
    Long id = reference.getId(); // accesstype.field 여도 프록시가 초기화 되지 않음
    System.out.println("id = " + id);  
    Long id2 = reference.findId();  //findXX로 빈 규약에 맞지 않아 메서드가 어떤 일을 하는지 알지 못하므로 객체가 초기화 된다.
}
```

- `em.getReference` 는 `MemberProxy` 엔티티의 프록시 객체를 반환한다. 
- 프록시는 엔티티의 식별자를 저장하고 있다. 따라서 식별자를 조회하더라도 프록시는 초기화되지 않는다.

```java
@DataJpaTest  
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)  
  
public class InitProxy {  
    @Autowired  
    EntityManager em;  
  
    @Test  
    @Transactional    
    void OneToOne_에서_주인에서_Lazy로_조회(){  
        ParentOne parentOne = new ParentOne();  
        SonOne sonOne = new SonOne();  
        sonOne.setParentOne(parentOne);  
        em.persist(sonOne);  
        em.flush();  
        em.clear();  
    
  
        SonOne sonOne1 = em.find(SonOne.class, sonOne.getId());  
        //초기화 되지 않음을 알 수 있다.
        Long id = sonOne1.getParentOne().getId();  
        Assertions.assertThat(id).isEqualTo(parentOne.getId());  
    }    
    
    @Test  
    @Transactional   
     void OneToOne_에서_비주인에서_Lazy로_조회(){  
        ParentOne parentOne = new ParentOne();  
        SonOne sonOne = new SonOne();  
        sonOne.setParentOne(parentOne);  
        em.persist(sonOne);  
        em.flush();  
        em.clear();  
        //  
  
        ParentOne parentOne1 = em.find(ParentOne.class, parentOne.getId());  
        //초기화 된다.
        Long id = parentOne1.getSonOne().getId();  
        Assertions.assertThat(id).isEqualTo(sonOne.getId());  
    }  
    @Test  
    @Transactional    
    void ManyToOne_에서_비주인에서_Lazy로_조회(){  
        ParentMany parentMany = new ParentMany();  
        SonMany sonOne = new SonMany();  
  
        sonOne.setParent(parentMany);  
        em.persist(sonOne);  
        em.flush();  
        em.clear();  
  
  
        ParentMany parent1 = em.find(ParentMany.class, parentMany.getId());  
        List<SonMany> sons = parent1.getSons();  
        // 프록시 로 바로 초기화 되지 않는다.  
        System.out.println("sons.getClass() = " + sons.getClass());  
        sons.size(); //초기화 된다!.  
  
    }  
  
    @Test  
    @Transactional    
    void ManyToOne_에서_주인에서_Lazy로_조회(){  
  
        ParentMany parentMany = new ParentMany();  
        SonMany sonOne = new SonMany();  
  
        sonOne.setParent(parentMany);  
        em.persist(sonOne);  
        em.flush();  
        em.clear();  

  
        SonMany son1 = em.find(SonMany.class, sonOne.getId());  
        ParentMany parent = son1.getParent();  
        //sons.getClass() = class com.example.entity.test.ParentMany$HibernateProxy$CbhHXYud  
        System.out.println("sons.getClass() = " + parent.getClass());  
        System.out.println(1);  
  
        parent.getId(); // id 만 조회하므로 초기화 되지 않는다.  
  
        System.out.println(2);  
  
        parent.getSons(); //초기화 된다!.  
  
  
    }  
  
}
```
- 정리하면 다음과 같다. DB 에 외래키가 존재하는 엔티티 (**연관관계 주인**) 은 `select * from Son` 과 같은 쿼리만으로 `parent` 엔티티의 식별자를 확인할 수 있다.
- 그래서 `em.getReference()` 로 프록시를 만들어 둠을 알 수 있다. 그렇기 때문에 `getId()` 를 호출 했을 땐 추가적으로 쿼리가 나가지 않는다.
- 반면에, **연관관계 주인이 아닌** 엔티티 `Parent` 에서 `Son` 엔티티에 접근하기 위해서는 `select * from Son s where s.parent_id=1`  로 쿼리를 따로 수행해보는 방법 말고는 불가능하다.

>JPA 도 결국은 DB 에 기반하여 동작하기 때문에 DB 기본 동작을 무시하고 동작할 수 없다. JPA 는 객체지향과 RDBMS 의 매퍼일 뿐이지 마법을 부리는게 아니다.

### 6.3.2 PersistentBag
- **컬렉션 래퍼** : 하이버네이트에서 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 **컬렉션 래퍼** 로 변경한다. `PersitentBag`
- 엔티티는 프록시 객체가 지연 로딩을 수행하게끔 해주고, 컬렉션은 **컬렉션 래퍼가** 이를 담당한다.
	- 위의 코드에서 `parent` 엔티티를 조회히 `sons.size()` 등으로 접근 했을 때에서야 실제 쿼리가 실행되어 `select * from sons where parent_id =1L` 의미를 갖는 **JPQL** 이 실행됨을 확인했다.
# 7. CascadeType ,  OrphanRemoval

## 7.1 CascadeType
- CascadeType은 부모 엔티티의 특정 작업을 자식 엔티티에도 전파하는 옵션이다.
	- CascadeType.ALL: 모든 Cascade 작업을 적용
	- **CascadeType.PERSIST**: 부모 엔티티 저장 시 자식 엔티티도 함께 저장
	- **CascadeType.REMOVE**: 부모 엔티티 삭제 시 자식 엔티티도 함께 삭제
	- **CascadeType.MERGE**: 부모 엔티티 병합 시 자식 엔티티도 함께 병합
	- **CascadeType.REFRESH**: 부모 엔티티 새로고침 시 자식 엔티티도 함께 새로고침
	- **CascadeType.DETACH**: 부모 엔티티 분리 시 자식 엔티티도 함께 분리
    

## 7.2 orphanRemoval
- orphanRemoval은 부모 엔티티와의 연관관계가 끊어진 자식 엔티티(고아 객체)를 자동으로 삭제하는 옵션
## 7.3 CascadeType.REMOVE vs orphanRemoval

1. **CascadeType.REMOVE**:
    - 부모 엔티티가 삭제될 때 자식 엔티티도 함께 삭제
    - 자식 엔티티와의 연관관계가 끊어져도 자식 엔티티는 삭제 x
        
2. **orphanRemoval = true**:
    - 부모 엔티티가 삭제될 때 자식 엔티티도 함께 삭제
    - 부모 엔티티와 자식 엔티티의 연관관계가 끊어지면 자식 엔티티가 자동으로 삭제

**CommentService.java**
```java

  
public Comment createComment_v2(Long userId, Long postId, String content) {  
    User user = userService.findById(userId);  
    Post post =postService.findById(postId);  
  
  
    Comment comment = Comment.init(user, post, content);  
    post.addComment(comment);  
    return comment;  
}
  
public void delete(Long postId, Long commentId) {  
    Post post =postService.findById(postId);  
    post.removeCommentById(commentId);  
}

```

**Post.java**
```java
...

@OneToMany(fetch =  FetchType.LAZY,mappedBy = "post", cascade =  CascadeType.ALL,orphanRemoval = true)  
@JsonIgnore  
private List<Comment> comments = new ArrayList<>();
...

public void addComment(Comment  comment) {  
  
    this.comments.add(comment);  
}  
  
  
public void removeCommentById(Long commentId) {  
    Comment comment=null;  
    for ( var c : comments){  
        if(c.getId()==commentId)  
            comment = c;  
    }    if(comment !=null){  
        this.comments.remove(comment);  
  
    }  
}

```


> orphanRemoval  = false, CascadeType.ALL 로 지정해놓고도 실행해보면  댓글이 삭제 되지 않음을 알 수있다. 이는 연관관계 해제시 자식 엔티티를 삭제하는 것은 `orphanRemoval` 이고 `CascadeType.REMOVE`은 부모 엔티티가 삭제될 때 자식도 삭제되게 해주는 옵션임을 알 수 있다.
> (oraphanRemoval = true 인 경우에도 부모 엔티티 삭제시에 자식엔티티가 삭제되는데, 부모엔티티의 삭제도 연관관계가 끊긴것을 의미하기 때문이다.)


# 8.  JPQL

- 테이블이 아닌 **객체 대상으로 검색**하는 객체지향 쿼리이다.
- SQL을 추상화해 특정 데이터베이스에 의존하지 않는다.
- 요구사항이 변경되어 게시글들을 조회할 때  `status` 뿐만 아니라  `createdAt` 을 사용해 특정 기간동안 만들어진 게시글들을 확인 하고싶다.
- 이럴때 **spring data jpa** 를 사용해 다음과 같이 구현할 수 있다.  
 ```java
public interface PostRepository  extends JpaRepository<Post,Long> {  
    Boolean existsByIdAndUser(Long id, User user);  
  
    Page<Post> findAllByStatus(PostStatus status, Pageable pageable);  
  
    Page<Post> findByStatusAndCreatedAtGreaterThanEqualAndCreatedAtLessThanEqual( Pageable pageable,PostStatus status, LocalDateTime startDate, LocalDateTime endDate);  
  
}
```
- 보다시피 **spring data jpa** 를 사용해 코드를 작성하면 함수명이 너무 길어진다.

- 또, 요구사항이 추가되어서 `userId` 값또한 `where` 조건절에 넣어주고싶다 . 그러면 다음과 같이 함수를 추가해주어야 한다.

```java
Page<Post> findByStatusAndCreatedAtGreaterThanEqualAndCreatedAtLessThanEqualAndUserId( Pageable pageable,PostStatus status, LocalDateTime startDate, LocalDateTime endDate,Long userId);
```


>몰론, `findByStatusAndCreatedAtGreaterThanEqualAndCreatedAtLessThanEqual()` 는 `findaAll()` 함수를  사용한후에 서비스 레이어에서 필터를 사용해 구현할 수 있다. 그러나 어플리케이션 에서 불필요하게 메모리를 차지하게 된다. 처음부터 DB 에서 알맞게 조회 했으면 해결 됐을 문제이다.

## 8.1 n+1 문제
---
- `post` 엔티티를 조회할 때 연관된 `comments` 들 또한 함께 조회 해주고 싶다.
- `comments` 에 `@JsonIgnore` 을 달아주었던 코드를 잠시 해제 해주자
### FetchType.Lazy

```java
    @Transactional  
    public List<Post> findAllWithCommentsLazy(Pageable pageable){  
//        User user = userService.findById(userId);  
        Page<Post> allByStatus = postRepository.findAll(pageable);  
  
        allByStatus.getContent().stream().forEach((p)-> System.out.println(p.getComments().size()));  
        return allByStatus.getContent();  
    }
```

- `Post` 엔티티의 `comments`를 지연로딩으로 설정해주자
- 이후 postman 을 통해 해당 기능을 수행하게 되면 모든 `Post` 결과마다 아래의 쿼리가 반복해서 실행된다.
```sql
 select
        c1_0.post_id,
        c1_0.commnet_id,
        c1_0.content,
        c1_0.created_at,
        c1_0.updated_at,
        c1_0.user_id 
    from
        comment c1_0 
    where
        c1_0.post_id=?
```
- 이렇게 1개의 `Post` 를 조회하는 쿼리 , `Post` 결과 값 n 개 `comment` 에 대해 쿼리를 수행한다 해서 **N+1** 문제라고 부른다.
### FetchType.Eager

```java
@Transactional  
public List<Post> findAllWithCommentsEager(Pageable pageable){  
//        User user = userService.findById(userId);  
	Page<Post> allByStatus = postRepository.findAll(pageable);  

//        allByStatus.getContent().stream().forEach((p)-> System.out.println(p.getComments().size()));  
	return allByStatus.getContent();  
}
```
- 즉시 로딩으로 설정 해도 마찬가지이다. n 개의 `Post` 결과들에 대해 추가적으로 `comments` 를 조회하는 쿼리를 수행하게 된다.

```java
@Transactional  
public Post findByIdWithCommentsEager(Pageable pageable){  
//        User user = userService.findById(userId);  
        Post post = postRepository.findById(1L).orElseThrow(EntityNotFoundException::new );  
  
//        allByStatus.getContent().stream().forEach((p)-> System.out.println(p.getComments().size()));  
        return post;  
    }
```
- 참고로 위와같이 `findById` 와 즉시 로딩인 경우에는 처음부터 `left join` 을 실행한다.
```sql
select
	...
from
	post p1_0 
left join 
	comment c1_0 
		on p1_0.post_id=c1_0.post_id 
where
	p1_0.post_id=?
```

## 8.2 Fetch Join 
---

**data jpa 만 사용했을 경우 문제점**
- **spring data jpa**  에 추가되는 함수들 이름이 너무 길어 이해하기 힘들다.
- 지연 로딩이나 즉시로딩 시에 연관 엔티티의 정보가 필요할 때 n+1 문제가 발생하는데, 이름 해결해 주기 어렵다.

#### 8.2.1 n+1 문제 Fetch Join 으로 해결
```java
@Query("SELECT p FROM Post p LEFT JOIN FETCH p.comments")  
List<Post> findAllWithCommentsFetchJoin();
```

- `Fetch Join` 을 사용하면 글로벌 로딩 전략과 무관하게 연관 엔티티를 동시에 조회할 수 있다. 즉, n+1 번 실행되던 문제가 해결된다.

####  8.2.2 너무 긴 함수명
```java
@Query(value="select p from Post p where p.status=:status and p.createdAt >= :startDate and p.createdAt <=:endDate",  
countQuery = "select count(p) from Post p"  
)  
Page<Post> findCustom( Pageable pageable,PostStatus status, LocalDateTime startDate, LocalDateTime endDate);
```

#### 8.2.3 Fetch Join 의 한계점

**Fetch Join 은 둘 이상의 컬렉션을 동시에 조회할 수 없다**
```java
List<User> users = em.createQuery( "SELECT u FROM User u " + "JOIN FETCH u.posts " + "JOIN FETCH u.comments", User.class) .getResultList();
```
- `MultipleBagFetchException` 이 발생한다. 따라서 두 조인 쿼리를 나눠서 각각 실행해주어야 한다.


 **fetch join의 대상은 on, where 등에서 필터링 조건으로 사용하면 안된다**
```java
List<User> users =  
        em.createQuery("SELECT u FROM User u "                
        + "LEFT JOIN FETCH u.posts p on p.status=:status " , User.class)  
                .setParameter("status", PostStatus.APPROVED)  
                .getResultList();
```

```sql
select * from user u left join post p on u.user_id=p.user_id and p.status ='APPROVED'
```

- 위와 같이 전체 유저를 조회 하나, 승인된 포스트들을 조인을 통해 함께 조회 하고싶다. 실제로 많이 있을법한 유형의 쿼리이다. 
- 그러나 `jpql` 에서는 이를 허용하지 않는다. 영속성 컨텍스트와 DB 의 **일관성** 이 깨질 수 있기 때문이다.
- 페치 조인은 연관된 모든 엔티티가 조회한 객체 (User) 에 존재할 것이라 기대한다. 그러나 `on, where` 절에 연관 엔티티로 걸어주게 되면 그렇지 않다.

```java
@Test  
@Transactional  
public void 페치조인_where절_일관성_문제() {  
    User user =  
            em.createQuery("SELECT u FROM User u "                            + "LEFT JOIN FETCH u.posts p where p.status=:status and u.id=1L" , User.class)  
                    .setParameter("status", PostStatus.APPROVED)  
                    .getSingleResult();  
  
    Assertions.assertThat(em.contains(user)).isTrue();  
    user.addPost(Post.init(user,"업데이트!"));  
    Assertions.assertThat(user.getPosts().size()).isEqualTo(11);  
  
   
    em.flush();  
    em.clear();  
  
    User byId = userRepository.findById(user.getId()).orElseThrow();  
  
    byId.getPosts().stream().forEach(e -> System.out.println(e.getStatus()));  
  
}
```

| user_id | ... | post_id | ... | status   |
| ------- | --- | ------- | --- | -------- |
| 1L      |     | 1L      |     | approved |
| 1L      |     | 2L      |     | approved |
| ...     |     |         |     |          |
- `"SELECT u FROM User u LEFT JOIN FETCH u.posts p where p.status=:status and u.id=1L"` 의 결과는 위와 같을 것이다. 
- 따라서 영속성 컨텍스트에는 `User(1L`) 의 포스트가 일부만 영속성 컨텍스트에 존재하게된다.
- 이때 변경 감지가 일어나거나,  `em.merge` 로 엔티티의 수정이 일어날 경우 최악의 경우 지연 상태인 포스트들이 모두 삭제될 수도 있다. ( hibernate 버전이 오래된 경우)

## 8.3 JPQL 과 영속성 컨텍스트
---
-  **JPQL은 실행되기 전에 em.flush()를 호출한다.**
- DB 에 직접 조회하는 특성 때문에 영속성 컨텍스트와 불일치가 생길 수 있기 때문이다.

```java
@Test  
@Transactional  
void JPQL_영속화_테스트() {  
    /// user1,2,3,4 생성 및 영속화  
    User user1 = User.init("테스트1");  
    User user2 = User.init("테스트2");  
    User user3 = User.init("테스트3");  
    User user4 = User.init("테스트4");  
  
    em.persist(user1);  
    em.persist(user2);  
    em.persist(user3);  
    em.persist(user4);  
  
    //user 1~ 4는 아직 영속성 컨텍스트에만 존재한다.  
  
    // JPQL을 이용해서 4번째 insert된 user4의 이름을 'changed name by jpql'로 update
    // em.flush() 가 암묵적으로 호출된다. insert 쿼리 4건 실행  
    Query query = em.createQuery(  
            "UPDATE User u set u.name = 'changed name by querydsl' where u.id = 4"    );  
    query.executeUpdate();  
    }}
```
# 9. QueryDsl
-  `fetch join `  을 사용해 **n+1 문제**와 **함수명이 길어져 가독성**이 떨어지는 문제를 해결 했다.
- 그러나,  **spring data jpa**나 **jpql** 로는 **동적 쿼리**를 작성하기 어렵다.
- **동적 쿼리**는 `findByStatusAndCreatedAtGreaterThanEqualAndCreatedAtLessThanEqual` 과 같은 쿼리중 `where` 절의 일부가 동적으로 결정되는 상황을 의미한다. 예를들면 다음과 같다.

1. 게시글의 상태만 조건 절로 사용
2. 게시글의 상태와 생성 날짜를 조건절로 사용
3. 생성 날짜만 조건절로 사용
4. 게시글 작성 사원을 조건절로 사용
5. .....

- 위와 같이 모든 케이스 ( 2^n  개) 만큼 함수를 작성하거나, 혹은 **jpql** 을 동적으로 만들어주는 알고리즘을 직접 작성해야한다.
```java
private String buildJpql(Status status, LocalDate date) { 
	StringBuilder jpql = new StringBuilder("SELECT p FROM EntityName p ");
	boolean hasCondition = false; 
	if (status != null) { 
		jpql.append("WHERE p.status = :status "); 
		hasCondition = true; } 
	if (date != null) { 
		jpql.append(hasCondition ? "AND " : "WHERE ");
		jpql.append("p.date = :date "); } return jpql.toString(); }
}
```

- 이런 동적 쿼리를 `builder` 패턴을 제공해 함수를 호출 함으로 편하게 구현할 수 있게 해주는게 **QueryDsl** 이다.


**build.gradle**

``` groovy
//QueryDSL 추가  
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'  
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"  
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"  
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```


**QueryDslConfig.java**
```java

@Configuration  
@RequiredArgsConstructor  
public class QueryDslConfig {  
    private final EntityManager entityManager;  
    @Bean  
    public JPAQueryFactory jpaQueryFactory(){  
        return new JPAQueryFactory(entityManager);  
    }}
```

**PostRepositoryCustom.java, PostRepository.java**

```java

public interface PostRepositoryCustom {  
    Page<Post> findPostsByDynamicConditions(  
            PostStatus status  
            ,LocalDate startDate,LocalDate endDate,  
            Long userId,  
            Pageable pageable  
    );  
  
    List<PostRepositoryImpl.PostDto> findGroupByStatusCount(Long employeeId);  
}
```


```java
public interface PostRepository  extends JpaRepository<Post,Long>, PostRepositoryCustom
```
- 구현체를 자동으로 생성해주는 **spring data jpa**에 구현하고자 하는 함수를 정의한 인터페이스를 상속 받아는다.
- `PostRepository` 는 `PostRepositoryCustom` 에서 정의한 함수를 모두 포함 하게된다.
- `PostRepository`가 빈으로 등록될 때 `PostRepositoryCustom` 의 구현체를 찾아 서로 자동으로 연결해준다,


**PostRepositoryCustomImpl.java**

```java  
@RequiredArgsConstructor  
public class PostRepositoryCustomImpl implements  PostRepositoryCustom{  
  
    private final JPAQueryFactory queryFactory;  
  
    @Override  
    public Page<Post> findPostsByDynamicConditions(PostStatus status,LocalDate startDate,LocalDate endDate,Long userId, Pageable pageable) {  
        QPost post = QPost.post;  
  
        BooleanBuilder builder = new BooleanBuilder();  
        if (status != null) {  
            builder.and(post.status.eq(status));  
        }        if(startDate!=null){  
            builder.and(post.createdAt.goe(LocalDateTime.of(startDate, LocalTime.MIN)));  
        }        if(endDate!=null){  
            builder.and(post.createdAt.loe(LocalDateTime.of(endDate, LocalTime.MAX)));  
        }        if (userId != null) {  
            builder.and(post.user.id.eq(userId));  
        }  
        List<Post> content = queryFactory  
                .selectFrom(post)  
                .where(builder)  
                .orderBy(post.createdAt.desc())  
                .offset(pageable.getOffset())  
                .limit(pageable.getPageSize())  
                .fetch();  
  
  
        // Total count 쿼리  
        long total = queryFactory  
                .selectFrom(post)  
                .where(builder)  
                .fetch()  
                .size();  
  
  
  
        return new PageImpl<>(content, pageable, total);  
    }  
    @Override  
    public List<PostDto> findGroupByStatusCount(Long employeeId) {  
  
        QPost post = QPost.post;  
  
        return queryFactory  
                        .select(  
                                Projections.constructor(PostDto.class,  
                                        post.status,  
                                        post.count()  
                                )                        )
	                    .from(post)  
                        .groupBy(post.status,post.user)  
                        .having(post.user.id.eq(employeeId))  
                        .orderBy(post.count().asc())  
                        .fetch();  
    }  
    @AllArgsConstructor  
    @NoArgsConstructor    
    @Getter    
    public static class PostDto{  
        PostStatus status;  
        Long count;  
    }  
}
	
```
