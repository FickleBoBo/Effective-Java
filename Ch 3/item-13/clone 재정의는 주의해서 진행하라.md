# Item 13: clone 재정의는 주의해서 진행하라

## 들어가며

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스(mixin interface)지만, 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는 데 있다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

### Cloneable 인터페이스의 역할

```java
public interface Cloneable {
    // 메서드가 하나도 없다!
}
```

Cloneable 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다.

- **Cloneable을 구현한 클래스:** clone()을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환
- **Cloneable을 구현하지 않은 클래스:** CloneNotSupportedException을 던짐

이는 인터페이스를 이례적으로 사용한 예이며, 따라하지 말아야 할 방식이다. 일반적으로 인터페이스를 구현한다는 것은 그 클래스가 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위인데, Cloneable의 경우에는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이다.

### clone 메서드의 일반 규약

Object 명세에서의 일반 규약:

**1. x.clone() != x**

복제본과 원본은 서로 다른 객체여야 한다. 반드시 참이어야 한다.

**2. x.clone().getClass() == x.getClass()**

복제본과 원본은 같은 클래스여야 한다. 일반적으로 참이지만 필수는 아니다.

**3. x.clone().equals(x)**

복제본과 원본은 논리적 동치성을 만족해야 한다. 일반적으로 참이지만 필수는 아니다.

**4. 관례상 반환된 객체는 super.clone을 호출해 얻어야 한다**

이 규약이 가장 중요하다. 만약 생성자 체인처럼 super.clone을 연쇄적으로 호출하지 않으면 하위 클래스에서 문제가 발생한다.

```java
// 잘못된 예
public class Parent implements Cloneable {
    @Override
    public Parent clone() {
        return new Parent(); // super.clone() 호출 안 함
    }
}

public class Child extends Parent {
    private int value;
    
    @Override
    public Child clone() {
        return (Child) super.clone(); // ClassCastException!
        // Parent.clone()이 new Parent()를 반환하므로
        // Child로 형변환할 수 없음
    }
}
```

```java
// 올바른 예
public class Parent implements Cloneable {
    @Override
    public Parent clone() {
        try {
            return (Parent) super.clone(); // 올바름
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

super.clone()을 사용하면 Object.clone()까지 올라가서 실제 인스턴스 타입(Child)의 객체를 생성하므로 형변환이 안전하다.

## 가변 상태를 참조하지 않는 클래스용 clone 메서드 

모든 필드가 기본 타입이거나 불변 객체를 참조한다면, 아무것도 수정할 필요가 없다. 단, 쓸데없는 복사를 지양한다는 관점에서 보면 불변 클래스는 굳이 clone 메서드를 제공하지 않는 것이 좋다.

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();        
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없다
    }   
}
```

- **공변 반환 타이핑(covariant return typing):** Object가 아닌 PhoneNumber를 반환한다 (자바 1.5부터 가능). 재정의한 메서드의 반환 타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다. 이 방식을 사용하면 클라이언트가 형변환하지 않아도 되므로 권장한다.
- **CloneNotSupportedException:** Cloneable을 구현했으므로 발생하지 않는다. 하지만 super.clone()이 검사 예외를 던지므로 try-catch로 감싸야 한다.
- 모든 필드가 기본 타입이거나 불변이므로 super.clone()만으로 충분하다.

## 가변 객체를 참조하는 클래스의 clone

클래스가 가변 객체를 참조하는 순간 앞서의 간단한 구현으로는 문제가 생긴다.

### Stack 클래스

Stack은 내부적으로 Object 배열로 원소를 저장한다. 이 배열은 가변 객체이므로 주의가 필요하다.

