# Methods

이 챔터에서는 메서드 디자인의 여러 측면에 대해 이야기합니다. (어떻게 파라미터를 처리하고, 값을 리턴하는지, 메서드 서명을 어떻게 디자인하는 지, 메서드를 어떻게 문서화하는지) 이러한 대부분의 자료들은 생성자와 메서드에에 적용됩니다. 특히, 유용성, 견고성 및 유연성에 중점을 둡니다.

## Item 49. 매개 변수의 유효성을 확인합니다.

대부분의 메서드와 생성자는 매개 변수에 전달할 수 있는 값에 대한 몇가지 제한이 있습니다. 그렇기 때문에, 특정 실패에 대해 예외처리를 해줘야합니다.

그러나 Java 7에서 추가된 `Object.requireNonNull`처럼, 유연하고 편리한 방법을 통해서 null 검사 등을 수동으로 할 필요가 없게 되었습니다.

```java
// Java의 null 검사 기능 인라인 사용
this.strategy = Objects.requireNonNull (strategy, "strategy");
```

그 이후로, Java 9에서는 범위 검사 기능이 `java.util.Objects`에 추가되었으며, 이러한 방법은 checkFromIndexSize, checkFromToIndex, checkIndex 등을 사용할 수 있습니다.

nonpublic method는 `assertions`을 사용해서 매개변수를 확인할 수 있습니다.

```java
// 재귀 적 정렬을 위한 private helper function
private static void sort(long a[], int offset, int length) {
  assert a != null;
  assert offset >= 0 && offset <= a.length;
  assert length >= 0 && length <= a.length - offset;
  ... // Do the computation
}
```

`assert`의 기본 원리는 패키지가 클라이언트에 의해 사용되는 방식에 관계없이 asserted condition이 true라는 주장으로 진행됩니다. 그렇기 때문에, 일반 유효성 검사와 달리 assertions은 만약 실패할시, `AssertionError`가 발생합니다.

- [Assertions 공식 문서](https://docs.oracle.com/javase/8/docs/technotes/guides/language/assert.html)

이외에도 여러 조건에 따라 확인할 수 있는 부분이 있고, 확인할 수 없는 부분이 있습니다.

결론적으로는 **메소드나 생성자를 작성할 때마다, 매개 변수에 어떤 제한이 있는지를 생각**해야합니다. 이러한 제한 사항을 문서화하여야하며, method body의 시작 부분에 명시적인 검사를 적용해야하며, 이러한 습관을 가지고 있어야합니다.

<br/>

## Item 50. 필요할 때, 방어적 사본을 생성합니다.

<br/>

## Item 51. 메서드 서명을 신중하게 설계합니다.

<br/>

## Item 52. 오버로딩을 신중하게 사용합니다.

<br/>

## Item 53. Varargs를 신중하게 사용합니다.

<br/>

## Item 54. Null이 아닌 빈 컬렉션이나 배열을 반환합니다.

<br/>

## Item 55. Optionals를 신중하게 반환합니다.

<br/>

## Item 56. 노출된 모든 API 요소에 대한 문서 주석을 작성합니다.
