# Item 37 - ordinal 인덱싱 대신 EnumMap을 사용하라

## 들어가며
배열이나 리스트에서 원소를 꺼낼 때 `ordinal()` 메서드로 인덱스를 얻는 코드가 있다. 하지만 이 방식은 여러 문제점을 가지고 있다. 이번 아이템에서는 `ordinal()` 인덱싱의 문제점과 이를 해결하는 `EnumMap`에 대해 알아보자.

## ordinal()을 배열 인덱스로 사용 - 안티 패턴

식물을 나타내는 클래스가 있다고 가정해보자. 이 식물들을 생애주기(한해살이, 여러해살이, 두해살이)별로 묶어야 한다.
```java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }
}
```

이를 구현하기 위해 `ordinal()`을 배열 인덱스로 사용하는 코드를 작성할 수 있다.

```java
// Using ordinal() to index into an array - DON'T DO THIS!  (Page 171)
Set<Plant>[] plantsByLifeCycleArr =
        (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycleArr.length; i++)
    plantsByLifeCycleArr[i] = new HashSet<>();
for (Plant p : garden)
    plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
// Print the results
for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
    System.out.printf("%s: %s%n",
            Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
}

```

동작은 하지만 문제가 많은 코드다.

### ordinal() 인덱싱의 문제점

1. 배열은 제네릭과 호환되지 않아 비검사 형변환을 수행해야 한다
   - 경고가 뜨는데 깔끔하게 컴파일되지 않는다

2. 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다
   - 인덱스가 무엇을 의미하는지 수동으로 표시해야 한다

3. 정확한 정수값을 사용한다는 것을 직접 보증해야 한다
   - 정수는 열거 타입과 달리 타입 안전하지 않기 때문에 잘못된 값을 사용하면 런타임에 `ArrayIndexOutOfBoundsException`이 발생하거나 잘못된 동작을 수행할 수 있다

## EnumMap 사용으로 해결

`EnumMap`은 열거 타입을 키로 사용하도록 설계한 매우 빠른 Map 구현체다. 위 코드를 `EnumMap`으로 다시 작성해보자.

```java
// Using an EnumMap to associate data with an enum (Page 172)
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
        new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```

훨씬 짧고 명료하게 작성된다.

### EnumMap의 장점

1. 안전하지 않은 형변환을 쓰지 않는다
2. 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공한다
   - 출력 결과에 직접 레이블을 달 필요가 없다
3. 배열 인덱스를 계산하는 과정에서 오류가 날 가능성이 원천봉쇄된다
4. 내부에서 배열을 사용하여 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 장점을 모두 가져갈 수 있게 된다.

### 스트림을 사용한 코드

스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다.
```java
// Naive stream-based approach - unlikely to produce an EnumMap!  (Page 172)
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle)));

```

이 코드는 `EnumMap`이 아닌 고유한 맵 구현체를 사용하기 때문에 `EnumMap`을 써서 얻은 공간과 성능 이점이 사라진다.

```java
// Using a stream and an EnumMap to associate data with an enum (Page 173)
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle,
                () -> new EnumMap<>(LifeCycle.class), toSet())));
```

스트림을 사용하면 `EnumMap`만 사용했을 때와는 살짝 다르게 동작한다. `EnumMap` 버전은 언제나 식물의 생애주기당 하나씩의 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다. 

## 두 열거 타입을 매핑하는 경우

이제 두 가지 열거 타입 값을 매핑하느라 `ordinal`을 두 번 쓴 배열들의 배열을 본 적이 있을지도 모르겠다. 다음은 상태(Phase)와 전이(Transition)를 매핑한 예제다.
```java
// ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것!
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

        // 행과 열을 Phase의 ordinal을 인덱스로 사용 - 따라 하지 말 것!
        private static final Transition[][] TRANSITIONS = {
            { null,    MELT,     SUBLIME  },
            { FREEZE,  null,     BOIL     },
            { DEPOSIT, CONDENSE, null     }
        };

        // 한 상태에서 다른 상태로의 전이를 반환한다
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

이 코드는 앞서 본 간단한 예제와 마찬가지로 컴파일러가 `ordinal()`과 배열 인덱스의 관계를 알 수 없다. 즉, `Phase`나 `Transition` 열거 타입을 수정하면서 표(`TRANSITIONS`)를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다. 그리고 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어난다.

전이 하나를 얻으려면 이전 상태(from)와 이후 상태(to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다.
```java
// EnumMap 버전
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // Initialize the phase transition map
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
```

여기서 새로운 상태인 플라즈마(PLASMA)를 추가해보자. 이 상태와 연결된 전이는 2개다. 기체에서 플라즈마로 변하는 이온화(ionization)와 플라즈마에서 기체로 변하는 탈이온화(deionization)다.

배열로 만든 코드를 수정하려면 새로운 상수를 `Phase`에 1개, `Transition`에 2개를 추가하고, 원소 9개짜리인 배열들의 배열을 원소 16개짜리로 교체해야 한다. 원소 수를 너무 적거나 많이 기입하거나, 잘못된 순서로 나열하면 런타임 오류가 날 것이다.

반면 `EnumMap` 버전에서는 상태 목록에 `PLASMA`를 추가하고, 전이 목록에 `IONIZE(GAS, PLASMA)`와 `DEIONIZE(PLASMA, GAS)`만 추가하면 끝이다.
```java
public enum Phase {
    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);

        ...
    }
}
```

나머지는 기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 적다.

## 마치며
배열의 인덱스를 얻기 위해 `ordinal()`을 쓰는 것은 일반적으로 좋지 않으니, 대신 `EnumMap`을 사용하라. 다차원 관계는 `EnumMap<..., EnumMap<...>>`으로 표현하라. 애플리케이션 프로그래머는 `Enum.ordinal`을 웬만해서는 사용하지 말아야 한다"(아이템 35)는 일반 원칙의 특수한 사례다.