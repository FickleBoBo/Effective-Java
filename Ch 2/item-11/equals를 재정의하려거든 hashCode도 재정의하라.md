# equals를 재정의하려거든 hashCode도 재정의하라

## 핵심 규약

equals를 재정의한 클래스는 반드시 hashCode도 재정의해야 한다. 그렇지 않으면 HashMap, HashSet 같은 컬렉션에서 제대로 동작하지 않는다.

## hashCode 규약

1. equals 비교에 사용되는 정보가 변경되지 않았다면, hashCode는 항상 같은 값을 반환해야 한다.
2. equals(Object)가 두 객체를 같다고 판단했으면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

## hashCode를 구현하지 않는다면?

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNum = lineNum;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber pn = (PhoneNumber) o;
        return areaCode == pn.areaCode 
            && prefix == pn.prefix 
            && lineNum == pn.lineNum;
    }
    
}
``` 

```java
public class Main {
    public static void main(String[] args) {
        Map<PhoneNumber, String> map = new HashMap<>();
        
        PhoneNumber number1 = new PhoneNumber(010, 1234, 5678);
        map.put(number1, "홍길동");
        
        PhoneNumber number2 = new PhoneNumber(010, 1234, 5678);
        
        System.out.println(number1.equals(number2)); // true
        
        System.out.println(map.get(number2)); // null (예상: "홍길동")
    }
}
```

### 왜 null이 나올까?

1. number1과 number2는 equals로 비교하면 같다.
2. 하지만 hashCode가 다르다. (기본 hashCode는 객체 주소 기반이다)
3. HashMap은 hashCode로 버킷을 찾는다.
4. number2의 hashCode로 찾은 버킷에는 number1이 없다.
5. 결과적으로 null 반환된다.

## hashCode의 구현
### 최악의 hashCode 구현법

```java
@Override
public int hashCode() {
    return 42; // 모든 객체가 같은 해시코드
}
```
위의 코드는 모든 객체가 같은 버킷에 저장되어 해시 테이블이 연결리스트 처럼 동작해서 O(1)의 성능이 O(n)으로 저하된다.

### 방법 1: 전형적인 구현

```java
@Override
public int hashCode() {
    int result = Integer.hashCode(areaCode);
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    return result;
}
```

31을 곱하는 이유

- 31은 홀수이면서 소수(prime)
- 31 * i는 (i << 5) - i로 최적화 가능
- 전통적으로 사용되어 온 관례

### 방법 2: Objects.hash 사용 (간편하지만 느리다)

```java
@Override
public int hashCode() {
    return Objects.hash(areaCode, prefix, lineNum);
}
```

장점: 한 줄로 간단하게 구현
단점: 속도가 느림 (배열 생성 및 박싱/언박싱 비용)

### 방법 3: 캐싱 사용 (불변 객체일 때)

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;
    
    private int hashCode; // 0으로 자동 초기화
    
    @Override
    public int hashCode() {
        int result = hashCode;
        if (result == 0) {
            result = Integer.hashCode(areaCode);
            result = 31 * result + Integer.hashCode(prefix);
            result = 31 * result + Integer.hashCode(lineNum);
            hashCode = result;
        }
        return result;
    }
}
```

장점: 해시 코드를 한 번만 계산 (지연 초기화)
적합한 경우: 불변 객체이고 해시 계산 비용이 클 때

### Ex 1) Person 클래스의 equals와 hashCode 재정의

```java
public class Person {
    private final String name;
    private final int age;
    private final String email;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return age == person.age 
            && Objects.equals(name, person.name) 
            && Objects.equals(email, person.email);
    }
    
    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        result = 31 * result + (email != null ? email.hashCode() : 0);
        return result;
    }
}
```

### 결론 
> equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다. 재정의한 hashCode는 Object의 API 문서의 기술된 일반 규약에 따라야하며 서로 다른 인스턴스라면 되도록 해시코드도 다르게 구현해야한다.