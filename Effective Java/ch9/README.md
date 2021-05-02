# General Programming

이번 챕터는 언어의 이론이나 관념이 아닌 실제적인 사실들에 대해 정리합니다. 지역 변수, 제어 구조, 라이브러리, 데이터 유형 및 두가지의 언어 외 기능(`reflection`와 `native method`)에 대해 설명합니다. 그리고 최적화 및 명명 규칙에 대해 정리합니다.

## Item 57. 지역 변수의 범위를 최소화합니다.

[클래스 및 멤버의 접근성 최소화](https://github.com/Azderica/Book-Record/tree/master/Effective%20Java/ch4#item-15-%ED%81%B4%EB%9E%98%EC%8A%A4-%EB%B0%8F-%EB%A9%A4%EB%B2%84%EC%9D%98-%EC%A0%91%EA%B7%BC%EC%84%B1%EC%9D%84-%EC%B5%9C%EC%86%8C%ED%99%94%ED%95%A9%EB%8B%88%EB%8B%A4)의 내용과 비슷하며, 지역 변수의 점위를 최소화함으로써 **코드의 가독성과 유지 관리성을 높이고 오류 가능성을 줄일 수 있습니다.**

지역 변수의 범위를 최소화하는 가장 강력한 기술은 처음 사용되는 위치에 선언하는 것입니다. 변수를 사용하기 전에 선언하면 프로그램이 무엇을 하려는지 어려워집니다.

거의 모든 지역 변수 선언에는 이니셜라이저가 필요합니다. 이가 없다면, 선언할 때까지 선언을 연기해야합니다.

대표적으로 루프는 변수의 범위를 최소화할 수 있는 기능을 제공합니다. 또한 while 루프 보다는 for 루프를 사용하는 것이 좋습니다.

```java
// 컬렉션 또는 배열을 반복하는 데 선호되는 관용구
for (Element e : c) {
  ... // Do Something with e
}

// 반복자가 필요한 경우
for (Iterator <Element> i = c.iterator (); i.hasNext ();) {
  Element e = i.next ();
  ... // Do something with e and i
}
```

일반적으로 for 루프를 선호하는 이유는 while 루프를 잘 못 사용하면 버그가 발생하기 쉽기 때문입니다.

```java
// 잘못된 결과를 만들기 쉬움
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
  doSomething(i.next());
}
...

Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) { // BUG!
  doSomethingElse(i2.next());
}
```

```java
// 컴파일시, 에러가 바로 나오게 됩니다.
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // Do something with e and i
}
...

// Compile-time error - cannot find symbol i
for (Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
  Element e2 = i2.next();
  ... // Do something with e2 and i2
}
```

이렇기 때문에 for loop를 좀 더 선호하는 것이 좋습니다.

또는, 아채처럼 지역 변수의 범위를 최소화할 수 있습니다.

```java
for (int i = 0, n = expensiveComputation(); i < n; i++) {
  ... // Do something with i;
}
```

이는, i와 n 모두가 loop 내에서만 범위를 가지고 있습니다.

마지막 기술은, 지역변수의 범위를 최소화하는 마지막 기술은 메서드를 작고 집중적으로 유지하는 것이 좋습니다. 동일한 방법으로 여러 동작을 수행하면 지역변수가 다른 코드 범위에 있을 수 있기 때문에 이러한 일이 발생하지 않도록 하는 것이 좋습니다.

<br/>

## Item 58. 전통적인 `FOR` 루프 보다는 `FOR-EACH` 루프를 더 선호합니다.

<br/>

## Item 59. 라이브러리를 알고 사용해야합니다.

<br/>

## Item 60. 정확한 답변이 필요한 경우, `FLOAT`와 `DOUBLE`을 피합니다.

<br/>

## Item 61. Boxed Primitive 보다 Primitive type을 선호합니다.

<br/>

## Item 62. 다른 유형이 적합한 문자열은 피합니다.

<br/>

## Item 63. 문자열 연결의 성능에 주의합니다.

<br/>

## Item 64. 인터페이스로 객체를 참조합니다.

<br/>

## Item 65. 리플렉션보다 인터페이스를 선호합니다.

- [Reflection ?](https://velog.io/@ptm0304/Java-%EC%9E%90%EB%B0%94-%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98)

<br/>

## Item 66. 네이티브 메서드를 신중하게 사용합니다.

<br/>

## Item 67. 신중하게 최적화합니다.

<br/>

## Item 68. 일반적으로 허용되는 명명 규칙을 준수합니다.
