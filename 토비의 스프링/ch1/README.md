# 1장. 오브젝트와 의존관계

## DAO

### 용어 정리

- DAO
  - DAO(Data Access Object)는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트
- JavaBean(자바 빈)
  - 비주얼 툴에서 조작 가능한 컴포넌트
  - 두 가지 조건이 필요
    - 디폴트 생성자 : 파라미터가 없는 디폴트 생성자가 필요
    - 프로퍼티 : 자바빈이 노출하는 이름을 가진 속성, 접근자(set/get)을 통해 수정 및 조회 가능
- 리팩토링
  - 기존의 코드를 외부의 동작방식에 변화 ㅇ벗이 내부 구조를 변경해서 재구성하는 작업/기술
  - 좋은 내부 설계로 인해 유지보수와 생산성, 품질이 높아집니다.
  - 마틴 하울러의 **리팩토링** 책 추천

### JDBC 작업의 순서

1. DB 연결을 위한 Connection을 가져온다
2. SQL을 담은 Statement를 만든다
3. 만들어진 Statement를 실행한다
4. 조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 Object에 옮겨준다.
5. 작업 중에 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫는다.
6. JDBC API가 만들어내는 예외를 잡아서 직접 처리하거나, 메소드에 throws를 선언해서 예외가 발생시 메소드 밖으로 던집니다.

## 코드를 통한 리팩토링

### 리팩토링 전 코드

오브젝트 클래스

```java
@Getter @Setter
public class User {
  String id;
  String name;
  String password;
}
```

DB를 관리할 수 있는 DAO 클래스

```java
public class UserDao {
  public void add(User user) throws ClassNotFoundException, SQLException {
    Class.forName("come.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost:springbook", "spring", "book");

    PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
    ps.setString(1, user.getId());
    ps.setString(2, user.getName());
    ps.setString(3, user.getPassword());

    ps.executeUpdate();

    ps.close();
    c.close();
  }

  public void get(String id) throws ClassNotFoundException, SQLException {
    Class.forName("come.mysql.jdbc.Driver");
    Connection c = DriverManager.getConnection("jdbc:mysql://localhost:springbook", "spring", "book");

    PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
    ps.setString(1, id);

    ResultSet rs = ps.executeQuery();
    rs.next();
    User user = new User();
    user.setId(rs.getString("id"));
    user.setName(rs.getString("name"));
    user.setPassword(rs.getString("password"));

    rs.close();
    ps.close();
    c.close();

    return user;
  }
}

```

