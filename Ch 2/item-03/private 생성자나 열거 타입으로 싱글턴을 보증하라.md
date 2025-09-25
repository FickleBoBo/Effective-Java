
# private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴(Singleton)이란? 
인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 하지만 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트는 테스트하기 어려워질 수도 있다. 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱글턴이 아니라면싱글턴 인스턴스가 가짜(Mock) 구현으로 대체할 수 없기 떄문이다.

## 싱글턴을 만드는 방식
### 1. public static 멤버가 final 필드인 방식

```Java
package effectivejava.chapter2.item3.field;

// Singleton with public final field  (Page 17)
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```
private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 떄 딱 한 번만 호출된다. 따라서 인스턴스가 전체 시스템에서 하나뿐임이 보장되지만 리플렉션 API를 통해 private 생성자를 호출할 수 있다. 이러한 공격을 방어하기 위해 두번째 객체가 생성되려 할때 예외를 던지면 된다. 이 방법은 해당 클래스가 싱글턴임이 API에서 명백하게 들어난다는 점과 간결하게 작성할 수 있다는 장점이 있다.

**Reflection API란?**

원문 (출처: [Java Docs - AccessibleObject](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AccessibleObject.html))

> The AccessibleObject class is the base class for Field, Method and Constructor objects.  
> It provides the ability to flag a reflected object as suppressing default Java language access control checks when it is used.  
> The access checks--for public, default (package) access, protected, and private members--are performed when Fields, Methods or Constructors are used to set or get fields, to invoke methods, or to create and initialize new instances of classes, respectively.  
> Setting the accessible flag in a reflected object permits sophisticated applications with sufficient privilege, such as Java Object Serialization or other persistence mechanisms, to manipulate objects in a manner that would normally be prohibited.  
>  
> By default, a reflected object is not accessible.  

해석

AccessibleObject 클래스는 Field, Method, Constructor 객체의 기반 클래스다.  
이 클래스는 리플렉션으로 얻은 객체를 사용할 때, 기본적인 자바 언어의 접근 제어 검사(public, 기본(패키지), protected, private)를 생략하도록 플래그를 설정할 수 있는 기능을 제공한다.  
이러한 접근 검사는 각각 필드를 읽거나 쓰거나, 메서드를 호출하거나, 클래스의 새 인스턴스를 생성·초기화할 때 수행된다.  

리플렉션 객체에 accessible 플래그를 설정하면 Java 객체 직렬화와 같이 충분한 권한을 가진 고급 애플리케이션이나 다른 영속화 메커니즘이 원래는 허용되지 않는 방식으로 객체를 조작할 수 있게 된다.  

기본적으로 리플렉션으로 얻은 객체는 접근 가능하지 않다. 

### 2. 정적 팩터리 메서드 방식의 싱글턴


```Java
package effectivejava.chapter2.item3.staticfactory;

// Singleton with static factory (Page 17)
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();
    }
```
이 방법 역시 두 번째 인스턴스는 생성되지 않지만 리플렉션의 예외는 똑같이 적용된다. 이 방법의 장점은 API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점이다. 또한 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있고 정적 팩터리의 메서드 참조를 타입 캐스팅 하여 다양한 인스턴스의 공급자(supplier)로 사용할 수 있다는 점이다. 

하지만 두 가지 방법 모두 직렬화하려면 `readResolve()`를 통해 가짜 인스턴스가 생성되는 것을 막아야한다. 이러한 한계를 극복한 방식이 Enum 타입 방식이다.

### 3. 열거(Enum) 타입 방식의 싱글턴 
```Java
package effectivejava.chapter2.item3.enumtype;

// Enum singleton - the preferred approach (Page 18)
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    // This code would normally appear outside the class!
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```

Enum 타입의 싱글턴 방식으로 데이터베이스를 연결하는 DatabaseManager 클래스를 작성해보면 다음과 같다.
```Java
// 데이터베이스 연결 관리자 예시
public enum DatabaseManager {
    INSTANCE;
    
    private Connection connection;
    
    DatabaseManager() {
        // 데이터베이스 연결 초기화
        this.connection = createConnection();
    }
    
    public Connection getConnection() {
        return connection;
    }
    
    private Connection createConnection() {
        // 실제 DB 연결 로직
        return DriverManager.getConnection("jdbc:...");
    }
}

// 사용
DatabaseManager.INSTANCE.getConnection();
```
이렇게 사용하면 생성 비용이 큰 작업을 전역적으로 하나만 관리해야 할 때 매우 유용하게 성능을 높일 수 있다. 
