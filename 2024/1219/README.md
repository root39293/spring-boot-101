### Spring Boot 학습 기록

---

#### Spring Boot의 구조와 계층 이해

Spring Boot를 학습하며 기존 MVC 패턴이 Spring Boot에서는 더 세분화된 구조로 확장되었다는 것을 알게 되었다. MVC 패턴에서의 **Model**은 Spring Boot에서 **Entity**와 **Repository**로 분리되었고, **Controller**는 **Service**와 함께 사용자의 요청을 처리하며 더 명확한 역할을 갖게 되었다. **View**는 현대적 프론트엔드 프레임워크(React, Vue 등)와 연계되어 완전히 독립적으로 동작한다.

예를 들어, 사용자를 조회하는 기능을 구현할 때, 계층별로 이렇게 역할이 나뉜다

1. **Entity**:
   ```java
   @Entity
   public class User {
       @Id
       @GeneratedValue(strategy = GenerationType.IDENTITY)
       private Long id;
       private String name;
       private String email;
   }
   ```
   - 데이터베이스 테이블과 매핑되는 클래스이다. 여기서 `@Entity`와 `@Id` 등의 어노테이션을 통해 이 클래스가 JPA 엔티티임을 선언한다.

2. **Repository**:
   ```java
   @Repository
   public interface UserRepository extends JpaRepository<User, Long> {
       User findByEmail(String email);
   }
   ```
   - 데이터를 저장하거나 조회하는 작업은 모두 Repository 계층에서 담당한다. `findByEmail(String email)` 메소드를 작성하면 JPA가 알아서 SQL 쿼리를 생성해주는 점이 매우 편리하다.

3. **Service**:
   ```java
   @Service
   public class UserService {
       private final UserRepository userRepository;

       public UserService(UserRepository userRepository) {
           this.userRepository = userRepository;
       }

       public User findUserByEmail(String email) {
           return userRepository.findByEmail(email);
       }
   }
   ```
   - 비즈니스 로직은 Service 계층에 작성하여 Controller와 데이터를 처리하는 부분을 분리했다. 예를 들어, 사용자를 찾기 전에 입력된 이메일의 유효성을 검사하는 로직도 이곳에 추가할 수 있다.

4. **Controller**:
   ```java
   @RestController
   @RequestMapping("/api/users")
   public class UserController {
       private final UserService userService;

       public UserController(UserService userService) {
           this.userService = userService;
       }

       @GetMapping("/{email}")
       public ResponseEntity<User> getUserByEmail(@PathVariable String email) {
           User user = userService.findUserByEmail(email);
           return ResponseEntity.ok(user);
       }
   }
   ```
   - 클라이언트가 요청을 보냈을 때, 요청을 받아 적절한 Service 계층 메소드를 호출하고 그 결과를 반환한다. `@RestController`와 `@GetMapping` 같은 어노테이션으로 HTTP 요청을 처리하는 부분이 간결하게 표현된다.

이 계층 구조는 각 부분의 역할이 명확히 나뉘어 있어 코드의 가독성과 유지보수가 매우 쉬워진다.

---

#### Spring Boot의 의존성 주입(Dependency Injection)

Spring Boot에서 의존성 주입(DI)은 객체 생성과 관리를 개발자가 아닌 프레임워크가 대신 해주는 기능이다. 이를 통해 객체 간의 의존성을 관리하고 코드의 재사용성과 테스트 용이성을 높인다.

예를 들어, 사용자를 관리하는 로직을 작성한다고 가정했을 때, 의존성 주입을 사용하지 않으면 다음과 같이 객체를 직접 생성해야 한다:
```java
public class UserController {
    private final UserService userService = new UserService(new UserRepository());
}
```
하지만 Spring Boot에서는 DI를 통해 객체 생성 과정을 프레임워크에 맡긴다
```java
@RestController
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```
이처럼 생성자 주입을 사용하면 코드가 더 간결해지고, 테스트할 때도 `Mock` 객체를 쉽게 주입할 수 있어 효율적이다.

---

#### JPA와 데이터 영속성

Spring Boot에서 JPA를 사용하면 객체 지향적으로 데이터를 다룰 수 있다. 예를 들어, SQL로 작성하면 다음과 같은 코드를
```sql
SELECT * FROM users WHERE email = 'test@example.com';
```
JPA에서는 이렇게 간단히 표현할 수 있다
```java
User user = userRepository.findByEmail("test@example.com");
```
특히 Spring Data JPA는 메소드 이름만으로도 쿼리를 자동 생성해주는 기능이 있어 편리하다. `findByEmail`, `findByNameAndAge` 같은 메소드를 작성하면 자동으로 SQL 쿼리가 생성된다.

만약 복잡한 쿼리가 필요하다면, `@Query` 어노테이션을 사용해 직접 작성할 수도 있다
```java
@Query("SELECT u FROM User u WHERE u.age > :age")
List<User> findOlderThan(@Param("age") int age);
```

---

#### Spring Boot와 프론트엔드의 연동

Spring Boot는 과거 JSP와 같은 서버 사이드 렌더링 방식을 벗어나 REST API 서버로 동작하며, 프론트엔드와 완전히 분리된 구조를 지원한다. 이를 통해 프론트엔드 개발자는 React, Next.js 같은 최신 프레임워크를 사용해 유연하게 화면을 구성할 수 있다.

예를 들어, Next.js와 Spring Boot를 조합하면 다음과 같은 방식으로 연동이 이루어진다

1. **Spring Boot**: REST API 제공
   ```java
   @RestController
   @RequestMapping("/api/users")
   public class UserController {
       @GetMapping
       public List<User> getUsers() {
           return userService.findAllUsers();
       }
   }
   ```

2. **Next.js**: API 호출 및 데이터 렌더링
   ```javascript
   import React, { useEffect, useState } from "react";

   function UserList() {
       const [users, setUsers] = useState([]);

       useEffect(() => {
           fetch("http://localhost:8080/api/users")
               .then((res) => res.json())
               .then((data) => setUsers(data));
       }, []);

       return (
           <div>
               {users.map((user) => (
                   <div key={user.id}>{user.name}</div>
               ))}
           </div>
       );
   }

   export default UserList;
   ```

이러한 방식은 백엔드와 프론트엔드가 각자의 역할에 집중할 수 있도록 해준다. Spring Boot는 데이터 처리와 비즈니스 로직에만 집중하고, Next.js는 사용자 인터페이스와 화면 렌더링에 집중하는 구조다.