테스트 클래스

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
  UserDao dao = new UserDao();

  User user = new User();
  user.setId("123");
  user.setName("myName");
  user.setPassword("myPassword");

  dao.add(user);

  System.out.println(user.getId() + " is created");

  User user2 = dao.get(user.getId());

  System.out.println(user2.getName());
  System.out.println(user2.getPassword());
  System.out.println(user2.getId() + "is validate");
}
```

### 해당 코드의 문제점.

관심사의 분리가 없음. -> **분리와 확장**이 필요 (관심사의 분리, Separation of Concerns)

객체 지향의 장점을 살리지 못했음

- 변화에 효과적으로 대처할 수 잇음
- 가상의 추상세계를 효과적으로 구성 가능
- 이를 자유롭고 편리하게 변경, 발전, 확장이 가능합니다.

### 디자인 패턴

소프트웨어 설계 시 특정 상황에서 자주 만나는 문제를 해결하기 위해 사용할 수 있는 재사용 가능한 솔루션

패턴의 내부 설계의 구조는 일반적으로 비슷한데, 대부분 두 구조로 정리된다.

- 클래스 상속
- 오브젝트 합성

패턴에서 가장 중요한 내용은 **각 패턴의 핵심이 담긴 목적이나 의도**이다.

#### 템플릿 메소드 패턴

**의도** : 상속을 통해서 슈퍼클래스의 기능을 확장하기 사용하는 가장 대표적인 방법

변하지 않는 기능은 슈퍼클래스에서 만들고, 자주 변경되며 확장할 기능은 서브클래스에서 만듭니다.

슈퍼 클래스에서 미리 추상 메소드나 오버라이드 가능한 메소드를 정의하고, 이를 활용한 템플릿 메소드를 만듭니다. 이를 일반적으로 훅(hook) 메소드라고 합니다.

#### 팩토리 메소드 변경

**의도** : 상속을 통해 기능을 확장하는 패턴

슈퍼클래스 코드에서 서브클래스에서 구현할 메소드를 호출해서 필요한 타입의 오브젝트를 가져와서 사용합니다.

### 리팩토링 전략

다음과 같은 솔루션을 사용했습니다.

- 반복적인 코드 제거
- 상속 사용 -> but, 상속이 가지는 2가지 문제가 존재
  - 다중 상속 불가, 상속 클래스는 생각보다 밀접함
- 따라서 **인터페이스 사용**
- 관계설정 책임의 분리
- 불필요한 의존 제거 -> 다형성을 이용
- 각종 원칙과 패턴을 적용
  - 해당 문제에서는 개방 폐쇄 원칙을 적용하였습니다.
- IoC 사용

> 원칙과 패턴

객체지향 설계 원칙(SOLID)는 크게 다음과 같이 5가지로 구성됩니다.

- SPR(The Single Responsibility Principle) : 단일 책임 원칙
- OCP(The Open Closed Principle) : 개방 폐쇄 원칙
- LSP(The Liskov Substitution Principle) : 리스코프 치환 원칙
- ISP(The Interface Segregation Principle) : 인터페이스 분리 원칙
- DIP(The Dependency Inversion Principle) : 의존관계 역전 원칙

> 높은 응집도와 낮은 결합도 (High coherence and low coupling)

개방 폐쇄 원칙(OCP)는 **높은 응집도와 낮은 결합도**로 설명가능합니다.

- 높은 응집도
  - 변화가 일어날 때 해당 모듈에서 변하는 부분이 큽니다.
  - 응집도가 적으면 모듈의 일부분만 변하기 때문에 이를 확인해야하는 부담이 필요합니다.
- 낮은 결합도
  - 책임과 관심사가 다른 오프젝트 또는 모듈과는 낮은 결합도(느슨한 연결)을 유지합니다.

### 리팩토링 한 코드

![image](https://user-images.githubusercontent.com/42582516/108572122-f3d1eb80-7354-11eb-982b-e1c7dd558eb3.png)

다음과 같이 설계됩니다.

DaoFactory

```java
public class DaoFactory {
  public UserDao userDao() {
    return new UserDao(connectionMaker());
  }

  public AccountDao accountDao() {
    return new AccountDao(connectionMaker());
  }

  public MessageDao messageDao() {
    return new MessageDao(connectionMaker());
  }

  public ConnectionMaker connectionMaker() {
    return new DConnectionMaker();
  }
}
```

ConnectionMaker Interface

```java
public interface ConnectionMaker {
  public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

UserDao

```java
public class UserDao {
  private ConnectionMaker connectionMaker;

  public UserDao(ConnectionMaker connectionMaker) {
    this.connectionMaker = connectionMaker;
  }

  public void add(User user) throws ClassNotFoundException, SQLException {
    Connection c = connectionMaker.makeConnection();
    ...
  }

  public void get(String id) throws ClassNotFoundException, SQLException {
    Connection c = connectionMaker.makeConnection();
    ...
  }
}
```

Test

```java
public class UserDaoTest {
  public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao dao = new DaoFactory().userDao();

  }
}
```

## IoC

IoC는 제어의 역전(Inversion of Control)의 약자입니다.

### Factory

객체의 생성 방법을 결정하고, 만들어진 오브젝트를 돌려줍니다.

추상 팩토리 패턴이나 팩토리 메소드 패턴에서 이야기하는 팩토리와는 다릅니다.
