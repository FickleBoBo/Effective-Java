
# Item 24 : 멤버 클래스는 되도록 static으로 만들라

### 들어가며

중첩 클래스(nested class)는 다른 클래스 안에 정의된 클래스를 말한다. 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.
중첩 클래스의 종류는 네 가지다.

- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

이 중 첫 번째를 제외한 나머지는 내부 클래스(inner class)에 해당한다. 이번 아이템에서는 각 중첩 클래스를 언제, 왜 사용해야 하는지 이야기한다.

<br/>

## 정적 멤버 클래스

정적 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다.

```java
public class Calculator {
    
    // 정적 멤버 클래스로 선언된 Operation 열거형
    public static enum Operation {
        PLUS("+") {
            public double apply(double x, double y) { return x + y; }
        },
        MINUS("-") {
            public double apply(double x, double y) { return x - y; }
        },
        TIMES("*") {
            public double apply(double x, double y) { return x * y; }
        },
        DIVIDE("/") {
            public double apply(double x, double y) { return x / y; }
        };
        
        private final String symbol;
        
        Operation(String symbol) {
            this.symbol = symbol;
        }
        
        public abstract double apply(double x, double y);
    }
    
    public double calculate(Operation op, double x, double y) {
        return op.apply(x, y);
    }
}
```

정적 멤버 클래스는 **바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 흔히 쓰인다.** `Operation`은 `Calculator`와 논리적으로 밀접하게 연관되어 있지만, `Calculator`의 인스턴스에 접근할 필요는 없다.

<br/>

## 비정적 멤버 클래스

비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 그래서 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다.

**주요 특징: 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없다.**

```java
public class Outer {
    private String name = "Outer";
    
    // 비정적 멤버 클래스
    public class Inner {
        public void printOuter() {
            System.out.println(name);  // 바깥 인스턴스의 필드 접근
        }
    }
    
    // 정적 멤버 클래스
    public static class StaticNested {
        public void print() {
            // System.out.println(name);  // 컴파일 에러!
        }
    }
}

// 사용 예시
Outer outer = new Outer();

// 비정적: 반드시 바깥 인스턴스를 통해 생성
Outer.Inner inner = outer.new Inner();  // outer.new 문법 필요!

// 정적: 바깥 인스턴스 없이 생성 가능
Outer.StaticNested nested = new Outer.StaticNested();
```

<br/>

비정적 멤버 클래스는 **어댑터**를 정의할 때 자주 쓰인다. 즉, 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용하는 것이다.

```java
public class MySet {
    private Object[] elements;
    private int size = 0;
    
    // 비정적 멤버 클래스 - 바깥 인스턴스 필요
    public class MyIterator implements Iterator {
        private int cursor = 0;
        
        @Override
        public boolean hasNext() {
            // 바깥 클래스의 size 필드에 접근
            return cursor < size;
        }
        
        @Override
        public E next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            // 바깥 클래스의 elements 배열에 접근
            return (E) elements[cursor++];
        }
    }
    
    public Iterator iterator() {
        return new MyIterator();  // 바깥 인스턴스(this) 자동 전달
    }
}
```

예컨대 `Map` 인터페이스의 구현체들은 보통 자신의 컬렉션 뷰(`keySet`, `entrySet`, `values`)를 구현할 때 비정적 멤버 클래스를 사용한다. 이들은 맵의 데이터에 접근해야 하므로 바깥 인스턴스가 필요하다.

<br/>

## 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여라

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다. 이 관계는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다.

### 비정적 멤버 클래스의 숨겨진 비용

```java
public class MyMap {
    
    // 잘못된 예: 비정적 멤버 클래스
    public class Entry {
        private K key;
        private V value;
        
        public K getKey() { return key; }
        public V getValue() { return value; }
        public void setValue(V value) { this.value = value; }
    }
}
```

위 코드에서 `Entry` 클래스는 `MyMap` 인스턴스를 전혀 사용하지 않는다. 그런데 비정적 멤버 클래스로 선언했기 때문에 **모든 `Entry` 인스턴스는 바깥 `MyMap` 인스턴스의 참조를 숨은 필드로 가지게 된다.**

