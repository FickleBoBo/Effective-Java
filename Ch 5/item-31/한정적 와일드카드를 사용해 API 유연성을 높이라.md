# Item 31 한정적 와일드카드를 사용해 API 유연성을 높이라

## 들어가며
매개변수화 타입은 불공변(invariant)이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때 List<Type1>과 List<Type2>는 상위 타입도 아니고 하위 타입도 아니다. 하지만 때론 불공변 방식보단 유연한 무언가가 필요하다. 책에서는 Stack 클래스의 예시를 들며 API의 유연성을 높이는 방법을 이야기한다.

## pushAll()
일련의 원소를 스택에 넣는 메서드를 추가해야한다고 해보자.
```Java
// pushAll staticfactory without wildcard type - deficient!
public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
}
```

이 메서드는 컴파일은 되지만 완벽하지 않다. Stack<Number>로 선언한 후 pushAll(intVal)을 호출하면 논리적으로는 잘 작동할 것 같지만 실제로는 매개변수화 타입의 불공변으로 인한 에러가 발생한다. Java에서는 이러한 상황에 대처할 수 있게 한정적 와일드카드 타입이라는 특별한 매개변수화 타입을 지원한다. 

pushAll의 매개변수 타입은 'E의 Iterable'뿐만 아니라 'E의 하위 타입의 Iterable'도 받을 수 있어야 하며, 와일드카드 타입 Iterable<? extends E>가 정확히 이런 뜻이다. 수정해보면 다음과 같다. 