```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

### 잘못된 clone 구현

단순히 super.clone()만 호출하면 어떻게 될까?

```java
@Override
public Stack clone() {
    try {
        return (Stack) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

이 코드는 컴파일되고 실행도 되지만, 제대로 동작하지 않는다.

```java
Stack stack1 = new Stack();
stack1.push("첫 번째");
stack1.push("두 번째");

Stack stack2 = stack1.clone();

stack2.pop(); // "두 번째" 제거

System.out.println(stack1.size()); // 1 (정상처럼 보임)
System.out.println(stack1.pop());  // null! (버그)
```

### 왜 문제가 발생할까?

clone 메서드가 단순히 super.clone()의 결과를 그대로 반환한다면, 반환된 Stack 인스턴스의 size 필드는 올바른 값을 갖겠지만, **elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조**하게 된다.

원본이나 복제본 중 하나를 수정하면 다른 하나도 수정되어, 불변식을 해친다. 프로그램이 이상하게 동작하거나 NullPointerException을 던질 것이다.

**Stack 클래스의 하나뿐인 생성자를 호출한다면 이러한 상황은 절대 일어나지 않는다.** clone 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, **clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.**

### Stack 클래스 (올바른 구현)

Stack의 clone 메서드가 제대로 동작하려면 스택 내부 정보를 복사해야 하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출하는 것이다.

```java
@Override
public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone(); // 배열도 복제!
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

이제 원본과 복제본은 각자의 배열을 갖게 된다:

```
원본 stack1: elements ──> [첫 번째][두 번째][null][null]...

복제본 stack2: elements ──> [첫 번째][두 번째][null][null]... (별도 배열)
```

**주의사항:**
- elements 배열의 clone을 호출할 때는 형변환이 필요 없다. 배열의 clone은 런타임 타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 따라서 **배열을 복제할 때는 배열의 clone 메서드를 사용하라고 권장한다. 사실, 배열은 clone 기능을 제대로 사용하는 유일한 예라 할 수 있다.**
- 한편, elements 필드가 final이었다면 이 방식은 작동하지 않는다. final 필드에는 새로운 값을 할당할 수 없기 때문이다. 이는 근본적인 문제로, **직렬화와 마찬가지로 Cloneable 아키텍처는 '가변 객체를 참조하는 필드는 final로 선언하라'는 일반 용법과 충돌한다.** 단, 원본과 복제된 객체가 그 가변 객체를 공유해도 안전하다면 괜찮다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

## 복잡한 가변 객체의 clone

clone을 재귀적으로 호출하는 것만으로는 충분하지 않을 때도 있다.

### HashTable 클래스

HashTable은 내부적으로 버킷 배열을 사용하고, 각 버킷은 key-value 쌍을 담는 연결 리스트의 첫 번째 엔트리를 참조한다.

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    // ... 나머지 코드 생략
}
```

### 잘못된 구현 - 배열만 복제

Stack에서처럼 단순히 버킷 배열의 clone을 재귀적으로 호출해보자.

```java
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone(); // 배열만 복제
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

**문제점:**

복제본은 자신만의 버킷 배열을 갖지만, 이 배열은 원본과 같은 연결 리스트를 참조한다. 원본과 복제본 모두 예기치 않게 동작할 가능성이 생긴다.

```
원본: buckets[0] ──┐
                   ├──> Entry → Entry → Entry
복제본: buckets[0] ─┘
```

이를 해결하려면 각 버킷을 구성하는 연결 리스트까지 복사해야 한다.

### 올바른 구현 - 깊은 복사 (재귀 방식)

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;
    
    private static class Entry {
        final Object key;
        Object value;
        Entry next;
        
        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        // 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
        Entry deepCopy() {
            return new Entry(key, value,
                next == null ? null : next.deepCopy());
        }
    }
    
    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++)
                if (buckets[i] != null)
                    result.buckets[i] = buckets[i].deepCopy();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

private 클래스인 HashTable.Entry는 깊은 복사(deep copy)를 지원하도록 보강되었다. HashTable의 clone 메서드는 먼저 적절한 크기의 새로운 버킷 배열을 할당한 다음 원래의 버킷 배열을 순회하며 비지 않은 각 버킷에 대해 깊은 복사를 수행한다.

**문제점:**

Entry의 deepCopy 메서드는 자신이 가리키는 연결 리스트 전체를 복사하기 위해 자신을 재귀적으로 호출한다. 이 방법은 간단하며, 버킷이 너무 길지 않다면 잘 작동한다. 하지만 **연결 리스트가 길면 스택 오버플로를 일으킬 위험이 있다.** 재귀 호출 때문에 리스트의 원소 수만큼 스택 프레임을 소비하기 때문이다.

### 개선된 구현 - 반복자 사용

이 문제를 피하려면 deepCopy를 재귀 호출 대신 반복자를 써서 순회하는 방향으로 수정해야 한다.

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next)
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    return result;
}
```

이제 연결 리스트가 아무리 길어도 스택 오버플로 위험 없이 복사할 수 있다.

## clone 재정의 시 주의사항

### 1. 재정의 가능한 메서드를 호출하지 말 것

복잡한 가변 객체를 복제하는 또 다른 방법은, super.clone을 호출하여 얻은 객체의 모든 필드를 초기 상태로 설정한 다음, 원본 객체의 상태를 다시 생성하는 고수준 메서드들을 호출하는 것이다.

```java
// 위험한 코드
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];
        result.clear(); // 재정의 가능한 메서드 호출!
        for (Entry e : buckets)
            if (e != null)
                result.put(e.key, e.value);
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

이 방법은 간단하지만 저수준에서 바로 처리할 때보다는 느리다. 또한 Cloneable 아키텍처의 기초가 되는 필드 단위 객체 복사를 우회하기 때문에 전체 Cloneable 아키텍처와는 어울리지 않는 방식이기도 하다.

**중요:** 생성자에서는 재정의될 수 있는 메서드를 호출하지 않아야 하는데(아이템 19), clone 메서드도 마찬가지다. **clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 가능성이 크다.** 따라서 앞 문단에서 얘기한 put(key, value) 메서드는 final이거나 private이어야 한다.

### 2. public clone 메서드에서는 throws 절을 없애야 한다

```java
@Override
public PhoneNumber clone() { // throws 절 제거
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

Object의 clone 메서드는 CloneNotSupportedException을 던진다고 선언했지만 재정의한 메서드는 그렇지 않다. **public인 clone 메서드에서는 throws 절을 없애야 한다. 검사 예외를 던지지 않아야 그 메서드를 사용하기 편하기 때문이다**(아이템 71).

### 3. 상속용 클래스는 Cloneable을 구현해서는 안 된다

상속해서 쓰기 위한 클래스 설계 방식 두 가지(아이템 19) 중 어느 쪽에서든, 상속용 클래스는 Cloneable을 구현해서는 안 된다.

**방법 1: 제대로 작동하는 clone 메서드를 구현해 protected로 두고 CloneNotSupportedException도 던질 수 있다고 선언**

이 방식은 Object의 방식을 모방한 것으로, Cloneable 구현 여부를 하위 클래스에서 선택하도록 해준다.

```java
@Override
protected Object clone() throws CloneNotSupportedException {
    return super.clone();
}
```

**방법 2: clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 한다**

```java
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

### 4. 스레드 안전 클래스를 작성할 때는 동기화 필요

Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다(아이템 78). Object의 clone 메서드는 동기화를 신경 쓰지 않았다. 그러니 super.clone 호출 외에 다른 할 일이 없더라도 clone을 재정의하고 동기화해줘야 한다.

```java
@Override
public synchronized Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

## 더 나은 객체 복사 방식

사실 Cloneable을 구현하는 모든 클래스는 clone을 재정의해야 한다. 이때 접근 제한자는 public으로, 반환 타입은 클래스 자신으로 변경한다. 이 메서드는 가장 먼저 super.clone을 호출한 후 필요한 필드를 전부 적절히 수정한다. 일반적으로 이 말은 그 객체의 내부 '깊은 구조'에 숨어 있는 모든 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 함을 뜻한다.

그러나 이처럼 복잡한 경우는 드물다. 기본 타입 필드와 불변 객체 참조만 갖는 클래스라면 아무 필드도 수정할 필요가 없다. 단, 일련번호나 고유 ID는 비록 기본 타입이나 불변일지라도 수정해줘야 한다.

> 하지만 Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다. 그렇지 않은 상황에서는 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

### 복사 생성자 (Copy Constructor)

복사 생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다.

```java
public class Person {
    private String name;
    private int age;
    
    // 복사 생성자
    public Person(Person original) {
        this.name = original.name;
        this.age = original.age;
    }
}

// 사용
Person original = new Person("철수", 25);
Person copy = new Person(original);
```

### 복사 팩터리 (Copy Factory)

복사 팩터리는 복사 생성자를 모방한 정적 팩터리(아이템 1)다.

```java
public class Person {
    private String name;
    private int age;
    
    // 복사 팩터리
    public static Person newInstance(Person original) {
        return new Person(original.name, original.age);
    }
}

// 사용
Person copy = Person.newInstance(original);
```

### 복사 생성자/팩터리의 장점

복사 생성자와 그 변형인 복사 팩터리는 Cloneable/clone 방식보다 나은 면이 많다:

1. **언어 모순적인 객체 생성 메커니즘(생성자를 쓰지 않는 방식)을 사용하지 않는다**
2. **엉성하게 문서화된 규약에 기대지 않는다**
3. **정상적인 final 필드 용법과 충돌하지 않는다**
4. **불필요한 검사 예외를 던지지 않는다**
5. **형변환이 필요 없다**
6. **해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다**

예컨대 관례상 모든 컬렉션 구현체는 Collection이나 Map 타입을 받는 생성자를 제공한다. 인터페이스 기반 복사 생성자와 복사 팩터리의 더 정확한 이름은 '변환 생성자(conversion constructor)'와 '변환 팩터리(conversion factory)'다. 이들을 이용하면 클라이언트는 원본의 구현 타입에 얽매이지 않고 복제본의 타입을 직접 선택할 수 있다.

```java
// 변환 생성자 - TreeSet을 HashSet으로 변환하면서 복사
public HashSet(Collection<? extends E> c) { ... }

TreeSet<String> treeSet = new TreeSet<>();
treeSet.add("A");
treeSet.add("B");

HashSet<String> hashSet = new HashSet<>(treeSet);
```

## 결론

> Cloneable을 이미 구현한 클래스를 확장한다면 어쩔 수 없이 clone을 잘 작동하도록 구현해야 한다. 그렇지 않은 상황에서는 **복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다**. 단, **배열만은 clone 메서드 방식이 가장 깔끔한**, 이 규칙의 합당한 예외라 할 수 있다.