#### 왜 비정적 멤버 클래스는 바깥 인스턴스 참조를 가질까?

비정적 멤버 클래스는 바깥 클래스의 인스턴스 멤버에 접근할 수 있다. 이를 가능하게 하려면 "어떤" 바깥 인스턴스인지 알아야 한다.

```java
public class Outer {
    private int value = 100;
    
    public class Inner {
        public void printValue() {
            System.out.println(value);  // 바깥 인스턴스의 value 접근
        }
    }
}

Outer outer1 = new Outer();  // value = 100
Outer outer2 = new Outer();  // value = 100

Outer.Inner inner1 = outer1.new Inner();
Outer.Inner inner2 = outer2.new Inner();

// inner1은 outer1의 value를 봐야 하고
// inner2는 outer2의 value를 봐야 함

// 어떻게 구분? → this$0 참조가 필요함
```

컴파일러는 비정적 멤버 클래스에 자동으로 `this$0`이라는 숨겨진 필드를 추가한다. 이 필드는 바깥 클래스 인스턴스를 가리키는 참조다.

```java
// 개발자가 작성한 코드
public class Inner {
    public void printValue() {
        System.out.println(value);
    }
}

// 컴파일러가 실제로 만드는 코드
public class Outer$Inner {
    final Outer this$0;  // 컴파일러가 자동으로 추가!
    
    Outer$Inner(Outer outer) {
        this.this$0 = outer;  // 바깥 인스턴스 참조 저장
    }
    
    public void printValue() {
        System.out.println(this$0.value);  // this$0를 통해 접근
    }
}
```

반면 정적 멤버 클래스는 바깥 인스턴스의 멤버에 접근할 수 없다. 접근할 필요가 없으니 `this$0` 참조도 필요 없다. 이것이 메모리를 절약하는 핵심 이유다.

#### 메모리 구조 비교

JVM 힙 메모리에 객체가 저장될 때, 비정적 멤버 클래스의 인스턴스는 바깥 인스턴스 참조를 필드로 저장한다. 64비트 JVM에서 객체 참조는 메모리 주소를 나타내므로 8바이트를 차지한다.

```
1. 비정적 Entry 객체 (나쁜 예):
메모리 주소: 0x2000

    객체 헤더      (12 byte)      ◀─── JVM이 자동 추가 (GC, 동기화 정보)
    key 참조       (8 byte)      ◀─── 0x3000 (String 객체 주소)
    value 참조     (8 byte)      ◀─── 0x4000 (Integer 객체 주소)
    this$0 참조    (8 byte)      ◀─── 0x1000 (MyMap 객체 주소)
                                    ^^^^^^^^ 이 참조가 낭비되는 부분!
총 36 byte


2. 정적 Entry 객체 (좋은 예):
메모리 주소: 0x2000

    객체 헤더        (12 byte)
    key 참조         (8 byte)
    value 참조       (8 byte)

총 28 byte (this$0 필드가 없음!)

Entry 1,000개 기준:
비정적: 36KB
정적:   28KB
절약:   8KB (22% 절약!)
```

**왜 참조는 8바이트인가?**

64비트 JVM에서는 메모리 주소가 64비트(8바이트)다. 모든 객체 참조는 메모리 주소를 저장하므로 8바이트를 차지한다. 예를 들어 `key` 참조는 `0x0000000000003000`처럼 8바이트 메모리 주소를 담고 있다.

**참조는 어디에 저장되는가?**

객체의 참조 필드들은 해당 객체가 할당된 힙 메모리 내부에 저장된다. 비정적 멤버 클래스의 경우, 컴파일러가 자동으로 추가한 `this$0` 필드도 객체 내부에 저장되어 8바이트를 차지한다.

```
힙 메모리 예시:

0x1000: MyMap 인스턴스
            ▲
            │ this$0 = 0x1000
            │
0x2000:
        Entry 인스턴스  
        key = 0x3000   
        value = 0x4000 
        this$0 = 0x1000 ◀─── 8바이트 낭비
```

