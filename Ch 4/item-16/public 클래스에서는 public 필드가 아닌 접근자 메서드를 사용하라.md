# Item 16: public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 들어가며

때때로 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있다.

### 퇴보한 클래스 - public 필드를 직접 노출

```java
class Point {
    public double x;
    public double y;
}
```

이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.

문제점
1. API를 수정하지 않고는 내부 표현을 바꿀 수 없다
   - 만약 x, y를 극좌표(r, θ)로 바꾸고 싶다면? 모든 클라이언트 코드를 수정해야 한다.

2. 불변식을 보장할 수 없다
   ```java
   Point p = new Point();
   p.x = -1;  // 음수 좌표를 허용하고 싶지 않아도 막을 수 없다
   p.y = 999; // 범위를 제한하고 싶어도 막을 수 없다
   ```

3. 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다
   - 필드가 변경될 때 로깅을 남기고 싶다면? 불가능하다.
   - 필드 접근 횟수를 카운팅하고 싶다면? 불가능하다.

## 개선된 클래스 - 접근자와 변경자(mutator) 메서드 사용
필드들을 모두 private으로 바꾸고 public 접근자(getter)를 추가하는 방법으로 개선할 수 있다.

```java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

**public 클래스**에서라면, 이 방식이 확실히 맞다. **패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공**함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

### 접근자를 사용하면 얻는 이점

```java
class Point {
    private double x;
    private double y;
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    public void setX(double x) {
        if (x < 0) {
            throw new IllegalArgumentException("x는 음수일 수 없습니다");
        }
        this.x = x;
    }
    
    public void setY(double y) {
        if (y < 0) {
            throw new IllegalArgumentException("y는 음수일 수 없습니다");
        }
        this.y = y;
    }
}
```

이제 불변식을 강제할 수 있다!

```java
Point p = new Point(3, 4);
p.setX(-1); // IllegalArgumentException 발생!
```

## package-private 클래스 또는 private 중첩 클래스라면 예외

하지만 **package-private 클래스** 혹은 **private 중첩 클래스**라면 데이터 필드를 노출한다 해도 하등 문제가 없다. 

### package-private 클래스 예시

```java
class Point {
    public double x;
    public double y;
    
    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
}
```

이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔하다. 

**왜 괜찮은가?**

클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, **클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.** 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.

### private 중첩 클래스 예시

```java
public class TopPoint {
    // private 중첩 클래스
    private static class Point {
        public double x;
        public double y;
        
        public Point(double x, double y) {
            this.x = x;
            this.y = y;
        }
    }
    
    private Point center;
    
    public TopPoint(double x, double y) {
        this.center = new Point(x, y);
    }
    
    public double getCenterX() {
        return center.x; // 중첩 클래스의 public 필드에 직접 접근
    }
    
    public double getCenterY() {
        return center.y;
    }
}
```

private 중첩 클래스의 경우라면 수정 범위가 더욱 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.

## public 클래스의 필드가 불변이라면?

public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 여전히 좋은 생각은 아니다.

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    public final int hour;
    public final int minute;
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    
    // 나머지 코드 생략
}
```

**불변 필드를 노출한 경우:**

1.  **불변식은 보장할 수 있다**
   - final 필드이므로 생성 후 값이 변경되지 않는다
   - 생성자에서 유효성 검사를 할 수 있다

2.  **여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없다**
   ```java
   Time t = new Time(13, 30);
   int hour = t.hour; // 모든 클라이언트가 이렇게 접근
   
   // 나중에 내부를 "분 단위"로 저장하도록 바꾸고 싶다면?
   // 모든 클라이언트 코드를 수정해야 한다!
   ```

3.  **필드를 읽을 때 부수 작업을 수행할 수 없다**
   ```java
   // 접근 횟수를 세고 싶다면? 불가능
   // 로깅을 남기고 싶다면? 불가능
   ```

따라서 불변 필드라 하더라도 접근자 메서드를 제공하는 것이 낫다:

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;
    
    private final int hour;
    private final int minute;
    
    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    
    public int getHour() { return hour; }
    public int getMinute() { return minute; }
    
    // 나머지 코드 생략
}
```

자바 플랫폼 라이브러리에도 public 클래스의 필드를 직접 노출하지 말라는 규칙을 어긴 사례가 있다.

대표적인 예가 `java.awt.package` 패키지의 Point와 Dimension 클래스다. 이 클래스들은 지금까지도 심각한 성능 문제를 일으키고 있다(아이템 67). 

내부를 노출한 Dimension 클래스의 심각한 성능 문제는 오늘날까지도 해결되지 못했다. 이 사례를 반면교사로 삼아 절대 흉내 내서는 안 된다.

## 결론

> public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.
