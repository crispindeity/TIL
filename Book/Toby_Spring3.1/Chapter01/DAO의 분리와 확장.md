# DAO의 분리와 확장
## 초난감 DAO
- DAO: DB를 사용해 데이터를 조회하거나, 조작하는 기능을 전담하도록 만든 오브젝트
- 자바 빈: 두 가지의 관례를 따라 만들어진 오브젝트
    1. 디폴트 생성자: 자바 빈 은 파라미터가 없는 디폴트 생성자를 가지고 있어야 한다.(툴 또는 프레임워크 에서 리플렉션을 사용하여 오브젝트를 만들기 위해)
    2. 프로퍼티: 자바 빈 이 노출하는 이름을 가진 속성, 프로퍼티는 set 으로 시작하는 수정 메서드와 get 으로 시작하는 접근자 메서드를 사용해 수정 또는 조회가 가능하다.

### User
```java
public class User {

    String id;
    String name;
    String password;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
- id, name, password 프로퍼티를 가지고 있는 User 오브젝트
- get, set 이 붙은 메서드틀 통해 조회와 수정이 가능하다.
- 즉 위에 오브젝트가 앞에서 설명한 자바 빈 이라고 할 수 있다.

### UserDAO
```java
public class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/toby_spring"
        );

        PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values (?, ?, ?)"
        );
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();
        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/toby_spring"
        );

        PreparedStatement ps = c.prepareStatement(
                "select * from users where id = ?"
        );

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
- User 오브젝트를 DB 에 넣고 관리하는 DAO 오브젝트
- ~~여담으로 코드스쿼드에서 순수 JDBC를 사용한 프로젝트를 진행할때 작성했던 코드와 매우 유사하다. 조금은 반가웠다 ㅋ.ㅋ~~
- 위에 작성된 코드를 현업 개발자가 작성하였다면 쫒겨났을꺼라는 토비님의 말씀이 있었다.
- 위에 코드가 어떤점이 문제인지, 이를 개선하면 앞으로 어떤 측면에서 좋은점을 갖는지 이걸 고민하는게 스프링을 공부하는 방법이라 한다.

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao userDao = new UserDao();

    User user = new User();

    user.setId("crispin");
    user.setName("crispindeity");
    user.setPassword("molru");

    userDao.add(user);

    System.out.println("등록 성공 = " + user.getId());

    User user2 = userDao.get(user.getId());

    System.out.println("user2.getName() = " + user2.getName());
    System.out.println("user2.getPassword() = " + user2.getPassword());

    System.out.println("조회 성공 = " + user2.getId());
}
```
- 위에 작성한 UserDao 오브젝트가 정상적으로 동작하는지 확인해보는 테스트용 main 메서드
- 물론 우리가 생각했던대로 등록과 조회가 매우 잘 되는걸 알 수 있다.
- 하지만, 기능이 잘 된다 해서 좋은 코드는 아니라는것

## DAO의 분리

### 관심사의 분리
- 관심이 같은 것 끼리는 하나의 객체 안으로 또는 친한 객체로 모이게 하고, 관심이 다른것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리해야한다.
- 만약 관심사를 따로 분리하지 않는다면, 어떠한 요구사항에 있어 변경이 일어나게 되면 코드에 많은 변경이 발생하게 된다.

### UserDao 의 관심사항
1. DB 와 연결을 위한 커넥션을 어떻게 가져올까 라는 관심
    - 더 세부적으로 어떤 DB 를 사용하고, 어떤 드라이버를 사용하며, 어떤 로그인 정보를 쓰고, 그 커넥션은 어떻게 만들것인지 이렇게 더 세부적으로 관심사를 분리 시킬수도 있다.
2. 사용자 등록을 위해 DB 에 보낼 SQL 문장을 담을 statement 를 만들고 실행하는 관심
    - 어떤 SQL 을 사용할지와, 파라미터를 어떻게 바인딩 시킬지 이것도 다른 관심사로 분리가 가능하다.
3. 공유 리소스를 반환하는 관심

### 중복 코드의 메서드 추출
```java
public void add(User user) throws ClassNotFoundException, SQLException {
    Connection c = getConnection();
    ...
}

