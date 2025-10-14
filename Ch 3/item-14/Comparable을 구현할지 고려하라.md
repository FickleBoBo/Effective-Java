# Item 14 : Comparable을 구현할지 고려하라

### 들어가며

객체들을 정렬해야 하는 상황에 종종 직면한다. 학생들을 성적순으로, 상품들을 가격순으로, 직원들을 입사일순으로 정렬하는 등의 작업 등이 있다.

Java는 이러한 정렬 작업을 위해 **`Comparable`** 과 **`Comparator`** 라는 두 가지 강력한 도구를 제공한다.이번 아이템 14에서는 `Comparable` 인터페이스를 구현할 때 지켜야 할 규약과 주의사항, 그리고 `Comparator` 와의 차이점을 살펴보며, 언제 어떤 방식을 선택해야 하는지 알아본다.

<br/>

## Comparable 인터페이스란?

> **객체 간의 순서를 비교할 수 있게 해주는 인터페이스.**

`Comparable` 인터페이스를 구현하면 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다.
![](https://velog.velcdn.com/images/kguswo/post/2f624190-f117-4b28-9c57-7bf59f86d590/image.png)

```java
public interface Comparable<T>{
	int comparTo(T t)
}
```

### `compareTo` 메서드의 일반 규약

다음 규약을 따라야 한다.

1. 반사성: 모든 x에 대해 **`sgn(x.compareTo(y)) == -sgn(y.compareTo(x))`** 여야 한다. 따라서 `x.compareTo(y)` 가 예외를 던지면 `y.compareTo(x)` 도 예외를 던져야 한다.

2. 추이성: **`(x.compareTo(y) > 0 && y.compareTo(z) > 0)`** 이면 **`x.compareTo(z) > 0`** 이다.

3. 일관성: **`x.compareTo(y) == 0`** 이면 모든 z에 대해 **`sgn(x.compareTo(z)) == sgn(y.compareTo(z))`** 다.

4. equals와의 일관성(권고): **`(x.compareTo(y) == 0) == (x.equals(y))`** 여야 한다. 이 권고를 지키지 않으면 명시해야 한다.

<br/>

#### 일관되지 않은 경우 예시; BigDecimal 비교 테스트

다음과 같이 테스트를 해보았다.

```java

	BigDecimal bd1 = new BigDecimal("1.0");
    
	BigDecimal bd2 = new BigDecimal("1.00");


    // equals
    System.out.println("1. equals() 비교:");
    System.out.println("bd1 = " + bd1);
    System.out.println("bd2 = " + bd2);
    System.out.println("bd1.equals(bd2) = " + bd1.equals(bd2));

    // compareTo
    System.out.println("2. compareTo() 비교:");
    System.out.println("bd1.compareTo(bd2) = " + bd1.compareTo(bd2));


    // HashSet
    System.out.println("3. HashSet 테스트 (equals 사용):");
    HashSet<BigDecimal> hashSet = new HashSet<>();
    hashSet.add(bd1);
    hashSet.add(bd2);
    System.out.println("HashSet 크기: " + hashSet.size());
    System.out.println("HashSet 내용: " + hashSet);

    // TreeSet
    System.out.println("4. TreeSet 테스트 (compareTo 사용):");
    TreeSet<BigDecimal> treeSet = new TreeSet<>();
    treeSet.add(bd1);
    treeSet.add(bd2);
    System.out.println("TreeSet 크기: " + treeSet.size());
    System.out.println("TreeSet 내용: " + treeSet);
```

---

실행 결과는 다음과 같다.

![](https://velog.velcdn.com/images/kguswo/post/76ca8605-6037-4c88-9cd3-f177de6d7bb4/image.png)

먼저 `equals` 는 scale(소수점 자릿수)까지 비교해서 false를 반환했고,`compareTo` 는 수학적 값만 비교하기 때문에 같다는 의미의 0을 반환한다.

![](https://velog.velcdn.com/images/kguswo/post/14faa60f-0591-4a8f-b37d-3bf134453535/image.png)

다음으로 `HashSet` 은 equals를 사용하기 때문에 2개의 원소로 취급되는 반면, `TreeSet` 은 compareTo로 같은 원소로 취급하기 때문에 1개다.

#### 결론
BigDecimal의 **`equals`와 `compareTo`는 일관성이 없다.**

이것은 `Comparable` 인터페이스 일반 규약을 위반하는 것은 아니지만, **정렬된 컬렉션 (TreeSet, TreeMap) 과 일반 컬렉션 (HashSet, HashMap) 에서 다르게 동작할 수 있기 때문에 주의해야한다.**


### compareTo 메서드 작성 요령

1. **기본 타입** 필드 비교

Java 7부터는 박싱된 기본 타입 클래스들에 추가된 정적 메서드 `compare` 를 사용한다.

```java
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {

    private final String s;
    
    @Override
    public int compareTo(CaseInsensitiveString cis) {
    
        return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
    
    }
    
}
```

2. **핵심 필드가 여러 개**일 때

가장 핵심적인 필드부터 비교하고, 비교 결과가 0이 아니면 즉시 반환한다.

```java
public int compareTo(PhoneNumber pn) {

    int result = Short.compare(areaCode, pn.areaCode);
    
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        
        if (result == 0) {
            result = Short.compare(lineNum, pn.lineNum);
        }
        
    }
    
    return result;
}
```

3. Comparator 인터페이스 활용 (Java 8 이후)

Java 8 에서는 **Comparator 인터페이스가 비교자 생성 메서드들을 제공**한다.

```java
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

<br/>

### Comparable vs Comparator: 무엇이 다른가?

### Comparable 인터페이스

> Comparable은 **객체 자신이 다른 객체와 어떻게 비교될지를 정의**하는 인터페이스다. 즉, 클래스 자체에 자연스러운 순서(natural ordering)를 부여한다.

**특징:**
- **자기 자신(this)과 매개변수 객체를 비교**
- 클래스 **내부에 구현**
- **단 하나의 정렬 기준**을 제공

**사용 예시:**

```java
public class Student implements Comparable<Student> {
    private String name;
    private int score;
    
    @Override
    public int compareTo(Student o) {
        return Integer.compare(this.score, o.score); // 점수로 정렬
    }
} 
```

### Comparator 인터페이스

> Comparator는 **외부에서 두 객체를 어떻게 비교할지를 정의**하는 인터페이스다. 클래스의 코드를 수정하지 않고도 다양한 정렬 기준을 제공할 수 있다.

![](https://velog.velcdn.com/images/kguswo/post/f78bae5d-92a8-4515-9002-a463a5cf61d8/image.png)

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
```

특징:

- **두 개의 매개변수 객체를 비교**한다
- 클래스 **외부에서 구현**한다
- **여러 가지 정렬 기준**을 제공할 수 있다
- Java 8부터 함수형 인터페이스로 지정되어 **람다식 사용이 가능**하다

```java
public class Student {
    private String name;
    private int score;
    // Comparable 구현 없음
}

// 점수로 정렬하는 Comparator
Comparator<Student> scoreComparator = (s1, s2) -> 
    Integer.compare(s1.getScore(), s2.getScore());

// 이름으로 정렬하는 Comparator
Comparator<Student> nameComparator = (s1, s2) -> 
    s1.getName().compareTo(s2.getName());
```

---

#### 키 추출자 (Key Extractor)
> 객체에서 **비교 기준이 되는 값을 추출하는 함수**다. Comparator의 정적 메서드들은 이 키 추출자를 받아서 자동으로 비교 로직을 생성해준다.

- 키 추출자의 실제 타입들: Java는 다양한 함수형 인터페이스를 키 추출자로 사용한다.

```java
// 일반 객체 추출 - Function<T, R>
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// int 추출 - ToIntFunction<T>
@FunctionalInterface
public interface ToIntFunction<T> {
    int applyAsInt(T value);
}

// long 추출 - ToLongFunction<T>
@FunctionalInterface
public interface ToLongFunction<T> {
    long applyAsLong(T value);
}

// double 추출 - ToDoubleFunction<T>
@FunctionalInterface
public interface ToDoubleFunction<T> {
    double applyAsDouble(T value);
}
```

<br/>

- 키 추출자 사용 예시:

```java
public class Student {
    private String name;
    private int score;
    private double gpa;
    
    // getter들
}

// 1. 메서드 참조로 키 추출자 전달
Comparator<Student> byScore = Comparator.comparingInt(Student::getScore);

// 2. 람다식으로 키 추출자 전달
Comparator<Student> byGpa = Comparator.comparingDouble(s -> s.getGpa());

// 3. 복잡한 키 추출 로직
Comparator<Student> byNameLength = 
    Comparator.comparingInt(s -> s.getName().length());
```

<br/>

- 키 추출자 내부 동작:

![](https://velog.velcdn.com/images/kguswo/post/7fa4d1d3-f4f6-4f2a-b4c8-87debbc37cab/image.png)

```java
// Comparator의 comparingInt 메서드
public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {

    Objects.requireNonNull(keyExtractor);
    
    return (Comparator<T> & Serializable)
        (c1, c2) -> Integer.compare(
            keyExtractor.applyAsInt(c1),  // c1에서 int 값 추출
            keyExtractor.applyAsInt(c2)   // c2에서 int 값 추출
        );
}
```

---

Comparator의 메서드들 (Java 8 이후)

1. `comparing` 계열 메서드: 키 추출 함수를 사용하여 Comparator를 생성한다.![](https://velog.velcdn.com/images/kguswo/post/c4315c6b-d776-48d5-99d4-bd9b9605ed87/image.png)


2. `thenComparing` 메서드: 여러 조건을 연쇄적으로 비교할 때 사용한다.

3. `reversed` 메서드: 역순 정렬을 위한 메서드다.

4. `nullsFirst` , `nullsLast` : null 값 처리를 위한 메서드다.

5. `naturalOrder` , `reverseOrder` : Comparable 구현 객체를 위한 기본 정렬 순서를 제공한다.

6. 전체 예제:

```java
public class Employee {
    private String department;
    private String name;
    private int salary;
    private LocalDate hireDate;
    
    // getter들
}

// 부서별로 묶고, 같은 부서 내에서는 급여 내림차순, 
// 급여가 같으면 입사일 오름차순으로 정렬
Comparator<Employee> complexComparator = Comparator
    .comparing(Employee::getDepartment)
    .thenComparing(Comparator.comparingInt(Employee::getSalary).reversed())
    .thenComparing(Employee::getHireDate);

List<Employee> employees = getEmployees();
employees.sort(complexComparator);
```

<br/>

#### Comparable vs Comparator 비교 전체 코드

```java
import java.util.*;

/**
 * Comparable vs Comparator 비교
 */

// 1. Comparable을 구현한 클래스
class Student implements Comparable<Student> {
    private String name;
    private int age;
    private int score;
    
    public Student(String name, int age, int score) {
        this.name = name;
        this.age = age;
        this.score = score;
    }
    
    // Comparable: 기본 정렬 기준 (나이순)
    @Override
    public int compareTo(Student other) {
        return Integer.compare(this.age, other.age);
    }

    // 아래는 IntelliJ IDE를 통해 자동 생성 후 조금의 수정 거친 메서드들 (정석 패턴)
    // equals 일반규약 지켜서 재정의하라 (Item 10)
    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        Student student = (Student) o;
        return age == student.age &&
               score == student.score &&
               Objects.equals(name, student.name);
    }

    // equals 재정의 하려거든 hashCode도 반드시 함께 재정의하라 (Item 11)
    @Override
    public int hashCode() {
        return Objects.hash(name, age, score);
    }

    // toString을 항상 재정의하라 (Item 12)
    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", score=" + score +
                '}';
    }

    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public int getScore() { return score; }

}

public class ComparableVsComparator {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("철수", 20, 85));
        students.add(new Student("영희", 22, 92));
        students.add(new Student("민수", 19, 88));
        students.add(new Student("수지", 21, 90));
        students.add(new Student("지훈", 20, 95));
        
        System.out.println("=== Comparable vs Comparator ===\n");
        
        System.out.println("원본 리스트:");
        System.out.println(students);
        System.out.println();
        
        // ========================================
        // 1. Comparable 사용 (기본 정렬)
        // ========================================
        System.out.println("1. Comparable 사용 - compareTo() 메서드");
        System.out.println("   클래스 내부에 정의된 '기본 정렬 기준' 사용");
        System.out.println("   Student 클래스에 정의: 나이순 정렬\n");
        
        List<Student> list1 = new ArrayList<>(students);
        Collections.sort(list1);  // Comparable의 compareTo() 사용
        System.out.println("   정렬 결과 (나이순):");
        System.out.println("   " + list1);
        System.out.println();
        
        // ========================================
        // 2. Comparator 사용 (다양한 정렬)
        // ========================================
        System.out.println("2. Comparator 사용 - compare() 메서드");
        System.out.println("   클래스 외부에서 '다양한 정렬 기준' 제공\n");
        
        // 2-1. 이름순 정렬
        System.out.println("   2-1. 이름순 정렬 (Comparator):");
        Comparator<Student> nameComparator = new Comparator<Student>() {
            @Override
            public int compare(Student s1, Student s2) {
                return s1.getName().compareTo(s2.getName());
            }
        };
        
        List<Student> list2 = new ArrayList<>(students);
        Collections.sort(list2, nameComparator);
        System.out.println("   " + list2);
        System.out.println();
        
        // 2-2. 점수순 정렬 (내림차순)
        System.out.println("   2-2. 점수순 정렬 - 높은 점수부터 (Comparator):");
        Comparator<Student> scoreComparator = new Comparator<Student>() {
            @Override
            public int compare(Student s1, Student s2) {
                return Integer.compare(s2.getScore(), s1.getScore()); // 내림차순
            }
        };
        
        List<Student> list3 = new ArrayList<>(students);
        Collections.sort(list3, scoreComparator);
        System.out.println("   " + list3);
        System.out.println();
        
        // 2-3. 복합 정렬: 나이순 → 같으면 점수순
        System.out.println("   2-3. 복합 정렬 - 나이순, 같으면 점수 높은순 (Comparator):");
        Comparator<Student> complexComparator = new Comparator<Student>() {
            @Override
            public int compare(Student s1, Student s2) {
                int ageCompare = Integer.compare(s1.getAge(), s2.getAge());
                if (ageCompare != 0) {
                    return ageCompare;
                }
                return Integer.compare(s2.getScore(), s1.getScore());
            }
        };
        
        List<Student> list4 = new ArrayList<>(students);
        Collections.sort(list4, complexComparator);
        System.out.println("   " + list4);
        System.out.println();
        
        // ========================================
        // 3. 람다식으로 간결하게 (Java 8+)
        // ========================================
        System.out.println("3. 람다식으로 Comparator 간결하게 작성:");
        System.out.println();
        
        System.out.println("   3-1. 점수순 (람다):");
        List<Student> list5 = new ArrayList<>(students);
        list5.sort((s1, s2) -> Integer.compare(s2.getScore(), s1.getScore()));
        System.out.println("   " + list5);
        System.out.println();
        
        System.out.println("   3-2. Comparator.comparing() 사용:");
        List<Student> list6 = new ArrayList<>(students);
        list6.sort(Comparator.comparing(Student::getName));
        System.out.println("   이름순: " + list6);
        System.out.println();
        
        System.out.println("   3-3. 복합 정렬 (Comparator 체이닝):");
        List<Student> list7 = new ArrayList<>(students);
        list7.sort(Comparator.comparing(Student::getAge)
                            .thenComparing(Comparator.comparing(Student::getScore).reversed()));
        System.out.println("   나이순 → 점수 높은순: " + list7);
        System.out.println();
    }
}
```

![](https://velog.velcdn.com/images/kguswo/post/d0478210-c8c0-4fb9-898d-3d262ca51ff3/image.png)


<br/>

#### 그럼 언제 무엇을 사용할까?

- Comparable을 사용하는 경우:

  - 객체에 자연스러운 **순서가 명확하게 하나** 존재할 때
  - **객체의 기본 정렬** 방식을 정의하고 싶을 때
  - 예: Integer, String, LocalDate 등

- Comparator를 사용하는 경우:

  - 다양한 정렬 기준이 필요할 때
  - **클래스의 소스 코드를 수정할 수 없을 때**
  - **정렬 기준을 런타임에** 선택하고 싶을 때
  - 예: 사용자 정의 정렬, 다중 조건 정렬 등

<br/>

### 주의사항 및 개선 방법

1. **_값의 차를 이용한 비교는 위험하다._**

아래는 해시코드 값의 차를 기준으로 하는 비교자 예시 코드다. 이것은 추이성을 위배한다.

```java
// 나쁜 예 - 정수 오버플로우 가능성
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode(); // 위험!
    }
};
```

대신 다음과 같이 개선할 수 있다.

```java
// 방법 1. 정적 compare 메서드 사용
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};

// 방법 2. Comparator 생성 메서드 사용
static Comparator<Object> hashCodeOrder = 
    Comparator.comparingInt(o -> o.hashCode());
```

#### Comparable로 얻는 이점

1. 정렬: Arrays.sort(a)와 같이 손쉽게 정렬할 수 있다.

2. 검색: 이진 검색 등을 사용할 수 있다.

3. 컬렉션: TreeSet, TreeMap 등 정렬된 컬렉션에서 사용할 수 있다.

4. 유틸리티: Collections 클래스의 다양한 알고리즘을 활용할 수 있다.

<br/>

### 마치며

`Comparable` 인터페이스는 객체에 자연스러운 순서를 부여하는 강력한 도구다. 순서를 고려해야 하는 값 클래스를 작성한다면 반드시 `Comparable` 인터페이스 구현을 고려하자.

- `compareTo` 규약을 반드시 지키자 (반사성, 추이성, 일관성)

- `equals` 와의 일관성을 유지하자 (필수는 아니지만 강력히 권장)

- 필드 비교 시 `<` , `>` 연산자 대신 `compare` 메서드를 사용하자 (오버플로우 방지)

- Comparable은 **기본 정렬**, Comparator는 **다양한 정렬 기준**에 사용하자

- Java 8 이후의 Comparator 메서드 체이닝으로 간결하고 읽기 쉬운 코드를 작성하자

이러한 원칙들을 지키면 안전하고 유지보수하기 쉬운 정렬 로직을 구현할 수 있다.

---

### References

- Effective Java 3/E