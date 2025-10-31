
<div align="center">

# Deep Dive  
Deep Dive into REffective-Java

--- 
</div>
 
## Items

### 2장. 객체의 생성과 파괴 
1. [생성자 대신 정적 팩토리 메서드를 고려하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-01/%EC%83%9D%EC%84%B1%EC%9E%90%20%EB%8C%80%EC%8B%A0%20%EC%A0%95%EC%A0%81%20%ED%8C%A9%ED%84%B0%EB%A6%AC%20%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md)
2. [생성자에 매개변수가 많다면 빌더를 고려하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-02/%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90%20%EB%A7%A4%EA%B0%9C%EB%B3%80%EC%88%98%EA%B0%80%20%EB%A7%8E%EB%8B%A4%EB%A9%B4%20%EB%B9%8C%EB%8D%94%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md)
3. [private 생성자나 열거 타입으로 싱글턴을 보증하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-03/private%20%EC%83%9D%EC%84%B1%EC%9E%90%EB%82%98%20%EC%97%B4%EA%B1%B0%20%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C%20%EC%8B%B1%EA%B8%80%ED%84%B4%EC%9D%84%20%EB%B3%B4%EC%A6%9D%ED%95%98%EB%9D%BC.md)
4. [인스턴스화를 막으려거든 private 생성자를 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/tree/main/Ch%202/item-04)
5. [자원을 직접 명시하지 말고 의존 객체 주입을 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/tree/main/Ch%202/item-05)
6. [불필요한 객체 생성을 피하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-06/%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C%20%EA%B0%9D%EC%B2%B4%20%EC%83%9D%EC%84%B1%EC%9D%84%20%ED%94%BC%ED%95%98%EB%9D%BC.md)
7. [다 쓴 객체 참조를 해제하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-07/%EB%8B%A4%20%EC%93%B4%20%EA%B0%9D%EC%B2%B4%20%EC%B0%B8%EC%A1%B0%EB%A5%BC%20%ED%95%B4%EC%A0%9C%ED%95%98%EB%9D%BC.md)
8. [finalizer와 cleaner 사용을 피하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-08/finalizer%EC%99%80%20cleaner%20%EC%82%AC%EC%9A%A9%EC%9D%84%20%ED%94%BC%ED%95%98%EB%9D%BC.md)
9. [try-finally보다는 try-with-resources를 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/tree/main/Ch%202/item-09)

### 3장 모든 객체의 공통 메서드 
10. [equals는 일반 규약을 지켜 재정의하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/tree/main/Ch%203/item-10)
11. [equals를 재정의하면 hashCode도 재정의하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%202/item-11/equals%EB%A5%BC%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%A0%A4%EA%B1%B0%EB%93%A0%20hashCode%EB%8F%84%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC.md)
12. [toString을 항상 재정의하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%203/item-12/toString%EC%9D%84%20%ED%95%AD%EC%83%81%20%EC%9E%AC%EC%A0%95%EC%9D%98%ED%95%98%EB%9D%BC.md)
13. [clone 재정의는 주의해서 진행하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%203/item-13/clone%20%EC%9E%AC%EC%A0%95%EC%9D%98%EB%8A%94%20%EC%A3%BC%EC%9D%98%ED%95%B4%EC%84%9C%20%EC%A7%84%ED%96%89%ED%95%98%EB%9D%BC.md)
14. [Comparable을 구현할지 고려하라- 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%203/item-14/Comparable%EC%9D%84%20%EA%B5%AC%ED%98%84%ED%95%A0%EC%A7%80%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md)