```Java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

## popAll()
다음은 Stack 안의 모든 원소를 주어진 컬렉션으로 담은 popAll() 메서드를 작성해보자

```Java
// popAll staticfactory without wildcard type - deficient!
public void popAll(Collection<E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

이번에도 주어진 원소 타입이 스택의 원소 타입과 일치한다면 문제 없이 동작한다. 하지만 Stack<Number>의 원소를 Object용 컬렉션으로 옮기면 pushAll에서 발생했던 에러와 비슷한 에러가 발생한다. 이번에는 타입이 'E의 Collection'뿐만 아니라 'E의 상위 타입의 Collection'도 받을 수 있어야 한다. Collection<? super E>가 정확히 이런 의미이다.

```Java
// Wildcard type for parameter that serves as an E consumer
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

이제 Stack과 클라이언트 코드 모두 깔끔하게 컴파일된다.

## PECS: Producer-Extends, Consumer-Super

유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용해야한다. 한편, 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다. 타입을 정확히 지정해야 하는 상황이므로 이때는 와일드카드 타입을 쓰지 말아야 한다.

어떤 와일드카드 타입을 써야 하는지 기억하는 **PECS**(producer-extends, consumer-super) 라는 공식이 있다.

즉, 매개변수화 타입 T가 생산자라면 `<? extends T>`를 사용하고, 소비자라면 `<? super T>`를 사용하라. Stack 예에서 pushAll의 src 매개변수는 Stack이 사용할 E 인스턴스를 생산하므로 src의 적절한 타입은 `Iterable<? extends E>`이다. 한편, popAll의 dst 매개변수는 Stack으로부터 E 인스턴스를 소비하므로 dst의 적절한 타입은 `Collection<? super E>`이다. PECS 공식은 와일드카드 타입을 사용하는 기본 원칙이다.

## item - 30에서의 max()를 개선해보자

Item 30에서 다룬 max 메서드를 다시 살펴보자.
```Java
public static <E extends Comparable<E>> E max(List<E> list)
```

다음은 와일드카드 타입을 사용해 개선한 모습이다.
```Java
public static <E extends Comparable<? super E>> E max(List<? extends E> list)
```

이번에는 PECS 공식을 두 번 적용했다. 

첫 번째, 입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 `List<E>`를 `List<? extends E>`로 변경했다. 

두 번째는 타입 매개변수다. 원래 선언에서는 E가 `Comparable<E>`를 확장한다고 정의했는데, 이때 Comparable은 E 인스턴스를 소비한다(그리고 선후 관계를 뜻하는 정수를 생산한다). 그래서 매개변수화 타입 `Comparable<E>`를 한정적 와일드카드 타입인 `Comparable<? super E>`로 대체했다. Comparable은 언제나 소비자이므로 일반적으로 **`Comparable<E>`보다는 `Comparable<? super E>`를 사용하는 편이 낫다**. Comparator도 마찬가지다. 일반적으로 **`Comparator<E>`보다는 `Comparator<? super E>`를 사용하는 편이 낫다**.

수정된 max는 다음과 같이 사용할 수 있다.
```Java
List<ScheduledFuture<?>> scheduledFutures = ...;
```

수정 전 max로는 이 리스트를 처리할 수 없다. `ScheduledFuture`가 `Comparable<ScheduledFuture>`를 구현하지 않았기 때문이다. `ScheduledFuture`는 `Delayed`의 하위 인터페이스이고, `Delayed`는 `Comparable<Delayed>`를 확장했다. 다시 말해, `ScheduledFuture`의 인스턴스는 다른 `ScheduledFuture` 인스턴스뿐 아니라 `Delayed` 인스턴스와도 비교할 수 있어서 수정 전 max가 이 리스트를 거부하는 것이다. 

더 일반화해서 말하면, `Comparable`(혹은 `Comparator`)을 직접 구현하지 않고, 직접 구현한 다른 타입을 확장한 타입을 지원하기 위해 와일드카드가 필요하다.

## 타입 매개변수와 와일드카드

타입 매개변수와 와일드카드에는 공통되는 부분이 있어서, 메서드를 정의할 때 둘 중 어느 것을 사용해도 괜찮을 때가 많다. 예를 들어 주어진 리스트에서 명시한 두 인덱스의 아이템들을 교환(swap)하는 정적 메서드를 두 방식 모두로 정의해보자.
```Java
// 비한정적 타입 매개변수 사용
public static <E> void swap(List<E> list, int i, int j);

// 비한정적 와일드카드 사용
public static void swap(List<?> list, int i, int j);
```

public API라면 간단한 두 번째가 낫다. 어떤 리스트든 이 메서드에 넘기면 명시한 인덱스의 원소들을 교환해줄 것이다. 신경 써야 할 타입 매개변수도 없다.

**기본 규칙: 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라.** 비한정적 타입 매개변수라면 비한정적 와일드카드로, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾸면 된다.

그런데 두 번째 swap 선언에는 문제가 하나 있다. 다음과 같이 아주 직관적으로 구현한 코드가 컴파일되지 않는다는 것이다.
```Java
public static void swap(List list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

방금 꺼낸 원소를 리스트에 다시 넣을 수 없다는 오류 메시지가 나온다. 원인은 리스트의 타입이 `List<?>`인데, `List<?>`에는 null 외에는 어떤 값도 넣을 수 없다는 데 있다.

해결책은 와일드카드 타입의 실제 타입을 알려주는 메서드를 private 핼퍼 메서드로 따로 작성하여 활용하는 것이다. 실제 타입을 알아내려면 이 헬퍼 메서드는 제네릭 메서드여야 한다.
```Java
public static void swap(List list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 헬퍼 메서드
private static  void swapHelper(List list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
```

swapHelper 메서드는 리스트가 `List<E>`임을 알고 있다. 즉, 이 리스트에서 꺼낸 값의 타입은 항상 E이고, E 타입의 값이라면 이 리스트에 넣어도 안전함을 알고 있다. 다소 복잡하게 구현했지만 외부에서는 와일드카드 기반의 깔끔한 선언을 유지할 수 있다. swap 메서드 내부에서는 더 복잡한 제네릭 메서드를 이용했지만, 덕분에 외부에서는 신경 쓸 게 없어졌다.

## 마치며

조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다. PECS 공식을 기억하자. 즉, 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다. Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자. 