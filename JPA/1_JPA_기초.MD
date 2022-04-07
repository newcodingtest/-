해당 내용은 [자바 ORM 표준 JPA 프로그래밍](https://book.naver.com/bookdb/book_detail.nhn?bid=9252528)의 내용을 정리





# JPA

Dialect : db에 종속적이지 않게 db연결 인터페이스? 같은 느낌의 기술이다.

![JPA 시작](https://t1.daumcdn.net/cfile/tistory/2733045058307EB915)



하이버네이트의 전용 속성들

> hibernate_show_sql: 하이버네이트가 실행한 sql을 출력한다.

> hibernate.format_sql: 하이버네이트가 실행한 sql을 출력할 때 보기 쉽게 정렬한다.

> hibernate.use_sql_comments: 쿼리를 출력할 때 주석도 함께 출력한다.

> hibernate.id.new_generator_mappings: JPA 표준에 맟춘 새로운 키 생성 전략을 사용한다.



## 엔티티 매니저 설정

#### 엔티티 팩토리 매니저

- persistentce.xml 설정정보를 읽어 JPA를 동작시키기 위한 객체를 만들고

- JPA 구현체에 따라서 DB 커넥션 풀도 생성하여 생성 비용이 매우 크다.

- 때문에 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.

#### 엔티티 매니저

- JPA 기능 대부분은 엔티티 매니저가 제공한다.

- 엔티티 매니저를 통해 등록/수정/삭제/조회를 할 수 있다.

- 내부에 DB 커넥션 풀을 유지하면서 DB와 통신한다.

- DB 커넥션과 밀접한 관계가 있으므로 스레드간 공유 및 재사용 하면 안된다!





> 검색 쿼리 진행 시에 엔티티 객체를 대상으로 검색하려면 DB의 모든 데이터를 애플레케이션으로 불러와 엔티티 객체로 변경한 다음 검색해야 하는데 이는 불가능하다. 때문에 애플리케이션이 필요한 데이터만 DB에서 불러와 SQL을 사용해아한다.JPA는 이를 JPQL로 해결한다.



#### JPA는 SQL을 추상화한 JPQL(Java Persistence Query Language)라는 객체지향 쿼리를 제공한다.

- JPQL은 엔티티 객체를 대상으로 쿼리 한다.(클래스와 필드를 대상으로 쿼리)

- JPQL은 대소문자를 명확히 구분한다.

- SQL은 DB 테이블을 대상을 쿼리 한다.



#### 엔티티 매니저 팩토리 와 엔티티 매니저

엔티티 매니저 팩토리는 여러 스레드가 동시에 접근해도 안전하여 스레드 간 공유할수 있으나

엔티티 매니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하여 절대 공유해서는 안된다.

![3.1 엔티티 매니저 팩토리와 엔티티 매니저 · jpa](https://ultrakain.gitbooks.io/jpa/content/chapter3/images/JPA_3_1.png)

엔티티 매니저는 보통 트랜잭션을 시작할 때 커넥션을 획독한다.



## 영속성 컨텍스트

- 엔티티를 영구 저장하는 환경이라는 뜻이다.

엔티티 매니저로 엔티티를 저장하거나 조회하면 엔티티 매니저는 **영속성 컨텍스트**에 엔티티를 보관하고 관리한다.

**영속성 컨텍스트 **는 엔티티 매니저를 생성할 때 하나 만들어진다. 엔티티 매니저를 통해 영속성 컨텍스트에 접근 및 관리가 가능하다.



## 엔티티 생명주기

- 비영속

 	엔티티 객체를 생성했으나 아직 저장하지 않은 상태(영속성 컨텍스트나 DB와는 전혀 무관한 상태)

```
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```



- 영속

​	 엔티티 매니저를 통해서 엔티티를 영속성 컨텍스트에 저장한 상태

```
em.persist(member)
```



- 준영속

​	 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않는 상태

​	 특정 엔티티를 준영속 상태로 만들려면 em.detach 호출

​     영속성 컨텍스트를 닫거나 em.clear()를 호출하여 초기화해도  준영속 상태가 된다.

```
em.detach(member);
```

​	

- 삭제

​	 엔티티를 영속성 컨텍스트와 DB에서 삭제한 상태	

```
em.clear(member);
```

![엔티티 생명주기](http://image.slidesharecdn.com/ksug2015jpa3-150803103312-lva1-app6892/95/ksug2015-jpa3-jpa-11-638.jpg?cb=1438598300)



## 영속성 컨텍스트의 특징

- 영속성 컨텍스트와 식별자 값

  영속성 컨텍스트는 엔티티를 식별자 값(@Id)로 구분

  반드시 식별자 값이 있어야함(없으면 예외 발생)

- 영속성 컨텍스트와 DB저장

  JPA는 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 DB에 반영(Flush)

- 1차 캐시 보장

- 동일성 보장

- 트랜잭션 지원하는 쓰기 지연

- 변경 감지

- 지연 로딩

### 

#### 엔티티 조회

영속성 컨텍스트는 내부에 캐시를 가지고 있다 (=1차 캐시)

영속성 컨텍스트 내부에  Map이 있다.

**키**는 @Id로 매핑한 식별자 **값**은 엔티티 인스턴스

![JPA / 영속성 컨텍스트 / 1차 캐시 / 쓰기 지연](https://blog.kakaocdn.net/dn/blsBzW/btqFcN1dVRK/EYpjNc8R8tnJSpRtIC6Fj1/img.png)

```
//엔티티를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");

//1차 캐시에 저장됨
em.persist(member);

```



![영속성 컨텍스트(Persistence Context) 사용 시 이점 - Junhyunny&#39;s Devlogs](https://junhyunny.github.io/images/persistence-context-advantages-1.JPG)

```
//1차 캐시에서 조회
Member member = em.find(Member.class, "member1");
```



##### 조회 했는데 1차 캐시에 어떤 정보도 없는 경우

![JPA / 영속성 컨텍스트 / 1차 캐시 / 쓰기 지연](https://blog.kakaocdn.net/dn/bgRAP6/btqFbB1uCQG/6yRnxmE2SevKVNMWMsLAK1/img.png)

엔티티 매니저는 DB를 조회해서 엔티티를 생성후 - 1차 캐시에 저장후 - 영속 상태의 엔티티를 반환

#### 

#### 영속 엔티티의 동일성 보장

위의 사진을 참고하여 member1 하고 member2는 == 동일성 비교를 하면 같은 값이 나온다.

1차 캐시에 있는 같은 인스턴스이기 때문이다.

따라서 영속성 컨텍스트는 성능상 이점과 엔티티의 동일성을 보장





#### 엔티티 등록

![JPA - kimseokjin0324/Study Wiki](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FPoi51%2FbtqFaDfbl3g%2FLk3LB6966XcEG8nuNdyXC0%2Fimg.png)

- 엔티티 매니저는 트랜잭션을 커밋하기 직전가지 DB에 엔티티를 저장하지 않고 내부 쿼리 저장소에 쿼리문을 모아뒀다가 트랜잭션을 커밋할때 DB에 쿼리문을 보낸다. 이것을 **쓰기 지연** 이라고 한다.

- 트랜잭션을 커밋하면 엔티티 매니저는 우선 영속성 컨텍스트를 플러시 한다. 플러시(Flush)는 영속성 컨텍스트의 변경 내용을 DB에 동기화 하는 작업이다.(실제 순서는 동기화 후 - 실제DB 트랜잭션 커밋 실행)



#### 엔티티 수정

![img](https://media.vlpt.us/post-images/conatuseus/b5d57200-d0a0-11e9-90a8-3bdc8e61daef/image.png)

**변경감지** 를 통하여 엔티티 수정이 가능하다.

단순히 엔티티를 조회해서 데이터만 변경하면 끝난다.

```
transaction.begin();

//영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

//영속 엔티티 데이터 수정
memberA.setUsername("hi");
memberA.setAge(10);

transaction.commit();
```



##### JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 스냅샷을 떠둔다.

##### 그리고 플러시 시점에 스냅샷과 엔티티를 비교하여 변경된 엔티티를 찾는다



- 1.트랜잭션을 커밋하면 엔티티 매니저 내부에서 먼저 Flush가 호출된다.
- 2.엔티티와 스냅샷을 비교하여 변경된 엔티티 찾는다
- 3.변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보낸다
- 4.쓰기 지연 저장소의 SQL을 DB에 보낸다.
- 5.DB 트랜잭션을 커밋한다.



단 영속상태의 엔티티에만 적용된다.



**기본 수정 방법**은 내가 단 하나의 필드만 수정했어도 쿼리상으로는 전체가 다시 수정되어 반영되는 **정적 수정 쿼리**이다.



때문에 내가 수정한 필드만 수정 필드로 사용되는 **동적 수정쿼리** 도 있다.(저자는 30개 이상의 필드가 존재시에 사용을 권하고 있다.)

@DynamicUpdate 를 사용하자

```
@Entity
@org.hibernate.annotations.DynamicUpdate
@Table(name = "Member")
public class Member{...}
```



#### 엔티티 삭제

엔티티를 삭제하려면 먼저 삭제 대상 엔티티를 조회후 삭제해야한다.

```
Member memberA = em.find(Member.class, "memberA");
em.remove(memberA);
```

쓰기 지연 SQL 저장소에 삭제 쿼리가 등록된후

트랜잭션이 커밋해서 플러시를 호출하면 실제 DB에 삭제쿼리가 적용된다.

이후 영속성 컨텍스트에서 제거된다.



## 플러시(Flush)

**영속성 컨텍스트의 변경 내용을 DB에 반영한다.**

**"플러시라는 이름으로 인해 영속성 컨텍스트에 보관된 엔티티를 지운다고 생각하면 안된다!!.**

**영속성 컨텍스트의 변경 내용을 데이터 베이스에 동기화하는 것이 플러시다."**



영속성 컨텍스트를 플러시하는 방법은 3가지 이다.

1. em.flush() 를 직접 호출

   테스트나 다른 프레임워크와 JPA를 함께 사용할 때 제외하고 거의 사용하지 않는다.

2. 트랜잭션 커밋 시 플러시가 자동 호출

3. JPQL 쿼리 실행 시 플러시가 자동 호출

   식별자를 기준으로 조회하는 find() 메소드에는 플러시가 실행되지 않는다.



## 준영속

영속 상태의 엔티티가 영속성 컨텍스트에서 분리된 것

준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.



##### 1.em.detach(entity): 특정 엔티티만 준영속 상태로 전환

##### 2.em.clear(): 영속성 컨텍스트 완전히 초기화

##### 3.em.close(): 영속성 컨텍스트 종료



#### **1.em.detach(entity): 특정 엔티티만 준영속 상태로 전환**

```
//회원 엔티티 생성, 비영속 상태
Member member = new Member();
member.setId("memberA");
MEMBER.setUsername("회원A");

//회원 엔티티 영속 상태
em.persist(member);

//회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
em.detach(member);

transaction.commit(); //트랜잭션 커밋
```

![img](https://media.vlpt.us/images/hansoleee/post/e0dc41c9-e4b4-4caf-a9ad-bef14891d0cb/detach01.png)



![img](https://media.vlpt.us/images/hansoleee/post/07cf915a-2e7f-4380-94f4-44a4c9c7129d/detach02.png)



```
em.detach() 메소드를 호출하는 순간 1차 캐시부터 쓰기 지연 SQL 저장소까지 해당 엔티티를 관리하는 모든 정보가 삭제된다.
```

이를 영속성 컨텍스트가 관리하지 않는 준영속 상태라고 한다.

#### 2.em.clear(): 영속성 컨텍스트 완전히 초기화

```
//엔티티 조회, 영속 상태
Member member = em.find(Member.class, "memberA");

em.clear(); // 영속성 컨텍스트 초기화

//준영속 상태
member.setUsername("changeName");
```

![Untitled](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F0ee1f941-e8c2-4841-963a-b8de9f940f7c%2FUntitled.png?table=block&id=57e5b20e-288c-4876-a5bc-4690227fd825&spaceId=b453bd85-cb15-44b5-bf2e-580aeda8074e&width=2000&userId=80352c12-65a4-4562-9a36-2179ed0dfffb&cache=v2)

그림을 통한 memberA, memberB는 준영속 상태가 되었고

```
member.setUsername("changeName");
```

으로 이름을 바꿔도 DB에 적용되지 않는다.



#### 3.em.close(): 영속성 컨텍스트 종료

```
public void closeEntityManager(){

	EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
	
	EntityManager em = emf.createEntityManager();
	EntityTransaction transaction = em.getTransaction();
	
	transaction.begin();
	
	Member memberA = em.find(Member.class, "memberA");
	Member memberB = em.find(Member.class, "memberB");
	
	transaction.commit();
	
	em.close(); //영속성 컨텍스트 닫기(종료)

}
```

![close](https://user-images.githubusercontent.com/25560203/105717724-d2532f00-5f63-11eb-9454-c8decd9e7010.png)

```
영속성 컨텍스트가 종료되어 더는 memberA, memberB가 관리되지 않는다.
```

#영속성 컨텍스트가 종료되면서 준영속 상태가 되나 개발자가 직접 준영속 상태로 만드는 일은 드물다고 한다.



#### #준영속 상태의 특징

- 비영속에 가깝다

​		영속성 컨텍스트가 관리 하지 않으므로 1차캐시, 쓰기지연, 변경감지, 지연로딩을 포함한 영속성 컨텍스트가 제공하는 어떤 기능도 동작하지 않는다.

- 식별자 값을 갖고 있다

​		비영속 상태는 식별자 값이 없을수도 있지만 준영속 상태는 이미 한번 영속 상태였으므로 식별자 값을 반드시 가지고 있다.

- 지연 로딩을 할 수 없다.

​		지연로딩(Lazy Loading)은 실제 객체 대신 프록시 객체를 로딩해두고 해당 객체를 사용할때 영속성 컨텍스트를 통해 데이터를 불러오는 방법이다.

​		**하지만**

​		준영속 상태는 영속성 컨텍스트가 관리하지 않으므로 지연로딩시 문제가 발생한다



#### 병합: merge()

준영속 상태의 엔티티를 영속 상태로 변경할 때 사용된다.

```
member.setUsername("회원명변경"); //준영속 상태에서의 변경

mergeMember(member);
```

```
EntityManagerFactory emf= ...
.
.
.
static void mergeMember(Member member){
	
	EntityManager em1 = emf.createEntityManager();
	EntityTransaction transaction = em1.getTransaction();
	
	transaction.begin();

    Member memberMember = em1.merge(member);// 병합

	transaction.commit();
    .
    .
    .
    .
    .
    .
}
```



![img](https://blog.kakaocdn.net/dn/toLIq/btqBBViAn5G/bbF90GjZgP0VbzxkSl9er0/img.png)

1. merge()를 실행

2. 파라미터로 넘어온 준영속 엔티티 member의 식별자 값을 1차 캐시에서 조회

3. 1차 캐시에 엔티티가 없으면 DB에서 엔티티를 조회해서 1차 캐시에 저장

   *만약 DB에서도 조회를 못하면 새로운 엔티티를 생성하여 병합해야 한다.

4. 조회한 영속 엔티티(memberMember)에 준영속 엔티티(member) 값을 채워 넣는다.

5. memberMember를 반환한다.



> 여기서 생각해야 할 것은 member엔티티가 준영속 상태에서 영속상태로 변경된것이 아닌 mergeMember 라는 새로운 영속 상태의 엔티티가 반환것이다.



#### 비영속 병합

비영속 엔티티도 영속상태로 만들수 있다

```
Member member = new Member();
Member newMember = em.merge(member); //비영속 병합
tx.commit();
```