### 4장 클래스와 인터페이스
15. [클래스와 멤버의 접근 권한을 최소화하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/tree/main/Ch%204/item-15)
16. [public 클래스에는 public 필드 대신 접근자 메서드를 사용하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-16/public%20%ED%81%B4%EB%9E%98%EC%8A%A4%EC%97%90%EC%84%9C%EB%8A%94%20public%20%ED%95%84%EB%93%9C%EA%B0%80%20%EC%95%84%EB%8B%8C%20%EC%A0%91%EA%B7%BC%EC%9E%90%20%EB%A9%94%EC%84%9C%EB%93%9C%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md) 
17. [변경 가능성을 최소화하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/tree/main/Ch%204/item-17)
18. [상속보다는 컴포지션을 사용하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-18/%EC%83%81%EC%86%8D%EB%B3%B4%EB%8B%A4%EB%8A%94%20%EC%BB%B4%ED%8F%AC%EC%A7%80%EC%85%98%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
19. [상속을 고려해 설계하고 문서화하라. 그렇지 않았다면 상속을 금지하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-19/%EC%83%81%EC%86%8D%EC%9D%84%20%EA%B3%A0%EB%A0%A4%ED%95%B4%20%EC%84%A4%EA%B3%84%ED%95%98%EA%B3%A0%20%EB%AC%B8%EC%84%9C%ED%99%94%ED%95%98%EB%9D%BC.%20%EA%B7%B8%EB%9F%AC%EC%A7%80%20%EC%95%8A%EC%95%98%EB%8B%A4%EB%A9%B4%20%EC%83%81%EC%86%8D%EC%9D%84%20%EA%B8%88%EC%A7%80%ED%95%B4%EB%9D%BC.md)
20. [추상 클래스보다 인터페이스를 우선하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-20/%EC%B6%94%EC%83%81%20%ED%81%B4%EB%9E%98%EC%8A%A4%20%EB%B3%B4%EB%8B%A4%EB%8A%94%20%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%A5%BC%20%EC%9A%B0%EC%84%A0%ED%95%98%EB%9D%BC.md)
21. [인터페이스는 구현하는 쪽을 생각해 설계하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-21/인터페이스는%20구현하는%20쪽을%20생각해%20설계하라.md)
22. [인터페이스는 타입을 정의하는 용도로만 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-22/%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%8A%94%20%ED%83%80%EC%9E%85%EC%9D%84%20%EC%A0%95%EC%9D%98%ED%95%98%EB%8A%94%20%EC%9A%A9%EB%8F%84%EB%A1%9C%EB%A7%8C%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
23. [태그 달린 클래스보다 클래스 계층구조를 사용하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-23/%ED%83%9C%EA%B7%B8%20%EB%8B%AC%EB%A6%B0%20%ED%81%B4%EB%9E%98%EC%8A%A4%EB%B3%B4%EB%8B%A4%EB%8A%94%20%ED%81%B4%EB%9E%98%EC%8A%A4%20%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0%EB%A5%BC%20%ED%99%9C%EC%9A%A9%ED%95%98%EB%9D%BC.md)
24. [멤버 클래스는 되도록 static으로 만들라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-24/멤버%20클래스는%20되도록%20static으로%20만들라.md)
25. [톱레벨 클래스는 한 파일에 하나만 담아라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%204/item-25/%ED%86%B1%EB%A0%88%EB%B2%A8%20%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94%20%ED%95%9C%20%ED%8C%8C%EC%9D%BC%EC%97%90%20%ED%95%98%EB%82%98%EB%A7%8C%20%EB%8B%B4%EC%9C%BC%EB%9D%BC.md)

### 5장 제네릭
26. [로 타입은 사용하지말라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-26/%EB%A1%9C%20%ED%83%80%EC%9E%85%EC%9D%80%20%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80%20%EB%A7%90%EB%9D%BC.md)
27. [비검사 경고를 제거하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-27/비검사%20경고를%20제거하라.md)
28. [배열보다 리스트를 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-28/%EB%B0%B0%EC%97%B4%EB%B3%B4%EB%8B%A4%EB%8A%94%20%EB%A6%AC%EC%8A%A4%ED%8A%B8%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
29. [이왕이면 제네릭 타입으로 만들라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-29/%EC%9D%B4%EC%99%95%EC%9D%B4%EB%A9%B4%20%EC%A0%9C%EB%84%A4%EB%A6%AD%20%ED%83%80%EC%9E%85%EC%9C%BC%EB%A1%9C%20%EB%A7%8C%EB%93%A4%EB%9D%BC.md) 
30. [이왕이면 제네릭 메서드로 만들라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-30/이왕이면%20제네릭%20메서드로%20만들라.md)
31. [한정적 와일드카드로 API 유연성을 높여라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-31/%ED%95%9C%EC%A0%95%EC%A0%81%20%EC%99%80%EC%9D%BC%EB%93%9C%EC%B9%B4%EB%93%9C%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%B4%20API%20%EC%9C%A0%EC%97%B0%EC%84%B1%EC%9D%84%20%EB%86%92%EC%9D%B4%EB%9D%BC.md)
32. [제네릭과 가변인수는 함께 쓸 때 신중하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-32/제네릭과%20가변인수를%20함께%20쓸%20때는%20신중하라.md)
33. [타입 안전 이종 컨테이너를 고려하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%205/item-33/%ED%83%80%EC%9E%85%20%EC%95%88%EC%A0%84%20%EC%9D%B4%EC%A2%85%20%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%A5%BC%20%EA%B3%A0%EB%A0%A4%ED%95%98%EB%9D%BC.md)

### 6장 열거 타입과 애노테이션
34. [int 상수 대신 열거 타입을 사용하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-34/int%20%EC%83%81%EC%88%98%20%EB%8C%80%EC%8B%A0%20%EC%97%B4%EA%B1%B0%20%ED%83%80%EC%9E%85%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
35. [ordinal 메서드 대신 인스턴스 필드를 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-36/%EB%B9%84%ED%8A%B8%20%ED%95%84%EB%93%9C%20%EB%8C%80%EC%8B%A0%20EnumSet%20%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
36. [비트 필드 대신 EnumSet을 사용하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-36/비트%20필드%20대신%20EnumSet%20을%20사용하라.md)
37. [ordinal 인덱싱 대신 EnumMap을 사용하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-37/ordinal%20%EC%9D%B8%EB%8D%B1%EC%8B%B1%20%EB%8C%80%EC%8B%A0%20EnumMap%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
38. [확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-38/%ED%99%95%EC%9E%A5%ED%95%A0%20%EC%88%98%20%EC%9E%88%EB%8A%94%20%EC%97%B4%EA%B1%B0%20%ED%83%80%EC%9E%85%EC%9D%B4%20%ED%95%84%EC%9A%94%ED%95%98%EB%A9%B4%20%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
39. [명명 패턴보다 애노테이션을 사용하라 -현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-39/명명%20패턴보다%20애너테이션을%20사용하라.md)
40. [@Override 애너테이션을 일관되게 사용하라 - 민우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-40/%40Override%20%EC%95%A0%EB%84%88%ED%85%8C%EC%9D%B4%EC%85%98%EC%9D%84%20%EC%9D%BC%EA%B4%80%EB%90%98%EA%B2%8C%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)
41. [정의하려는 것이 타입이면 마커 인터페이스를 사용하라 - 장우](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%206/item-41/%EC%A0%95%EC%9D%98%ED%95%98%EB%A0%A4%EB%8A%94%20%EA%B2%83%EC%9D%B4%20%ED%83%80%EC%9E%85%EC%9D%B4%EB%9D%BC%EB%A9%B4%20%EB%A7%88%EC%BB%A4%20%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%9D%BC.md)

### 7장 람다와 스트림
42. [익명 클래스보다 람다를 사용하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%207/item-42/익명%20클래스보다는%20람다를%20사용하라.md)
43. [람다보다 메서드 참조를 사용하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%207/item-43/람다보다%20메서드%20참조를%20사용하라.md)
44. 표준 함수형 인터페이스를 사용하라
45. 스트림은 주의해서 사용하라
46. [스트림에서는 부작용 없는 함수를 사용하라 - 현재](https://github.com/BackEndDeepDive/Effective-Java/blob/main/Ch%207/item-46/스트림에서는%20부작용%20없는%20함수를%20사용하라.md)
47. 반환 타입으로는 스트림보다 컬렉션이 낫다
48. 스트림 병렬화는 주의해서 적용하라

### 8장 메서드
49. 매개변수 유효성을 검사하라
50. 적시에 방어적 복사본을 만들라
51. 메서드 시그니처를 신중히 설계하라
52. 다중정의는 신중히 사용하라
53. 가변인수는 신중히 사용하라
54. null이 아닌, 빈 컬렉션이나 배열을 반환하라
55. 옵셔널 반환은 신중히 하라
56. 공개 API 요소에는 항상 문서화 주석을 작성하라

### 9장 일반적인 프로그래밍 원칙
57. 지역변수의 범위를 최소화하라
58. 전통적인 for 문보다 for-each 문을 사용하라
59. 라이브러리를 익히고 사용하라
60. 정확한 계산에는 float와 double을 피하라
61. 박싱된 기본 타입보다 기본 타입을 사용하라
62. 다른 타입이 적절하다면 문자열 사용을 피하라
63. 문자열 연결은 느리니 주의하라
64. 객체는 인터페이스로 참조하라
65. 리플렉션보다 인터페이스를 사용하라
66. 네이티브 메서드는 신중히 사용하라
67. 최적화는 신중히 하라
68. 일반적으로 통용되는 명명 규칙을 따르라

### 10장 예외
69. 예외는 진짜 예외 상황에만 사용하라
70. 복구 가능한 상황엔 검사 예외를, 프로그래밍 오류엔 런타임 예외를 사용하라
71. 불필요한 검사 예외 사용을 피하라
72. 표준 예외를 사용하라
73. 추상화 수준에 맞는 예외를 던져라
74. 메서드가 던지는 모든 예외를 문서화하라
75. 예외 메시지에 실패 관련 정보를 담아라
76. 가능한 한 실패 원자적으로 만들라
77. 예외를 무시하지 말라

### 11장 동시성
78. 공유 중인 가변 데이터는 동기화해 사용하라
79. 과도한 동기화는 피하라
80. 스레드보다 실행자, 태스크, 스트림을 애용하라
81. wait와 notify보다 동시성 유틸리티를 애용하라
82. 스레드 안전성 수준을 문서화하라
83. 지연 초기화는 신중히 사용하라
84. 프로그램 동작을 스레드 스케줄러에 기대지 말라

### 12장 직렬화
85. 자바 직렬화의 대안을 찾으라
86. Serializable 구현 여부를 신중히 결정하라
87. 커스텀 직렬화 형태를 고려하라
88. readObject 메서드는 방어적으로 작성하라
89. 인스턴스 수를 통제해야 한다면 readResolve보다 열거 타입을 사용하라
90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라


## Conventions
1. 본인이 정리한 부분은 본인의 이름 브랜치에 기록한다.
    - 브랜치는 영어(소문자)로 설정

    ```
    main(default)
    choijangwoo
    yukminwoo
    kimjaehyun
    kimhyunjae
    ```

2. 각 주차 **세미나** 부분의 개인 링크를 수정할 때는 main에서 브랜치를 분기하여 readme를 수정 후 PR을 작성한 뒤 분기한 브랜치를 삭제한다.

    - 담당한 아이템의 부분을 다음과 같이 수정한다.
    ```
   [item명 - 이름](readme 링크)
    ```

---

## Contributors

|<img src="https://github.com/choijw1004.png" width="150" height="150"/>|<img src="https://github.com/FickleBoBo.png" width="150" height="150"/>|<img src="https://github.com/Kguswo.png" width="150" height="150"/>|
|:-:|:-:|:-:|
|최장우<br/>[@choijw1004](https://github.com/choijw1004)|육민우<br/>[@minwoo_Yuk](https://github.com/FickleBoBo)|김현재<br/>[@Kguswo](https://github.com/Kguswo)|
