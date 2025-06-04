---
{"dg-publish":true,"permalink":"//jpa/orphanremoval-cascade-type/"}
---

## 1. 엔티티 관계

####  User 엔티티 

```java
public class User {  
  
    @Id  
    @Column(name ="user_id")  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  

    private String name;  
  
    @OneToMany( mappedBy = "user", cascade = CascadeType.ALL,orphanRemoval = true)  
    @Builder.Default  
    private List<Post> posts= new ArrayList<>();  
  
    @OneToMany( mappedBy = "user", cascade = CascadeType.PERSIST ,orphanRemoval = true)  
    @Builder.Default  
    private List<Comment> comments= new ArrayList<>();  
  
  
    public void addPost(Post post){  
        this.posts.add(post);  
        post.setUser(this);  
    }  
    public void addComment(Comment comment){  
        this.comments.add(comment);  
        comment.setUser(this);  
    }}
```
- Post 와 1:N 양방향 매핑 되어 있으며 post 와 같은 생명주기를 갖는다.
- comment  와 1:N  양방향 매핑 되어 있으며 **CascadeType.PERSIT** 로 설정했다.

```java
public class Post {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "post_id", nullable = false)  
    private Long id;  
  
    @ManyToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "user_id" ,nullable = false)  
    private User user;  
  
    private String content;  
  
    @Enumerated(value = EnumType.STRING)  
    PostStatus status;  
  
    @OneToMany(mappedBy = "post" ,cascade = CascadeType.PERSIST, orphanRemoval = true)  
    @Builder.Default  
    private List<Comment> comments = new ArrayList<>();  
  
    public static Post init(String content){  
        return Post.builder()  
                .content(content)  
                .status(PostStatus.PENDING)  
                .build();  
    }  
  
    public void addComment(Comment comment){  
        this.comments.add(comment);  
        comment.setPost(this);  
    }  

}
```
- Comment 와 1:N 관계를 가지며 User 엔티티와 동일하다.


```java
public class Comment {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "id", nullable = false)  
    private Long id;  
  
  
    private String content;  
  
  
    @ManyToOne  
    @JoinColumn(name = "post_id")  
    private Post post;  
  
  
    @ManyToOne  
    @JoinColumn(name="user_id")  
    private User user;
```


## 2. 테스트 코드 실행 결과

#### orpahnRemoval 과 nullable  값에 따른 결과

```java
    @Test  
    void 유저_포스트_댓글_동시_삭제() {  
        User user = userFactory.create();  
        Post post = postFactory.create();  
        Comment comment = commentFactory.create();  
  
        user.addPost(post);  
        user.addComment(comment);  
        post.addComment(comment);  
  
        em.persist(user);  
        em.flush();  
        em.clear();  
  
        TypedQuery<User> findByUserId = em.createQuery("select u from User u where u.id=:userId",User.class)  
                .setParameter("userId", user.getId());  
        List<User> userList = findByUserId.getResultList();  
  
        em.remove(userList.get(0));  
        em.flush();
  
    }}
```

**1. orphanRemoval =true , comment nullable=true**
- comment 에 delete 쿼리 수행.


**2. orphanRemoval =false, comment nullable=true**
- commnet 로 인하여 외래키 제약조건에 위배돼 에러 발생


**3. orphanRemoval = true,  comment nullable=false**
- comment 에 delete 쿼리 수행.


**4. orphanRemoval =false,  comment nullable=false**
- commnet 로 인하여 외래키 제약조건에 위배돼 에러 발생


> **orphanRemoval = true** 부모 엔티티와 연관이 끊겻을 때 자식 엔티티 또한 삭제하는 설정이므로 부모가 삭제된것도 연관이 끊긴 것이라 판단하고 삭제 쿼리를 수행한한다는 것을 알 수 있다.

>**orphanReomval = false** 외래키 제약 조건으로 부모 엔티티 삭제시에 에러가 발생하므로 삭제시 자식 요소들을 제거하는 코드를 추가로 작성해주어야 한다.


#### 외래키 제약 조건 발생 해결

**@preRemove 이벤트 리스너 등록**
```java
    @PreRemove  
    private void preRemove() {  
        for (Comment comment : comments) {  
            comment.setUser(null);  
        }  
   }
```
- 위와 같은 코드로 User, Post 가 삭제전에 event listener 로 commet 엔티티의 외래키 값을 명시적으로 null 로 수정해 외래키 제약조건에 위배되는 현상을 회피할 수 있다.

**ON DELETE SET NULL 제약 조건 추가**
```sql
ALTER TABLE comment DROP FOREIGN KEY FK_post_comment; 
ALTER TABLE comment ADD CONSTRAINT FK_post_comment FOREIGN KEY (post_id) REFERENCES post (post_id) ON DELETE SET NULL;
```

## 3. 어떻게 사용해야할까
- User 데이터가 삭제되더라도 Post 나 Commnet 엔티티의 백업을 남겨야 할 상황이 많다. 이를 Jpa 로 해결하는 방법은 다음과 같다.
#### SoftDelete
```java

@Entity  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@Table(name = "user")  
@Builder  
@AllArgsConstructor  
@ToString  
@Setter  
@Getter  
@SQLDelete(sql = "UPDATE user SET delete_at = NOW() WHERE user_id = ?")  
@SQLRestriction("delete_at is null")  
public class User {  
  
    @Id  
    @Column(name ="user_id")  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    private LocalDateTime deleteAt;  
  
    private String name;  
  
    @OneToMany( mappedBy = "user", cascade = CascadeType.ALL,orphanRemoval = true)  
    @Builder.Default  
    private List<Post> posts= new ArrayList<>();  
  
    @OneToMany( mappedBy = "user", cascade = CascadeType.ALL)  
    @Builder.Default  
    private List<Comment> comments= new ArrayList<>();  
  
  
    public void addPost(Post post){  
        this.posts.add(post);  
        post.setUser(this);  
    }  
    public void addComment(Comment comment){  
        this.comments.add(comment);  
        comment.setUser(this);  
    }}
```

