## Item 12 : toString을 항상 재정의하라

### 들어가며
자바에서 클래스를 만들 때 `equals()`, `hashCode()` 재정의의 중요성은 잘 알고 있다. 하지만 `toString()` 메서드의 재정의는 종종 간과된다. 

이펙티브 자바 아이템 12는 toString()을 항상 재정의하는 것이 얼마나 중요한지 강조한다.

## Object의 기본 toString은 쓸모가 없다

개발자가 `toString()`을 재정의하지 않으면 `java.lang.Object` 클래스의 기본 구현이 상속된다. 실제 JDK의 `Object.toString()` 구현을 보자.

```java
/**
 * Returns a string representation of the object.
 * @implSpec
 * The {@code toString} method for class {@code Object}
 * returns a string consisting of the name of the class of which the
 * object is an instance, the at-sign character `{@code @}', and
 * the unsigned hexadecimal representation of the hash code of the
 * object. In other words, this method returns a string equal to the
 * value of:
 * <blockquote>
 * <pre>
 * getClass().getName() + '@' + Integer.toHexString(hashCode())
 * </pre></blockquote>
 */
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

이 코드에서 볼 수 있듯 기본 `toString()`은 **"클래스_이름@16진수_해시코드"** 형태의 문자열을 반환한다.
ex) `PhoneNumber@adbbd`

이 문자열은 **객체가 어떤 정보를 담고 있는지 전혀 알려주지 않는다.** 디버깅이나 로깅 시 아무런 유용한 정보를 제공하지 않기 때문에 사실상 쓸모가 없다.


## toString 재정의의 이유

`toString()` 메서드는 우리가 예상하는 것보다 훨씬 많은 곳에서 자동으로 호출된다.

1. 디버깅 및 로깅: 객체를 `System.out.println()`으로 출력하거나 로깅 프레임워크에 객체를 남길 때 자동으로 호출된다.
2. 문자열 연결: 객체를 문자열 연결 연산자( `+` )와 함께 사용할 때 자동으로 호출된다.

```java
"객체 정보: " + myObject  // toString()이 자동 호출됨
```

3. 예외 메시지: `assert` 구문이나 예외 메시지에 객체를 포함할 때도 사용된다.

제대로 구현된 `toString()` 덕분에 개발자는 객체를 출력하기만 하면 그 객체의 현재 상태를 한눈에 파악할 수 있다.

### Object의 toString 규약
> `Object.toString()` 의 Javadoc을 보면 다음과 같은 규약이 명시되어 있다.

![](https://velog.velcdn.com/images/kguswo/post/608c1081-76bc-4908-8b88-fea081704300/image.png)

핵심은 **간결하면서 사람이 읽기 쉬운 형태의 유익한 정보** 를 반환해야 한다는 것이다.
또한 **모든 하위 클래스에서 이 메서드를 재정의할 것을 권장한다.**

---

## toString을 잘 구현하는 방법

### 1. 객체가 가진 주요 정보를 모두 반환하라

`toString()` 메서드는 객체가 가진 모든 핵심 필드의 값을 담는 것이 좋다. 그래야 디버깅 효율이 극대화된다.

```java
public class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNum;

    // 좋은 toString 구현
    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
}
```

```java
PhoneNumber jenny = new PhoneNumber(707, 867, 5309);
System.out.println("제니 번호: " + jenny);
// 출력: 제니 번호: 707-867-5309

// toString을 재정의하지 않았다면
// 출력: 제니 번호: PhoneNumber@adbbd (쓸모없는 정보)
```

### 2. 반환 값의 포맷을 문서화할지 결정하라

`toString()` 반환 값의 포맷을 명확하게 문서화할지 여부를 결정해야 한다.

**포맷을 명시하는 경우:**

- 표준적이고 명확한 표현 제공
- 다른 프로그램이 파싱 가능
- 단, 한번 명시하면 평생 그 포맷에 얽매임

**포맷을 명시하지 않는 경우:**

- 유연하게 포맷 변경 가능
- 대부분의 경우 이 방식을 권장

### 3. toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하라

`toString()`이 반환하는 정보를 개별 필드로도 접근할 수 있도록 `getter`를 제공해야 한다. 그렇지 않으면 프로그래머들이 `toString()` 반환값을 파싱할 수밖에 없다.

```java
public class PhoneNumber {
    private final short areaCode;
    private final short prefix;
    private final short lineNum;

    // getter 제공
    public short getAreaCode() { return areaCode; }
    public short getPrefix() { return prefix; }
    public short getLineNum() { return lineNum; }

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
}
```

---

### 실제 JDK의 `String.toString()` 구현

`String` 클래스는 이미 완벽한 `toString` 구현의 예시다.

```java
/**
 * This object (which is already a string!) is itself returned.
 *
 * @return  the string itself.
 */
public String toString() {
    return this;
}
```

`String` 은 그 자체로 이미 문자열이므로 자기 자신을 반환한다. 이는 완벽하게 "간결하면서 유익한 정보"라는 규약을 만족한다.

### toString 재정의 예시

```java
public class Person {
    private final String name;
    private final int age;
    private final String city;

    public Person(String name, int age, String city) {
        this.name = name;
        this.age = age;
        this.city = city;
    }

    @Override
    public String toString() {
        return String.format("Person[name=%s, age=%d, city=%s]", name, age, city);
    }
}

// 사용 예
Person p = new Person("홍길동", 30, "서울");

// toString을 재정의했을 때
System.out.println(p); 
// 출력: Person[name=홍길동, age=30, city=서울]

// toString을 재정의하지 않았을 때
// 출력: Person@15db9742 (쓸모없는 정보)
```

### toString을 재정의하지 않아도 되는 경우
다음 경우에는 `toString()` 재정의가 불필요하다.

1. 정적 유틸리티 클래스: 인스턴스를 만들지 않으므로 불필요
2. 열거 타입(enum): 자바가 이미 완벽한 `toString`을 제공
3. 이미 완벽한 `toString`을 제공하는 상위 클래스를 상속한 경우

---

### References

- 이펙티브 자바 3/E