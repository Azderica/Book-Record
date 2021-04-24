# Lambdas and Streams

Java 8에서 함수 객체를 더 쉽게 만들 수 있도록, `functional interface`, 람다 및 메소드 참조가 추가되었습니다. 스트림 API는 이러한 언어 변경과 함께 추가되어 데이터 요소들의 시퀀스 처리를 위한 라이브러리 지원을 제공합니다.

## Item 42. 익명 클래스보다 람다를 선택합니다.

과거에는 하나의 추상 메소드를 가진 인터페이스가 function types로 사용되었습니다.

```java
// 함수 객체로서의 익명 클래스 인스턴스 - 폐기되었음
Collections.sort(words, new Comparator<String>() {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.length(), s2.length());
  }
});
```

이러한 익명 클래스는 object-oriented 객체를 필요로 하는 고전적인 객체 지향 디자인 패턴(특히. Strategy pattern)에 적합했습니다. 그러나 이는 매우 애매모호했기 때문에 이를 대체할 수단이 필요하게 되었습니다.

Java 8에서는 단일 추성 메서드와의 인터페이스가 필요하다는 개념을 공식화하였고, 이를 `function interface`라고 알려져 있으며, 람다식이나 람다를 통해서 이를 만들 수 있게 되었습니다.

```java
// 함수 객체로서의 람다 표현식 (익명 클래스 대체)
Collections.sort (words,
    (s1, s2)-> Integer.compare (s1.length (), s2.length () ));
```

프로그램이 명확해지지 않는 한, 모든 람다 파라미터의 유형을 생략하는 이 좋습니다. (컴파일러가 이를 유추할 수 없기 때문에 그렇습니다.)

앞선 주제에서 말했듯이 Raw 타입을 사용하지 않는 것이 여기서 중요합니다.

또한 이러한 람다식을 통해서 더 짧게 만들 수 있고, 의미적으로도 잘 보일 수 있습니다.

```java
// Before 코드
Collections.sort(words, comparingInt(String::length));

// After 코드
words.sort(comparingInt(String::length));

// 뒤의 코드가 앞의 코드보다 직관적입니다.
```

이전 챕터에서 나왔던 Enum형을 아래처럼 수정할 수도 있습니다.

```java
public enum Operation {
  PLUS  ("+", (x, y) -> x + y),
  MINUS ("-", (x, y) -> x - y),
  TIMES ("*", (x, y) -> x * y),
  DIVIDE("/", (x, y) -> x / y);

  private final String symbol;
  private final DoubleBinaryOperator op;

  Operation(String symbol, DoubleBinaryOperator op) {
    this.symbol = symbol;
    this.op = op;
  }

  @Override public String toString() { return symbol; }

  public double apply(double x, double y) {
    return op.applyAsDouble(x, y);
  }
}
```

그러나, 이러한 **람다는 name과 document가 없기 때문에 계산이 확실하지 않거나, 몇줄을 초과하는 경우에는 람다에 넣는 것은 좋지않습니다.** (일반적으로 1줄 ~ 3줄 사이가 적합)

현재는 람다의 등장으로 익명 클래스를 거의 사용하지 않지만, 익명 클래스만 할 수 있는 부분이 있습니다.

- 람다는 기능 인터페이스로 제한되나, 추상 클래스의 인스턴스를 만들려면 람다가 아닌 익명 클래스로 만들 수 있습니다.
- 익명 클래스를 사용해서 여러 추상 메서드가 있는 인터페이스의 인스턴스를 만들 수 있습니다.
- 람다는 자신에 대한 참조를 얻을 수 없습니다. 즉, 본문 내에서 함수 객체에 액세스해야하는 경우 익명 클래스를 사용해야합니다.

람다는 구현 중에 안정적으로 직렬화 및 역 직렬화 할 수 없는 속성은 익명 클래스와 공유합니다. 따라서 람다나 익명 클래스 인스턴스를 직렬화하는 경우는 거의 없습니다. Comparator 등의 직렬화를 쓸때는 nested class를 사용하는 것이 좋습니다.

이를 요약하면 Java 8에서 람다는 작은 함수 객체를 표현하는 가장 좋은 방법입니다. **기능 인터페이스가 아닌 유형의 인스턴스를 만들어야하는 경우가 아니면, 함수 개체에 익명 클래스를 사용하면 안됩니다.**

<br/>

## Item 43. 람다보다 메서드 참조를 선택합니다.

<br/>

## Item 44. 표준 기능 인터페이스를 선택합니다.

<br/>

## Item 45. 스트림을 신중하게 사용합니다.

<br/>

## Item 46. 스트림에서 부작용이 없는 함수를 선택합니다.

<br/>

## Item 47. Return 타입으로 스트림보다는 컬렉션을 선택합니다.

<br/>

## Item 48. 스트림을 병렬로 만들 때 주의합니다.
