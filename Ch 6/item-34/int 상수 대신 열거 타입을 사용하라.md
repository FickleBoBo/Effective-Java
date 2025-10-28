# Item 34 - int 상수 대신 열거 타입을 사용하라

## 들어가며
우리가 일반적으로 사용하는 정수 열거 패턴 기법에는 단점이 많다. 타입 안전성을 보장할 수 없으면서 표현력도 떨어진다. 따라서 이런 열거 패턴의 단점을 해결하는 열거 타입(enum type)에 대해 알아보자

### 가장 단순한 열거 타입

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

겉보기에는 c계열의 다른 언어들의 열거 타입과 비슷하지만 java에서의 열거 타입은 완전한 형태의 클래스라서 다른 언어의 열거 타입보다 훨씬 강력하다. java에서 열거 타입을 뒷받침하는 아이디어는 생각보다 단순하다. 열거 타입 자체는 클래스이며 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다. 또한 열거 타입은 밖에서 접근할 수 있는 생성자를 막아서 사실상 final이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재하는 즉 싱글턴이다.

더하여 열거타입은 메서드와 필드를 추가할 수 있는데, 각 상수와 연관된 데이터를 해당 상수 자체에 내재시키고 싶을 때 사용하면 된다. 다음은 책에서 예시로 든 태양계의 열거 타입이다.

```java
// Enum type with data and behavior  (159-160)
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // In kilograms
    private final double radius;         // In meters
    private final double surfaceGravity; // In m / s^2

    // Universal gravitational constant in m^3 / kg s^2
    private static final double G = 6.67300E-11;

    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}

```

열거 타입 상수 각각을 특정 데이터와 연결 지으려면 생성자에서 데이터를 받아 인스턴스에 저장하면 된다. 여기서 지켜야할 것이 있다.
1. 열거타입은 근본적으로 불변이라 모든 필드를 final로 정의해야한다.
2. 필드를 public으로 선언해도 되지만 final로 선언해두고 public 접근자 메서드를 두는 것이 낫다.

위의 코드를 활용하여 여덟 행성에서의 무게를 출력하는 일을 다음과 같이 작성할 수 잇다.

```java
// Takes earth-weight and prints table of weights on all planets (Page 160)
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("Weight on %s is %f%n",
                 p, p.surfaceWeight(mass));
   }
}
```

열거타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 `values()`를 제공하니 유용하게 사용해주면 된다. 명왕성이 태양계에서 빠진 것을 반영할 때에는 단순히 상수에서 하나 지워주고 다시 컴파일만 하면 될 것이다. 

### 값에 따라 분기하는 열거 타입 

```java
public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS:   return Operation.MINUS;
        case MINUS:  return Operation.PLUS;
        case TIMES:  return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;

        default:  throw new AssertionError("Unknown op: " + op);
    }
}
```

동작은 하지만 예쁜 코드는 아니다. 깨지기 쉽고 상수가 추가되는 경우 case문을 추가하지 않는다면 "Unknow op:" 라는 런타임 오류를 내면서 프로그래밍 종료되기 떄문이다. 따라서 열거타입은 상수별로 다른게 동작하는 코드를 구현하는 수단을 제공하는데 이를 상수별 메서드 구현(constant-specific method implementation)이라고 한다. 

상수별 메서드 구현을 적용한 코드이다.
```java
// Enum type with constant-specific class bodies and data (Pages 163-4)
public enum Operation {
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

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);

    // Implementing a fromString method on an enum type (Page 164)
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

    // Returns Operation for string, if any
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
```

`apply()`가 추상 메서드이므로 재정의하지 않으면 컴파일 오류가 발생한다. 따라서 상수를 추가할 때 구현을 깜빡할 수 없다.

한편 열거타입의 `toString()`을 재정의할 땐 `toString()`이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 `fromString()`메서드의 구현을 고려해보자.

### 전략 열거 타입 패턴
```java
// The strategy enum pattern (Page 166)
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // The strategy enum type
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}

```

상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다. 위의 급여명세서 예제에서 switch문을 사용한다면 휴가와 같은 새로운 값이 열거 타입에 추가될 때 case문을 깜빡할 수 있다. 이런 경우를 대비한 것이 전략 열거 타입 패턴이다.

전략 열거 타입 패턴은 잔업수당 계산을 private 중첩 열거 타입(`PayType`)으로 옮기고, `PayrollDay` 열거 타입의 생성자에서 이 중 적당한 것을 선택하도록 한다. 그러면 `PayrollDay` 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch문이나 상수별 메서드 구현이 필요 없게 된다. 

이 패턴은 switch문보다 복잡하지만 더 안전하고 유연하다. 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 강제하기 때문이다.

## 마치며
그렇다면 언제 열거 타입을 사용해야할까? 필요한 원소를 컴파일 시점에 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자. 열거 타입에 정의된 상수 개수가 영원히 고정되어 불변일 필요는 없고 나중에 상수가 추가되어도 호환 되도록 설계되었기 떄문이다.