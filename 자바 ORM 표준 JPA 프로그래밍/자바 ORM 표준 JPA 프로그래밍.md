# JPA 소개

## SQL을 직접 다룰때 발생하는 문제점

### JPA와 문제 해결
 * JPA를 사용하면 객체를 데이터베이스에 저장하고 관리할 때, 개발자가 직접 SQL을
 작성하는 것이 아니라 JPA가 제공하는 API를 사용하면 된다.

 ~~~java
 // 저장 기능
 jpa.persist(member);
 ~~~
> persist() 메소드는 객체를 데이터베이스에 저장한다.

~~~java
// 조회 기능
String memberId = "helloId";
Member member = jpa.find(Member.class, memberId); // 조회
jpa.persist(member);
~~~
> find() 메소드는 객체 하나를 데이터베이스에서 조회한다.

~~~java
// 수정 기능
Member member = jpa.find(Member.class, memberId);
member.setName("이름변경"); // 수정
~~~
> JPA는 별도의 수정 메소드를 제공하지 않는다.
대신에 객체를 조회해서 값을 변경만 하면 **트랜잭션을 커밋할 때**
데이터베이스에 적절한 UPDATE SQL이 전달된다.

~~~java
// 연관된 객체 조회
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam(); // 연관된 객체조회
~~~
> JPA는 연관된 객체를 사용하느 시점에 적절한 SELECT SQL을 실행한다.


## 패러다임의 불일치
- 객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다.
- 이것을 객체와 관계형 데이터베이스의 패러다임 불일치 문제라 한다.

### 상속
- 객체는 상속이라는 기능을 가지고 있지만 테이블은 상속이라는 기능이 없다.

#### JPA와 상속
- JPA는 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.

### 연관관계
- 객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 참조에 접근해서 연관된
객체를 조회한다.
- 반면에 테이블은 외래 키를 사용해서 다른 테이블과 연관관계를 가지고
조인을 사용해서 연관된 테이블을 조회한다.

#### 객체를 테이블에 맞추어 모델링
- 객체는 연관된 객체의 참조를 보관해야 다음처럼 참조를 통해 연관된 객체를 찾을 수 있다.
~~~java
Team team = member.getTeam();
~~~

#### 객체지향 모델링
- 객체는 참조를 통해서 관계를 맺는다.
- 객체지향 모델링을 사용하면 객체를 테이블에 저장하거나 조회하기가 쉽지 않다.
- Member 객체는 team필드로 연관관계를 맺고 MEMBER 테이블은 TEAM_ID 외래 키로
연관관계를 맺기 때문이다.
- 객체 모델은 외래 키가 필요 없고 단지 참조만 있으면 된다.
- 반면에 테이블은 참조가 필요 없고 외래 키만 있으면 된다.
- 결국, 개발자가 중간에서 변환 역할을 해야 한다.


#### JPA와 연관관계
* JPA는 연관관계와 관련된 패러다임의 불일치 문제를 해결해준다.
~~~java
member.setTeam(team);
jpa.persist(member);
~~~
* 객체를 조회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해준다.
~~~java
Member member = jpa.find(Member.class, memberId);
Team team = member.getTeam();
~~~

### 1.2.3 객체 그래프 탐색
* 객체에서 회원이 소속된 팀을 조회할 때는 다음처럼 참조를 사용해서 연관된 팀을
찾으면 되는데, 이것을 객체 그래프 탐색이라 한다.
~~~java
Team team = member.getTeam();
~~~
* SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지
정해진다.
* 이것은 객체지향 개발자에겐 너무 큰 제약이다.
* 비즈니스 로직에 따라 사용하는 객체 그래프가 다른데 언제 끊어질지 모를 객체
그래프를 함부로 탐색할 수는 없기 때문이다.

#### JPA와 객체 그래프 탐색
* JPA를 사용하면 객체 그래프를 마음껏 탐색할 수 있다.
~~~java
member.getOrder().getOrderItem()... //자유로운 객체 그래프 탐색
~~~
* JPA는 연관된 객체를 사용하는 시점에 적잘한 SELECT SQL을 실행한다.
* JPA를 사용하면 연관된 객체를 신뢰하고 마음껏 조회할 수 있다.
* 이 기능은 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룬다고 해서
지연 로딩이라 한다.
~~~java
Member member = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate(); // Order를 사용하는 시점에 SELECT ORDER SQL
~~~
* Member를 사용할 때마다 Order를 함께 사용하면, 이렇게 한 테이블씩 조회하는 것보다는
Member를 조회하는 시점에 SQL 조인을 사용해서 Member와 Order를 함께 조회하는 것이 효과적이다.
* JPA는 연관된 객체를 즉시 함께 조회할지 아니면 실제 사용되는 시점에 지연해서 조회할지를
간단한 설정으로 정의할 수 있다.

### 1.2.4 비교
* 데이터베이스는 기본 키의 값으로 각 로우를 구분한다.
* 반면에 객체는 동일성 비교와 동등성 비교라는 두 가지 비교 방법이 있다.

> 동일성 비교는 == 비교다. 객체 인스턴스의 주소 값을 비교한다.
> 동등성 비교는 euqals() 메소드를 사용해서 객체 내부의 값을 비교한다.

~~~java
String memberId = "1";
Member member1 = memberDAO.getMembers(memberId);
Member member2 = memberDAO.getMembers(memberId);

member1 == member2 // 다르다.
~~~

#### JPA와 비교
* JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다.
~~~java
String memberId = "100";
Member member1 = jps.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; //같다
~~~

### 1.2.5 정리
* 객체 모델과 관계형 데이터베이스 모델은 지향하는 패러다임이 서로 다르다.
* 이런 패러다임 차이를 극복하려고 갓발자님들이 너무 많은 시간과 코드를 소비한다.
* 이런 것들을 해결하기 위한 결과물이 JPA다.


## 1.3 JPA란 무엇인가?
* JPA는 자바 진영의 ORM 기술 표준이다.
* JPA는 애플리케이션과 JDBC 사이에서 동작한다.
* ORM(object-relational mapping)은 이름 그대로 객체와 관계형 데이터베이스를 매핑한다는 뜻이다.
* ORM 프레임워크는 객체와 테이블을 매핑해서 패러다임의 불일치 문제를 개발자 대신 해결해준다.