public void get(String id) throws ClassNotFoundException, SQLException {
    Connection c = getConnection();
    ...
}

private Connection getConnection() throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.cj.jdbc.Driver");
    Connection c = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/toby_spring"
    );
    return c;
}
```
- 커넥션을 가져오는 부분의 관심을 getConnection() 메서드로 분리하였다.
- 앞으로 DB 가 변경되어 커넥션을 변경해야 하면 다른 메서드는 건드릴 필요없이 getConnection() 메서드만 변경하면 되어, 수정이 매우 편리해졌다.

### 상속을 통한 UserDao 확장
- 각기 다른 형식으로 커넥션을 만들 필요가 발생하였을때, getConnection() 메서드만 변경하여 각기 다른 커넥션을 만들 수 있도록 상속을 통해 UserDao 를 확장시켜 보자.

```java
public abstract class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
        ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
        ...
    }

    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
```
- UserDao 를 추상클래스로 만들어 get() 과 add() 의 메서드는 기존의 로직대로 되어있고, 필요에따라 커넥션을 만드는 메서드만 따로 구현하여 사용할 수 있게 만든다.

```java
public class NUserDao extends UserDao {
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        // N DB 커넥션 생성코드
        ...
    }
}
```

```java
public class DUserDao extends UserDao {
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        // D DB 커넥션 생성코드
        ...
    }
}
```
- 위와 같이 두 가지의 다른 커넥션을 만들어야 할때 UserDao 객체 는 어떠한 수정도 필요없이, UserDao 를 상속받아서 구현만 하면 된다.
- 즉, 어떻게 데이터를 DB 에 등록하고, 등독되어 있는 데이터를 가져올것인가 에 대한 관심을 갖고있는 UserDao와 DB 연결 방법을 어떻게 할 것인가 에 대한 관심을 갖고있는 NUserDao, DUserDao 가 클래스 레벨로 구분이 되고 있다.
- 변경이 쉽다라는 수준을 넘어서 이제는 확장까지 손 쉽게 해결할 수 있는 단계가 되었다.

### 상속을 통한 UserDao 확장 단점
- 상속을 통해 관심이 다른 기능을 분리하였지만, 상속관계는 두 가지의 다른 관심사에 대해 긴밀한 결합을 허용한다.
- 서브 클래스의 경우 슈퍼 클래스의 기능을 그대로 사용이 가능하기 때문에 슈퍼 클래스의 변경이 있을 경우 모든 서브 클래스의 수정이 불가피 하다.
- 확장된 기능인 DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용 시킬 수 없다. UserDao 외에 DAO 클래스가 계속 만들어진다면 상속을 통해 만들어진 getConnection() 메서드의 구현 코드가 매 DAO 클래스 마다 중복돼서 나타날수 있다.

## DAO의 확장

### 클래스의 분리
- 상속을 활용하여 관심사를 분리하는 것이 아니라 그냥 클래스 자체를 나눠서 관심사를 분리시켜 보자.
```java
public class UserDao {

    private SimpleConnectionMaker simpleConnectionMaker;

