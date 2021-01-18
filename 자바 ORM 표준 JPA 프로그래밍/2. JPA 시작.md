## 1. 인텔리제이 설치와 프로젝트 불러오기

인젤리제이 설치는 [이곳](https://www.jetbrains.com/ko-kr/idea/)에서 할 수 있다. 학생이면 **Ultimate **버전을 다운받을 수 있다.

인텔리제이를 실행하면 다음과 같은 화면을 볼 수 있다.

![image](https://images.velog.io/images/tigger/post/1c0032ad-89b1-45db-a6b2-708f2eeb5c4d/image.png)

밑에 `Get from Version Control`을 클릭해보자.

![image](https://images.velog.io/images/tigger/post/627fa0a7-45a7-4d22-b624-b5bf496d6cec/image.png)

해당 git주소를 입력한다음 clone을 누른다.

![image](https://images.velog.io/images/tigger/post/ae4ab3ba-5c19-42c2-bd32-d94682dc99ab/image.png)

이렇게 프로젝트를 받을 수 있다.

## 2. H2 데이터베이스 설치

[H2 Database Engine](https://www.h2database.com/html/main.html) 홈페이지에서 Platform-Independent Zip(Last Stable)을 다운받아서 설치하자.

실행시키는 방법은 폴더 안에 bin 폴더가 있다. bin 폴더안에 있는 jar 파일을 실행시키면 H2 웹 브라우저 화면을 볼 수 있다.

![image](https://images.velog.io/images/tigger/post/2bf64a2c-2dab-4761-9036-261e086b333a/image.png)

화면이 나왔으면 실행을 해보자.

![image](https://images.velog.io/images/tigger/post/9cb4bf25-5568-4df3-8a54-c6477a77c0e7/image.png)

책을 보고 따라 했지만 이런 오류가 나왔다.🤔 무엇이 문제일까? 간단히 말하면 해당 경로에 존재하지 않는 데이터베이스를 연결 시도해서 나타나는 오류이다. 그럼 데이터베이스를 어떻게 만들까? 한번 만들어보자. 우선 밑에 그림과 같이 설정하고 실행한다.

![image](https://images.velog.io/images/tigger/post/776dd1c1-3248-4ac4-8e76-f96e8c517361/image.png)

해당 경로에 `test.mv.db`라는 파일이 생성된다. 그럼 다시 밑에 그림과 같이 Server로 설정하고 실행한다.

![image](https://images.velog.io/images/tigger/post/2bf64a2c-2dab-4761-9036-261e086b333a/image.png)

정상적으로 실행된다. 다음과 같은 화면이 나오면 성공이다.

![image](https://images.velog.io/images/tigger/post/bdf052ab-f7f2-49e5-bba7-d486a8c0da22/image.png)

그럼 이제 테이블을 만들어보자. 밑에 그림처럼 SQL을 입력하고 실행하면 왼쪽에 MEMBER 테이블이 생성된 것을 볼 수 있다.

![image](https://images.velog.io/images/tigger/post/1ab78b2f-b589-488e-a017-feef1726048d/image.png)

## 3. 라이브러리와 프로젝트 구조

### Gradle과 사용 라이브러리 관리

라이브러리는 Gradle이 관리해주고 있다. Maven도 있지만 Gradle을 사용해서 실습하는 이유는 간단하고 가독성이 좋기 때문이다. (Maven의 단점을 보완한 툴)

build.gradle로 가보자.

```java
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-gradle-plugin', version: '5.4.26.Final'
    compile group: 'com.h2database', name: 'h2', version: '1.4.200'
}
```

`dependencies`에 사용할 라이브러리를 지정한다. 원하는 라이브러리는 [이곳](https://mvnrepository.com/)에서 검색하면 찾을 수 있다.

### 프로젝트 구조

프로젝트 구조는 다음과 같이 되어있다.

```
src/main
├ java
│	└ jpastudy/start
│		├ JpaMain.java (실행 클래스)
│		└ Member.java (회원 클래스)
├ resources
│	└ META-INF
│		└ persistence.xml (JPA 설정 정보)
build.gradle
```

## 4. 객체 매핑 시작

H2 데이터베이스에 MEMBER 테이블을 생성했다.

```SQL
CREATE TABLE MEMBER (
    ID VARCHAR(255) NOT NULL, --아이디(기본 키)
    NAME VARCHAR(255),        --이름
    AGE INTEGER NOT NULL,     --나이
    PRIMARY KEY (ID)
)
```

이제 애플리케이션에서 사용할 Member 클래스를 만들어보자.

```java
package jpastudy.start;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="MEMBER")
public class Member {

    @Id
    @Column(name = "ID")
    private String id;

    @Column(name = "NAME")
    private String username;

    // 매핑 정보가 없는 필드
    private Integer age;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

![image](https://images.velog.io/images/tigger/post/d788d34e-61c8-4213-ace0-5d7139848fb1/image.png)

@Entity, @Table, @Column은 매핑 정보다. JPA는 매핑 어노테이션을 분석해서 어떤 객체가 어떤 테이블과 관계가 있는지 알아낸다. Member 클래스로 예를 들어 보자.

**@Entity**

- 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다.

**@Table**

- 엔티티 클래스에 매핑할 테이블 정보를 알려준다. name 속성을 사용해서 Member 엔티티를 MEMBER 테이블에 매핑했다. 만약 어노테이션을 생략하면 클래스 이름으로 매핑한다.

**@Id**

- 엔티티 클래스의 필드를 테이블의 기본 키(Primary key)에 매핑한다. 예시처럼 @Id가 사용된 필드를 식별자 필드라고 한다.

**@Column**

- 필드를 컬럼에 매핑한다. `username` 필드의 경우 name 속성을 사용해서 MEMBER 테이블의 NAME 컬럼에 매핑했다.

**매핑 정보가 없는 필드**

- `age` 핃드는 매핑 어노테이션이 없다. 이렇게 생략하면 필드명을 사용해서 컬럼명을 매핑한다. 만약 대소문자를 구분한다면 `username`처럼 명시해야한다.

## 5. persistence.xml 설정

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- xmlns, version 명시 -->
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

    <!-- 영속성 유닛 등록 -->
    <persistence-unit name="jpastudy">
        <!-- 엔티티 클래스 지정 -->
        <class>jpastudy.start.Member</class>
        <properties>
            <!-- 필수 속성 -->
            <!-- JPA 표준 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            <!-- 하이버네이트 속성 -->
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.format_sql" value="true" />
            <property name="hibernate.use_sql_comments" value="true" />
            <property name="hibernate.id.new_generator_mappings" value="true" />

            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>

</persistence>
```

### 데이터베이스 방언

JPA는 특정 데이터베이스에 종속적이지 않은 기술이다. 따라서 다른 데이터베이스로 쉽게 교체할 수 있다. 그런데 데이터베이스마다 제공하는 SQL 문법과 함수가 조금씩 다르다.

**가변 문자 타입**

- MySQL - VARCHAR
- 오라클 - VARCHAR2

**문자열 자르는 함수**

- SQL 표준 - SUBSTRING()
- 오라클 - SUBSTR()

**페이징 처리**

- MySQL - LIMIT
- 오라클 - ROWNUM

이처럼 SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA에서는 방언(Dialect)라고 한다. 하지만 하이버네이트를 포함한 대부분의 JPA 구현체들은 다양한 데이터베이스 방언 클래스를 제공한다.

## 6. 애플리케이션 개발

**코드 구성**

- 엔티티 메니저 설정
- 트랜젝션 관리
- 비즈니스 로직

```java
package jpastudy.start;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaMain {

    public static void main(String[] args) {

        //엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpastudy");
        EntityManager em = emf.createEntityManager(); //엔티티 매니저 생성

        EntityTransaction tx = em.getTransaction(); //트랜잭션 기능 획득

        try {
            tx.begin(); //트랜잭션 시작
            logic(em);  //비즈니스 로직
            tx.commit();//트랜잭션 커밋

        } catch (Exception e) {
            e.printStackTrace();
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); //엔티티 매니저 종료
        }

        emf.close(); //엔티티 매니저 팩토리 종료
    }

    public static void logic(EntityManager em) {

        String id = "id1";
        Member member = new Member();
        member.setId(id);
        member.setUsername("지한");
        member.setAge(2);

        //등록
        em.persist(member);

        //수정
        member.setAge(20);

        //한 건 조회
        Member findMember = em.find(Member.class, id);
        System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

        //목록 조회
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
        System.out.println("members.size=" + members.size());

        //삭제
        em.remove(member);
    }
}
```

### 1. 엔티티 메니저 설정

![image](https://images.velog.io/images/tigger/post/e68b7ade-b063-432d-acb0-2204698436c9/image.png)

**엔티티 매니저 팩토리 생성**

- persistence.xml의 설정 정보를 사용해서 엔티티 매니저 팩토리 생성
- Persistence 클래스 사용(엔티티 매니저 팩토리를 생성해서 JPA를 사용할 수 있게 준비함 )

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpastudy");
```

- META-INF/persistence.xml에서 이름이 jpastudy인 영속성 유닛을 찾아서 엔티티 매니저 팩토리 생성

> persistence.xml의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성하므로 엔티티 매니저 팩토리를 생성하는 비용은 아주 크다. 따라서 **엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.**

**엔티티 매니저 생성**

```java
EntityManager em = emf.createEntityManager();
```

- JPA 대부분의 기능 제공
- 등록/수정/삭제/조회 가능
- 내부에 데이터소스(데이터베이스 커넥션)를 유지하면서 데이터베이스와 통신(개발자는 엔티티 매니저를 가상의 데이터베이스로 생각할 수 있음)

> 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안 된다. 

**종료**

```java
// 마지막으로 사용이 끝난 엔티티 매니저 종료(반드시)
em.close(); // 엔티티 매니저 종료

// 애플리케이션 종료할 때 엔티티 매니저 팩토리 종료
emf.close(); // 엔티티 매니저 팩토리 종료
```

### 2. 트랜젝션 관리

- JPA를 사용하면 상항 트랜젝션 안에서 데이터를 변경해야 한다(트랜젝션 없이 데이터를 변경하면 예외가 발생).
- 트랜젝션을 시작하려면 엔티티 매니저에서 트랜젝션 API를 받아와야 한다.

```java
EntityTransaction tx = em.getTransaction(); //트랜잭션 API

try {
    tx.begin(); //트랜잭션 시작
    logic(em);  //비즈니스 로직 실행
    tx.commit();//트랜잭션 커밋

} catch (Exception e) {
    e.printStackTrace();
    tx.rollback(); //예외 발생 시 트랜잭션 롤백
}
```

### 3. 비즈니스 로직

```java
public static void logic(EntityManager em) {

    String id = "id1";
    // 엔티티 생성
    Member member = new Member();
    member.setId(id);
    member.setUsername("지한");
    member.setAge(2);

    //등록
    em.persist(member);

    //수정
    member.setAge(20);

    //한 건 조회
    Member findMember = em.find(Member.class, id);
    System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

    //목록 조회
    List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
    System.out.println("members.size=" + members.size());

    //삭제
    em.remove(member);
}
```

로직을 보면 CRUD 작업이 엔티티 메니저(em)에 의해 진행되는 것을 볼 수 있다.

**등록**

```java
em.persist(member);
```

- 엔티티 매니저의 `persist()` 메소드에 저장할 엔티티를 인자로 넣어준다.
- JPA가 Member 엔티티의 매핑 정보를 분석해서 다음과 같은 SQL을 만들어 전달한다.

그림은 위에서 설정한 하이버네이트 설정(persistence.xml)으로 터미널에 띄울 수 있다.

![image](https://images.velog.io/images/tigger/post/d7af5f94-3eda-4e05-ab04-ad4479e7de3b/image.png)

**수정**

```java
member.setAge(20);
```

여기서 의문이 있다. 수정은 엔티티 매니저를 사용 안 하는 것일까? JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있다. 자세한 내용은 3장(영속성 관리)에서 다룬다.

![image](https://images.velog.io/images/tigger/post/49c10ff4-025c-45c2-b115-d7fb5d2101f2/image.png)

**한 건 조회**

```java
Member findMember = em.find(Member.class, id);
System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());
```

@Id로 테이블의 기본 키와 매핑한 식별자 값으로 엔티티 하나를 조회한다. 왜 이 로직은 실행될 때 쿼리문을 안 보낼까? 그것도 물론 3장(영속성 관리)에서 이야기한다.

**삭제**

```java
em.remove(member);
```

엔티티를 삭제하려면 엔티티 매니저의 `remove()` 메서드에 삭제하려는 엔티티를 인자로 넣어준다.

![image](https://images.velog.io/images/tigger/post/ea796488-d668-4739-9dc2-e8f3d579319d/image.png)

## 4. JPQL

위에서 목록 조회에 대한 설명이 없었을 것이다. 왜일까? 해당 예시는 JPQL을 사용하여 목록을 조회했기 때문이다. 해당 예시는 JPQL을 사용하여 하나 이상의 Member 목록을 조회한 로직이다.

```java
List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
    System.out.println("members.size=" + members.size());
```

그럼 JPQL(Java Persistence Query Language)이 무엇일까? JPQL은 SQL을 추상화한 객체 지향 쿼리 언어이다. 그리고 JPA는 JPQL을 사용한다.

왜 JPQL을 사용할까? 이제까지 예시(CRUD)를 보면 SQL 문을 사용하지 않았다. 하지만 문제는 검색 쿼리다. JPA는 엔티티 객체를 중심으로 개발하므로 검색할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다. 데이터베이스의 모든 정보를 불러온 다음 객체로 만들어 검색하는 건 현실성이 없다. 이러한 문제를 해결해 주는 쿼리 언어가 JPQL이다. 문법도 SQL과 유사하다. 

그러면 차이점은 무엇일까? 

**차이점**

- JPQL - 엔티티 객체를 대상으로 쿼리(클래스와 필드 대상)
- SQL - 데이터베이스 테이블을 대상으로 쿼리

JPQL을 사용하려면 `em.createQuery(JPQL, 반환 타입)` 메서드를 실행해서 쿼리 객체를 생성한 후 쿼리 객체의 `getResultList()` 메서드를 호출하면 된다.

예제에서 사용한 `select m from Member m` 이것이 JPQL이다. 여기서 Member는 테이블이 아닌 Member 객체다. 다음 그림과 같이 JPA는 JPQL을 분석하고 적절한 SQL을 만들어 데이터베이스를 조회한다.

![image](https://images.velog.io/images/tigger/post/2819f4b1-15a8-4590-b17d-86c05fae83fb/image.png)