Entry가 1,000개라면 각각이 8바이트씩 바깥 참조를 저장하므로 총 8,000바이트가 낭비된다. 더 심각한 문제는 이 참조 때문에 가비지 컬렉터가 바깥 클래스 인스턴스를 회수하지 못해 메모리 누수가 발생할 수 있다는 점이다.

### private 정적 멤버 클래스의 올바른 사용

```java
public class MyMap {
    
    // 올바른 예: private 정적 멤버 클래스
    private static class Entry {
        private K key;
        private V value;
        
        Entry(K key, V value) {
            this.key = key;
            this.value = value;
        }
        
        public K getKey() { return key; }
        public V getValue() { return value; }
        public void setValue(V value) { this.value = value; }
    }
    
    private Entry[] table;
    
    public V get(K key) {
        for (Entry entry : table) {
            if (entry != null && entry.getKey().equals(key)) {
                return entry.getValue();
            }
        }
        return null;
    }
}
```

#### static의 진짜 의미

많은 사람들이 `static` = "한 번만 생성된다"고 오해하지만, 이는 `static` 변수에만 해당하는 이야기다.

```java
// static 변수: 클래스당 1개만 존재
public class Counter {
    static int count = 0;  // 모든 인스턴스가 공유
}

// static 클래스: 여러 개 생성 가능!
public class MyMap {
    private static class Entry {
        // Entry 객체는 필요한 만큼 여러 개 만들 수 있음!
    }
}

MyMap map = new MyMap();
Entry e1 = new Entry();  // 첫 번째
Entry e2 = new Entry();  // 두 번째
Entry e3 = new Entry();  // 세 번째
// ✓ 여러 개 생성 가능!
```

**`static class`의 진짜 의미:**
- "한 번만 만든다" ❌
- "바깥 인스턴스와 무관하게 만든다" ⭕
- "각 인스턴스가 바깥 참조를 가지지 않는다" ⭕

private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다. 

키와 값을 매핑시키는 `Map` 인스턴스를 생각해보자. 많은 `Map` 구현체는 각각의 키-값 쌍을 표현하는 엔트리(Entry) 객체들을 가지고 있다. 모든 엔트리가 맵과 연관되어 있지만 엔트리의 메서드들(`getKey`, `getValue`, `setValue`)은 맵을 직접 사용하지 않는다. 

따라서 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비고, private 정적 멤버 클래스가 가장 알맞다. 엔트리를 비정적으로 선언해도 동작하겠지만, 모든 엔트리가 바깥 맵으로의 참조를 갖게 되어 공간과 시간을 낭비하게 된다.

<br/>

## 익명 클래스

익명 클래스는 이름이 없다. 또한 바깥 클래스의 멤버도 아니다. 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다. 코드의 어디서든 만들 수 있다.

```java
// 익명 클래스 예시
List numbers = Arrays.asList(3, 1, 4, 1, 5);

Collections.sort(numbers, new Comparator() {
    @Override
    public int compare(Integer a, Integer b) {
        return a - b;
    }
});
```

익명 클래스는 람다가 등장하기 전에 즉석에서 작은 함수 객체나 처리 객체를 만드는 데 주로 사용했다. 하지만 이제는 람다에게 그 자리를 내주었다.

```java
// 람다로 대체
Collections.sort(numbers, (a, b) -> a - b);
```

익명 클래스의 또 다른 주요 쓰임은 정적 팩터리 메서드를 구현할 때다.

<br/>

## 지역 클래스

지역 클래스는 네 가지 중첩 클래스 중 가장 드물게 사용된다. 지역 변수를 선언할 수 있는 곳이면 어디서든 선언할 수 있고, 유효 범위도 지역 변수와 같다.

```java
public class LocalClassExample {
    private int instanceVar = 10;
    
    public void someMethod() {
        final int localVar = 20;
        
        // 지역 클래스
        class LocalClass {
            public void print() {
                System.out.println(instanceVar);  // 바깥 인스턴스 접근 가능
                System.out.println(localVar);     // 지역 변수 접근 가능
            }
        }
        
        LocalClass local = new LocalClass();
        local.print();
    }
}
```

지역 클래스는 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다. 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버는 가질 수 없다.

<br/>

---

### References

- 이펙티브 자바 3/E