    public UserDao(SimpleConnectionMaker simpleConnectionMaker) {
        this.simpleConnectionMaker = simpleConnectionMaker;
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.getConnection();
        ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = simpleConnectionMaker.getConnection();
        ...
    }
}
```
- 생성자 주입을 통해 simpleConnectionMaker 를 주입 받아 변수에 저장하여, 메소드에서 사용하면 된다.

```java
public class SimpleConnectionMaker {
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/toby_spring"
        );
        return c;
    }
}
```
- 상속을 할 필요가 없으니 메서드는 abstract 로 만들 필요도 없다.
- 기존의 코드가 많이 변경되었지만, 기능상의 변경은 전혀 없다.
- 관심사의 경우는 완벽하게 분리가 되었지만, 이전 처럼 여러 DB 커넥션을 생성하려 한다면 UserDao 의 수정 없이 되는 것이 아니라, UserDao 가 수정되게 된다.

### 인터페이스의 도입
- 클래스를 분리하면서, 위에 상황처럼 긴민한 관계를 끊어 주기 위해 중간에 추상적인 연결고리가 필요하다.
- 추상적인 연결고리를 만들기에 자바에서 제공하는 가장 유용한 도구가 인터페이스 이다.

```java
public interface ConnectionMaker {
    Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```
- ConnectionMaker 라는 이름의 인터페이스를 정의한다.
- 이 인터페이스를 사용하는 UserDao 입장에서는 어떤 클래스로 만들었는지 상관없이 ConnectionMaker 타입의 오브젝트라면 makeConnection() 메서드를 호출하기만 하면 Connection 타입의 오브젝트를 반환해줄것으로 기대할 수 있다.

```java
public class DConnectionMaker implements ConnectionMaker {
    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        // D DB 커넥션 생성코드
        ...
    }
}
```
- ConnectionMaker 를 구현한 DConnectionMaker 클래스
- 필요에 맞게 D DB 커넥션을 생성하도록 메서드를 구현만 해주면 된다.

```java
public class UserDao {
    
    private ConnectionMaker connectionMaker;

    public UserDao() {
        this.connectionMaker = new DConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
        ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
        ...
    }
}
```
- ConnectionMaker 를 받아서 그 안에 있는 makeConnection() 메서드를 사용하기 때문에 UserDao 의 입장에서는 어떤 클래스로 ConnectionMaker 가 구현되었는지 알 필요가 없다.
- 그러나 여기서 문제점이 하나 발생하는데, 초기에 생성자를 통해 DConnectionMaker 오브젝트를 사용하도록 결정하는 코드가 여전히 UserDao 에 남아 있다.
- 결국 다른 ConnectionMaker 로 변경을 하려할때 이 부분에서 UserDao 의 코드에 수정이 일어나게 된다.

### 관계설정 책임 분리
- UserDao 에서 생성자를 통해 DConnectionMaker 오브젝트를 사용하도록 결정하는 코드가 남아 있는 문제를 해결하기 위해 관계설정 책임을 분리시켜 보자.
- 현재로는 UserDao 의 생성자에서 관계를 설정하고 있어 그 책임이 UserDao 에 있다.
- 이 책임을 UserDao 가 아닌 UserDao 를 사용하는 쪽에 둔다면 어떻게 될까?
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

    public User get(String id) throws ClassNotFoundException, SQLException {
        Connection c = connectionMaker.makeConnection();
        ...
    }
}
```
- 책임을 분리하기 위해 기존에 생성자에서 사용할 오브젝트를 결정하는 것이 아니라 매개변수를 통해 사용할 오브젝트가 결정 되도록 변경하였다.

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
    ConnectionMaker connectionMaker = new DConnectionMaker();
    UserDao userDao = new UserDao(connectionMaker);
    ...
}
```
- UserDao 를 사용하는 main 메서드에서 이제 어떤 ConnectionMaker 를 사용할 것인지 설정을 하고 그걸 인자값으로 넘겨 UserDao 와 ConnectionMaker 간의 관계를 설정해준다.
- 이렇게 되면 UserDao 의 경우 어떤 클래스로 ConnectionMaker 를 구현하였는지 알 필요가 없게 된다.
- 이게 바로 다형성이 가지는 큰 힘이다.
- 앞으로 ConnectionMaker 를 구현하는 클래스가 변경되더라도 UserDao 의 코드는 전혀 수정할 필요없이, UserDao 를 사용하는 쪽에서 변경된 클래스와 관계를 설정해주기만 하면 된다.
- 다른 DAO 클래스에서도 ConnectionMaker 를 구현한 클래스를 그대로 가져다 사용이 가능하기 때문에 상속에 비해 확장에 있어 매우 유연하게 변경되었다.

----
## REFERENCE
[토비의 스프링 3.1 Vol. 1 스프링의 이해와 원리](http://www.yes24.com/Product/Goods/7516721)