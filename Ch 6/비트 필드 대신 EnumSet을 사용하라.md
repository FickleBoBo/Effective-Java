# Item 36: 비트 필드 대신 EnumSet을 사용하라

## 들어가며

과거에는 열거한 값들을 집합으로 사용할 때, 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용했다.

```java
public class Text {
    public static final int STYLE_BOLD          = 1 << 0;  // 1
    public static final int STYLE_ITALIC        = 1 << 1;  // 2
    public static final int STYLE_UNDERLINE     = 1 << 2;  // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3;  // 8
    
    public void applyStyles(int styles) { ... }
}
```

비트별 OR를 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이를 비트 필드(bit field)라 한다.

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

## 비트 필드의 문제점

1. 비트 필드 값이 그대로 출력되면 해석하기 어렵다.
2. 비트 필드에 녹아 있는 모든 원소를 순회하기 까다롭다.
3. 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다. API를 수정하지 않고는 비트 수를 늘릴 수 없다.

## 더 나은 대안: EnumSet

`EnumSet` 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현한다. `Set` 인터페이스를 구현하며 타입 안전하고 다른 `Set` 구현체와도 함께 사용할 수 있다.

내부적으로는 비트 벡터로 구현되어 있어 성능도 비트 필드만큼 좋다. 원소가 64개 이하라면 `EnumSet` 전체를 `long` 변수 하나로 표현하여 비트 필드 수준의 성능을 보인다. 그러면서도 비트를 직접 다룰 때의 에러들이 발생하지 않아 유용하다.

```java
// EnumSet - 비트 필드를 대체하는 현대적 기법
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set을 넘겨도 되나 EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) { ... }
}
```

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

`applyStyles` 메서드가 `EnumSet<Style>`이 아닌 `Set<Style>`을 받는 이유는 인터페이스로 받는 게 일반적으로 좋은 습관이기 때문이다(Item 64). 특이한 클라이언트가 다른 `Set` 구현체를 넘기더라도 처리할 수 있다.

## 마치며

열거 타입을 집합으로 사용한다고 해도 비트 필드를 사용할 이유는 없다. `EnumSet`이 비트 필드 수준의 성능과 열거 타입의 장점을 모두 제공하기 때문이다. 유일한 단점은 불변 `EnumSet`을 만들 수 없다는 것인데, `Collections.unmodifiableSet`으로 감싸 사용할 수 있다.