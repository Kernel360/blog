layout: post  
title: "JDBC, Hibernate, JPA 그리고 ORM"
author: "천승준"
categories: "기술블로그"
banner:
  image: 
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  tags: [`JDBC`, `JPA`, `ORM`]


# JDBC
- 자바 언어로 데이터베이스 프로그래밍을 하기위한 라이브러리
- DB에 직접 SQL을 실행하는 저수준 API
- jdbc를 매번 로딩해줘야하고 예외처리를 너무 많이 설정 해주어야함 -> JPA가 나옴

```
import java.sql.*;

public class JDBCDemo {
    public static void main(String[] args) {
        try {
            Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "user", "password");
            PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE email = ?");
            stmt.setString(1, "test@example.com");
            ResultSet rs = stmt.executeQuery();
            
            while (rs.next()) {
                System.out.println("User Name: " + rs.getString("name"));
            }

            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}


```


# JPA

- JPA란 JAVA ORM(Object Relational Mapping) 기술에 대한 인터페이스
  - ORM - 객체와 데이터베이스의 관계를 맵핑하는방법
- SQL을 직접 작성하지 않고, 엔티티 객체를 사용해 데이터 조작 가능
- JPA 자체는 인터페이스이므로, 직접 사용할 수 없음

EX)
```
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}

```

```
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User getUserByEmail(String email) {
        return userRepository.findByEmail(email);  // SQL 없이 데이터 조회
    }
}
```

# Hibernate
- JPA의 인터페이스를 구현한 라이브러리
- 내부적으로 JDBC를 사용하지만 자동으로 SQL을 생성하고 관리

```
Session session = HibernateUtil.getSessionFactory().openSession();
session.beginTransaction();

User user = session.get(User.class, 1);  // 자동으로 SQL 실행됨

session.getTransaction().commit();
session.close();

```

## 흐름
JPA ( 인터페이스 ) -> Hibernate ( 구현체 ) -> JDBC ( SQL 실행 )-> DB

- JPA는 Hibernate 같은 구현체를 통해 동작하고, Hibernate는 내부적으로 JDBC를 사용해서 SQL을 실행



# ORM (Object-Relational Mapping)
- ORM은 객체와 관계형 데이터 베이스 간의 매핑을 자동화 하는 기술
- SQL을 직접 작성하지 않고 객체(Entity)를 사용해 데이터베이스의 테이블을 조작할 수 있도록 하는 기술을 의미
- OOP(객체 지향 프로그래밍) 방식으로 데이터를 다룰 수 있으며 SQL을 직접 작성할 필요가 없어진다


### JPA (ORM 방식) 예시
```
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    private String name;
}

```

```
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByEmail(String email);
}

```

```
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User getUserByEmail(String email) {
        return userRepository.findByEmail(email);  // SQL 없이 데이터 조회
    }
}
```

### ORM의 장점
1. SQL 작성 필요없음
2. 객체 지향적인 개발 가능
3. DB 변경에 유연함
4. 생산성 증가 -> 비즈니스 로직에 집중 할 수 있음
5. 트랜잭션 관리 & 캐싱 -> Hibernate 같은 ORM 프레임워크는 자동으로 트랜잭션을 관리하고 성능 최적화를 제공

### ORM의 종류
#### 1. JPA (Java Persistence API)
- Java에서 ORM을 위한 표준 인터페이스.
- Hibernate, EclipseLink, OpenJPA 같은 구현체가 있음.

#### 2. Hibernate
- JPA의 대표적인 구현체로, JPA보다 더 많은 기능을 제공.
- 캐싱, 성능 최적화 기능 내장.

#### 3. MyBatis
- 완전한 ORM은 아니지만, XML 기반으로 SQL을 매핑하는 방식.
- 직접 SQL을 작성하지만, JDBC보다 간단하게 DB 매핑 가능.

### 그럼 비관계형 데이터 베이스랑은 사용할 수 없어?

#### -> ORM은 기본적으로 관계형 DB(RDBMS) 전용

- ORM은 SQL 기반의 관계형 데이터베이스(MySQL, PostgreSQL, Oracle, MariaDB 등)와 객체를 매핑하기 위해 설계됨.
- SQL을 자동 생성하고 테이블 구조를 객체(Entity)로 매핑하는 방식이기 때문에 MongoDB, Redis 같은 NoSQL과는 원래 맞지 않음.
- NoSQL은 테이블(스키마)이 없거나, 데이터 모델이 SQL과 다르기 때문에 기존 ORM 방식으로는 적용하기 어려움.

#### -> 일부 NoSQL 데이터베이스에서는 ORM과 유사한 기능을 제공하는 프레임워크가 있음
- 이를 ODM (Object-Document Mapping) 또는 ORM-like 프레임워크라고 부름.
- MongoDB - Mongoose (Node.js), Hibernate OGM ( Java ) / Redis - Spring Data Redis (Java) 같은 것이 대표적인 예 이다

# Spring Data JPA
- Hibernate 외에 어떠한 라이브러리를 써도 반복되는 작업의 발생때문에 이를 편리하게 하고 transaction 관리도 spring 에서 관리해주는 형태

```
    @Transactional
    public User save(User user) {
        return userRepository.save(user);
    }
    
    @Transactional
    @Override
    public <S extends T> S save(S entity) {
        
        Assert.notNull(entity, "Entity must no be null.");
        
        if(entityInformation.isNew(entity) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
    
```