- **SQLDelete** 로 논리적으로 엔티티가 삭제된것 처럼 표시
	- 데이터베이스 용량을 많이 잡아 먹는다는 단점이 존재한다.
	- 쿼리에 항상 where 절을 포함해 삭제 여부를 판단해야한다.
- **SQLRestriction** 을 설정하면 자동으로 where 절에 `delete_at is null` 을 포함해준다.
- **SQLRestriction**  를 적용하면 삭제된 데이터를 조회 하고 싶어도 ( admin 수준 )기본적으로 `delete_at is null` 이 붙으므로 `jpql`을 작성하든 `@Fileter, @FilterDef` 를 적용해야한다.




#### @PostUpdate, @PostUpdate 로 이력 테이블에 추가
- 엔티티가 처음 영속화 될 때나 update 될 때에 이력테이블을 변경해주면 별도의 테이블을 생성가능하다.
- 이력 테이블은 원래 엔티티의 pk 를 pk 를 공유하며 다른 엔티티와 관계는 id 값만 저장하도록 수정하였다.
```java
@Entity  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@AllArgsConstructor  
@Builder  
@Table(name = "comment")  
@ToString(exclude = {"post","user"})  
@Setter  
@Getter  
//@SQLDelete(sql = "UPDATE Comment SET delete_at = NOW() WHERE user_id = ?")  
//@SQLRestriction("delete_at is null")  
public class Comment {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name = "commnet_id", nullable = false)  
    private Long id;  
  
    private String content;  
  
  
    @ManyToOne  
    @JoinColumn(name = "post_id",nullable = true)  
    private Post post;  
  
  
    @ManyToOne  
    @JoinColumn(name="user_id",nullable = true)  
    private User user;  
  
    @OneToOne(cascade = {CascadeType.PERSIST,CascadeType.MERGE},mappedBy = "comment")  
    private CommentArchive commentArchive;  
  
    @PostPersist  
    private void prePost(){  
        this.commentArchive = new CommentArchive(this);  
    }  
    @PostUpdate  
    private void postUpdate(){  
        this.commentArchive=new CommentArchive(this);  
    }  
}
```

```java
@Entity  
@Table(name = "comment_archive")  
@Setter  
@Getter  
@NoArgsConstructor(access =  AccessLevel.PROTECTED)  
@ToString(exclude = "comment")  
public class CommentArchive {  
    @Id  
    @OneToOne    @JoinColumn(name="comment_id",foreignKey = @ForeignKey(ConstraintMode.NO_CONSTRAINT))  
    private Comment comment;  
    private String content;  
    private Long user_id;  
    private Long post_id;  
  
    public CommentArchive( Comment comment) {  
        this. comment= comment;  
        this.content = comment.getContent();  
        this.post_id= comment.getPost().getId();  
        this.user_id = comment.getUser().getId();  
    }}
```
- Comment ** 와 **CommentArchive** 를 1:1 관계로 두고 CommnetArchive 의 `@id` 를 식별자 관계로 사용하기 위해  아래와 같이 작성했다.
- 또 외래키 제약 조건 을 없애주어 Commnet 삭제시에 CommentArchive 로 인해 외래키 관련 에러가 나는 상황을 방지했다.
```java
    @Id  
    @OneToOne    @JoinColumn(name="comment_id",foreignKey = @ForeignKey(ConstraintMode.NO_CONSTRAINT))  
    private Comment comment;  
```

- Comment 가 처음 영속화 되거나 수정될 때 연관된 엔티티또한 변경되길 원하므로 `CascadeType` 을 다음과 같이 작성 했고 양방향 연관관계로 설정해 주었다.
	- `@OneToOne(cascade = {CascadeType.PERSIST,CascadeType.MERGE},mappedBy = "comment")`




#### @EventListener 사용
- `@preRemove`  에 `entityManger` 를 주입 받아 사용하면 도메인에 레포지토리 관련 코드가  추가되어 DB 와 상호작용 하게된다. 
- 그래서 `@EventListener` 를 엔티티에 등록하여 엔티티가 삭제전에 이력 데이터를 저장하는 방식으로 구현했다.

```java
@EntityListeners(ArchiveListener.class)  
public class Comment { }

@Component  
public class ArchiveListener {  
  
//    @Autowired  
//    private ApplicationContext applicationContext;  
  
    @Lazy  
    @Autowired    private  EntityManager em;  
  
    @PreRemove  
    public void preRemove(Comment comment){  
//        EntityManager em = applicationContext.getBean(EntityManager.class);  
        CommentArchive commentArchive = new CommentArchive(comment);  
        em.persist(commentArchive);  
  
        System.out.println("등록완료");  
  
    }  
}


```


- Spring Data Jpa는 application이 시작할 때 `EntityManagerFactory`를 먼저 bean으로 등록 한다.
- `EntityManagerFactory`를 Bean으로 등록할 때 `EntityListener`에 대해서 Bean으로 등록하는 작업이 존재 한다.
- 그래서 `EntityListener`에서 `EntityManagerFactory`를 사용하는 Bean이 존재하게 되면 문제가 발생한다.
- `@Lazy` 를 사용하면 처음에 proxy 가 주입된 이후에 처음 사용할 때 실제 초기화를 하기 때문에 EntityMangerFactory 가 생성된 이후에 Bean을 주입 받을 수 있다.




