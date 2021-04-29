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

Java의 장점 중 하나는, safe language입니다. 이는 메모리 손상 오류에 영향을 받지않음을 의미하며, 이를 통해서 어떤 일이 일어나도 불변성이 유지될 것이라 확신하고 진행할 수 있습니다.

다만, 이 경우에도 코드를 개판... 으로 짜면 문제가 발생할 수 있습니다. 따라서 **클래스의 클라이언트가 위험하게 구성될 수 있다는 가정하에, 방어적으로 프로그래밍해야합니다.**

```java
// Broken "immutable" time period class
public final class Period {
  private final Date start;
  private final Date end;

  public Period(Date start, Date end) {
    if (start.compareTo(end) > 0)
      throw new IllegalArgumentException(
    start + " after " + end);
    this.start = start;
    this.end   = end;
  }

  public Date start() { return start; }

  public Date end() {   return end;   }

  ...    // Remainder omitted
}

public static void main() {
  // Period 인스턴스의 내부를 공격한 경우.
  Date start = new Date();
  Date end = new Date();
  Period p = new Period(start, end);
  end.setYear(78); // p 내부가 수정됩니다.
}
```

이러한 경우처럼, Date가 더이상 사용되지 않으면 이를 새로운 코드에서 사용하면 안됩니다. 따라서 이러한 문제에서 인스턴스 내부를 보호하려면, **생성자에 대한 각 변경 가능한 매개 변수의 방어적 복사본을 만드는 것이 중요합니다.**

```java
// 변경된 생성자, 매개 변수의 방어적 복사본을 만듭니다.
public Period (Date start, Date end) {
  this.start = new Date (start.getTime ());
  this.end = new Date (end.getTime ());

  if (this.start.compareTo (this.end)> 0)
    throw new IllegalArgumentException (
  this.start + "after"+ this.end);
}
```

이렇게 사용하면 위의 문제를 해결할 수 있습니다. 그러나 아래처럼, 데이터를 바꿀 수 도 있습니다.

```java
Date start = new Date ();
Date end = new Date ();
Period p = new Period(start, end);
p.end().setYear(78); // p의 내부를 수정합니다!
```

이를 해결할려면 다음처럼 또 할 수 있습니다.

```java
// 수리 된 접근 자-내부 필드의 방어용 복사본 만들기
public Date start () {
  return new Date (start.getTime ());
}

public Date end () {
  return new Date (end.getTime ());
}
```

이와 같이 새로운 생성자와 새로운 접근자를 사용함을 통해서 방어적 코딩을 할 수 있습니다.

**클라이언트에서 클래스를 가져오거나, 반환하는 경우에 변경 가능한 요소가 있는 경우에는 클래스는 구성 요소를 방어적으로 복사해야합니다. 복사를 할 수 없는 환경이면, 사용하는 클라이언트를 신뢰하는 구조로 가야하면서, 이를 수정하지 않도록 문서화시켜야합니다.**

<br/>

## Item 51. 메서드 이름을 신중하게 설계합니다.

아래의 규칙을 지켜야합니다.

### 메서드 이름을 신중하게 선택해야합니다.

- 이해하기 동일한 패키지의 다른 이름과 일치하는 이름을 선택합니다.
- 광범위한 합의와 일치하는 이름을 선택하는 것이 좋습니다.

### 편리한 방법을 제공하는데 너무 과하게 사용하면 안됩니다.

- 너무 많아지면 이를 사용하고 문서화하고 테스트, 유지하는데 어려워집니다.

### 너무 긴 매개변수는 피합니다.

- 4개 이하의 매개변수를 사용하는 것이 좋습니다.
- 동일한 형식의 매개 변수 시퀀스가 길면 안좋습니다.

이를 해결하는 방법은 다음과 같습니다.

- 메서드를 여러 메서드로 나눕니다.
- 매개 변수 그룹을 보유하는 `helper class`를 만듭니다.
- 메서도 호출까지 Builder 패턴을 적용합니다.

### 매개변수 유형의 경우, 클래스보다 인터페이스를 선호합니다.

매개 변수를 정의하는데 적합한 인터페이스가 있는 경우, 인터페이스를 구현하는 클래스를 대신 사용하는 것이 좋습니다.

### boolean의 의미가 메서드 이름에서 명확하지 않으면,boolean 매개 변수 보다는 요소가 두개인 Enum 형을 쓰는 것이 중요합니다.

열거형을 통해서 코드를 더 쉽게 읽고 쓸 수 있습니다.

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
