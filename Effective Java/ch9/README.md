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

```java
// 컬렉션을 반복하는 것이 가장 좋은 방법은 아닙니다.
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
  Element e = i.next();
  ... // Do something with e
}
```

기존의 for loop 문도, while 문 보다는 낫지만 완벽하지 않습니다. 요소만 필요한 경우, 이는 복잡할 뿐입니다.

for-each 루프는 이러한 문제를 해결합니다.

```java
// 컬렉션 및 배열을 반복하는 데 선호되는 관용구
for (Element e : elements) {
  ... // Do something with e
}
```

이러한 for-each 문은 중첩된 반복문에서 좀 더 도움이 됩니다. 아래는 for문에서 발생하기 쉬운 버그입니다.

```java
enum Face {ONE, TWO, THREE, FOUR, FIVE, SIX}
...
Collection <Face> faces = EnumSet.allOf (Face.class);

for (Iterator <Face> i = faces.iterator (); i.hasNext ();)
  for (Iterator <Face> j = faces.iterator (); j.hasNext ();)
    System.out.println (i. next () + ""+ j.next ());

// Expected Output : {ONE, ONE}, {ONE, TWO}, ..., {SIX, SIX}
// Real Output : {ONE, ONE}, {TWO, TWO}, ..., {SIX, SIX}
```

이를 생각하는 값을 나오게 하기 위해서는 아래처럼 구성하면 됩니다.

```java
// 컬렉션 및 배열에 중첩 된 반복에 대한 기본 관용구
for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit, rank));
```

다만 for-each 문을 사용할 수 없는 경우는 세가지 상황이 있습니다.

- Destructive filtering(파괴적 필터링)
  - 선택한 요소를 제거하는 컬렉션을 탐색해야하는 경우, `remove` 메서드를 호출할 수 잇도록 명시적 반복자를 사용해야합니다.
  - Java 8에 추가된 Collection의 removeIf 메서드를 사용하여 명시적 순회를 피할 수 있습니다.
- Transforming(변형)
  - 목록 또는 배열을 탐색하고 해당 요소 값의 일부 또는 전체를 교체해야하는 경우, 요소 값을 바꾸기 위해서 list iterator 또는 array index가 필요합니다.
- Parallel iteration(병렬 반복)
  - 여러 컬렉션을 병렬로 트래버스해야하는 경우, 모든 반보기 또는 인덱스 변수를 잠금 단계로 진행할 수 있도록 iterator 또는 인덱스 변수를 명시적으로 제어해야합니다.

for-each 루프를 사용하면, 컬렉션과 배열을 반복할 수 있을 뿐만 아니라, 단일 메서드로 구성된 Iterable 인터페이스를 구현하는 모든 개체를 발복할 수 있습니다. 인터페이스의 모습은 다음과 같습니다.

```java
public interface Iterable <E> {
  //이 반복 가능한 요소에 대한 iterator를 반환합니다.
  Iterator <E> iterator ();
}
```

이렇게 하면, 사용자가 for-each 루프를 사용해서 type을 iterator할 수 있습니다. 즉, **for-each 루프는 for 성능 저하없이 명확성, 유연성 및 버그 방지 측면에서 기존 루프에 비해 강력한 이점을 제공합니다.** 또한 최대한 for-each 루프를 사용할 수 있습니다.

<br/>

## Item 59. 라이브러리를 알고 사용해야합니다.

예를 들어, random 을 사용해야할 때 random을 쓰려면 여러 버그가 발생할 수 있습니다. 이러한 경우, 여러 결점이 존재하기 때문에 이를 해결해야합니다.

대표적인 예시로 자바 7의, ThreadLocalRandom이 있습니다. 이를 사용하면, 높은 품질의 난수를 빠르게 생성할 수 있습니다.

표준 라이브러리 사용은 다음의 장점을 가집니다.

- **표준 라이브러리를 사용하면, 이를 작성한 전문가의 지식과 이전에 사용했던 경험을 활용할 수 있습니다.**
- 업무와 관련 없는 부분에 시간을 낭비할 필요가 없습니다.
- 시간이 지남에 따라 성능이 향상되는 경향이 있습니다.
- 시간이 지남에 따라 얻는 경향이 있습니다. (누락된 기능이 후속 추가될 수 있습니다.)

예를 들어 다음과 같이 보여줄 수 있습니다.

```java
// Java 9에 추가 된 transferTo를 사용하여 URL의 내용을 인쇄합니다.
public static void main (String [] args) throws IOException {
  try (InputStream in = new URL (args [0]). openStream ()) {
    in. transferTo (System.out);
  }
}
```

이러한 라이브러리는 문서들을 공부하기에는 너무 많습니다. 그러나, 모든 프로그래머들은 `java.lang`, `java.util`, `java.io`와 서브패키지의 기본 사항에 대해 잘 알고 있어야합니다.

이를 요약하면 다음과 같습니다. **라이브러리가 있는 경우 사용해야하고 모르는 경우에는 라이브러리가 있는지 확인해야합니다.** 일반적으로 라이브러리 코드는 사용자가 직접 작성하는 코드보다 좋으며 시간이 지남에 따라 개선 될 가능성이 높습니다.

<br/>

## Item 60. 정확한 답변이 필요한 경우, `FLOAT`와 `DOUBLE`을 피합니다.

`float`와 `double` 유형은 과학 및 공학 계산을 위해서 설게되었습니다. 그렇기 때문에 정확한 근사치를 신속하게 제공하기 위해서 설계된 구조입니다. 따라서, 정확한 결과에 제공하면 안되며 정확한 결과가 필요한 곳에서는 사용하면 안됩니다. (Ex. 금전 계산 등)

```java
// Broken - 화폐 계산에 부동 소수점을 사용합니다!
// 사용하면 안됩니다.
public static void main(String[] args) {
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
  }

  System.out.println(itemsBought + " items bought.");
  System.out.println("Change: $" + funds);
}
// output : 0.3999999... (잘못된 값)
```

따라서, 아래처럼 수정해야합니다.

```java
public static void main(String[] args) {
  final BigDecimal TEN_CENTS = new BigDecimal(".10");
  int itemsBought = 0;
  BigDecimal funds = new BigDecimal("1.00");
  for (BigDecimal price = TEN_CENTS;
        funds.compareTo(price) >= 0;
        price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
  }
  System.out.println(itemsBought + " items bought.");
  System.out.println("Money left over: $" + funds);
}
```

이 경우, BigDecimal을 사용하게 되면 정확한 결과를 만들 수 있습니다. (다만, 원시적인 값에 비해 조금 더 느려집니다.)

<br/>

## Item 61. Boxed Primitive 보다 Primitive type을 선호합니다.

자바는 `int`, `double`, `boolean과` 같은 기본(Primitive) 요소와 `String`이나 `List`와 같은 참조(Reference) 유형으로 구성되어 있습니다. 또한 모든 기본 유형(Primitive Type)에는 Boxed Primitive라고 하는 참조 유형이 있습니다. 이것이 바로, `int`, `double`, `boolean`에 해당하는 `Integer`, `Double`, `Boolean` 입니다.

Primitive와 Boxed Primitive 사이에는 세 가지 주요 차이점이 있습니다.

- Primitive는 값만 가지고 있는 반면에, Boxed Primitive는 값과 구별되는 ID를 가지고 있습니다.
- Primitive는 기본 값만 존재하는 반면에, Boxed Primitive는 null과 같이 비 기능적 값이 있습니다.
- Primitive는 Boxed Primitive보다 시간과 공간 효율적입니다.

이러한 차이를 참고해서 만들어야 합니다.

즉, 아래의 코드는 잘못된 코드입니다.

```java
Comparator <Integer> naturalOrder = (i, j)-> (i <j)? -1 : (i == j? 0 : 1);

naturalOrder.compare(new Integer(42), new Integer(42))
// output : 1 -> error
```

이와 같은 문제로, boxed primitives에는 `==` 연산자를 적용하는 것은 거의 대부분 잘못된 것입니다. 따라서 비교를 할때는 primitive를 사용하는 것이 더 좋습니다.

```java
public class Unbelievable {
  static Integer i;

  public static void main (String [] args) {
    if (i == 42)
      System.out.println ( "Unbelievable");
  }
}
```

다만, 위의 코드처럼 사용하는 것도 좋지 않습니다. primitive와 boxed primitive를 혼합해서 사용하는 경우, boxed primitive 타입이 박스 해제가 되는 문제가 있습니다.

```java
// 매우 느린 코드
public static void main (String [] args) {
  Long sum = 0L;
  for (long i = 0; i <Integer.MAX_VALUE; i ++) {
    sum + = i;
  }
  System.out.println (sum);
}
```

위 코드는 지역 변수 sum을 기본(Primitive) 타입이 아닌, Boxed Primitive 타입을 사용했기 때문에 반복적으로 boxing되고 unboxed 되는 문제가 존재합니다.

요약하자면, 선택권이 있는 경우에는 Boxed primitive 보다는 primitive를 사용하는 것이 좋습니다. Boxed Primitive를 사용해야하는 상황이면 조심히 사용해야합니다. **Auto boxing은 boxed primitives를 사용하는 위험은 아니지만, 자세한 정도를 줄입니다.**

프로그램이 boxed 및 unboxed primitive 를 포함하는 혼합 계산을 할때는 unboxing을 수행되고, 프로그램이 unboxing을 수행할 때는 `NullPointerException`을 throw할 필요가 있습니다. 프로그램이 Primitive 타입을 Boxed Primitive에 넣으면 비용이 많이 들고 불필요한 개체 생성이 발생할 수 있습니다.

<br/>

## Item 62. 다른 유형이 적합한 문자열은 피합니다.

문자열은 텍스트를 위해 설계되었습니다. 따라서 문자열로 몇가지를 하면 안되는 경우가 있습니다.

### 문자열은 다른 값 타입을 대체하지 못합니다.

- 입력에서 문자열로 받는 경우가 있지만, 숫자인 경우에는 int, float, BigInteger로 변환해야하고 참/거짓의 경우에는 Enum 또는 boolean으로 처리해야합니다.

### 문자열은 Enum 형을 대체하지 못합니다.

- Enum은 문자열보다, Enum형 상수를 사용하는 것이 중요합니다.

### 문자열은 aggregate 타입을 대체하지 못합니다.

- Entity에 여러 구성이 있는 경우, 사용하지 않는 것이 좋습니다.

### 문자열은 capabilities를 대체하지 못합니다.

- 때때로 문자열은 일부 기능에 대한 액세스 권한을 부여하기위해 사용하는데, 스레드 로컬 변수를 사용할 때 문제가 생길 수 있습니다.

```java
// Broken - 문자열을 기능으로 부적절하게 사용했습니다!
// 두 클라이언트가 독립적으로 스레드 로컬을 사용하기로 결정하면 의도하지않게 변수를 공유하므로 여러 문제가 발생가능합니다.
public class ThreadLocal {
  private ThreadLocal() { } // Noninstantiable

  // 명명 된 변수에 대한 현재 스레드의 값을 설정합니다.
  public static void set(String key, Object value);

  // 명명 된 변수에 대한 현재 스레드의 값을 반환합니다.
  public static Object get(String key);
}
```

이를 해결하는 코드는 아래와 같습니다.

```java
public final class ThreadLocal<T> {
  public ThreadLocal();
  public void set(T value);
  public T get();
}
```

이를 요약하면, 더 나인 데이터 유형이 존재하거나 쓸 수 있을 때 객체를 문자열로 나타내는 자연스러운 경향을 피해야합니다. 부적절하게 사용되는 문자열은 다른 유형보다 번거롭고 유연성이 떨어지며 느리고, 오류가 발생하기 쉽습니다.

<br/>

## Item 63. 문자열 연결의 성능에 주의합니다.

문자열 연결 연산자, `+`는 몇개의 문자열을 하나로 결합하는 편리하고 좋은 방법입니다. 작은 범위에서는 좋을 수 있지만, 문자열 연결 연산자를 사용해서 n개의 문자열을 연결하는 경우, n 타임이 걸리게 됩니다.

즉, 아래는 잘못된 사용 코드입니다.

```java
// 부적절한 문자열 연결 사용-성능이 좋지 않습니다!
public String statement() {
  String result = "";
  for (int i = 0; i < numItems(); i++)
    result += lineForItem(i);  // String concatenation
  return result;
}
```

이를 해결하기 위해서는 `StringBuilder`를 사용하는 것이 좋습니다.

```java
public String statement () {
  StringBuilder b = new StringBuilder (numItems () * LINE_WIDTH);
  for (int i = 0; i <numItems (); i ++)
    b.append (lineForItem (i));
  return b.toString ();
}
```

자바 6이후로, 문자열 연결 속도를 높였으나 아직까지는 `StringBuilder`를 사용하는 것이 좋습니다.

즉, 성능이 관련이 없는 경우가 아니면, 문자열 연결 연산자(`+`)를 사용해서 몇개의 문자열을 결합하지 않는 것이 중요합니다.

<br/>

## Item 64. 인터페이스로 객체를 참조합니다.

객체를 참조하려면 클래스보다 인터페이스를 사용을 선호해야합니다. **적절한 인터페이스 유형이 있는 경우, 매개 변수, 반환 값, 변수 및 필드는 모두 인터페이스 유형을 사용하여 선언해야합니다.**

즉, 아래처럼 작성하는 것이 중요합니다.

```java
// Good Case - 인터페이스를 유형으로 사용
Set<Son> sonSet = new LinkedHashSet<>();

// Bad Case - 클래스를 유형으로 사용한 것
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

인터페이스를 유형으로 사용하게 되면 프로그램이 좀 더 **유연**해집니다. 다만, 적절한 인터페이스가 없는 경우에는 인터페이스가 아닌 클래스에서 객체를 참조하는 것이 전적으로 중요합니다.

따라서, 인터페이스를 사용할 수 있다면 인터페이스를 사용해서 객체를 참조시켜 프로그램이 더 유연하고 세련되게 구성합니다. 적절한 인터페이스가 없으면 필요한 기능을 제공하는 클래스 계층 구조에서 가장 덜 구체적인 클래스를 사용하는 것이 중요합니다.

<br/>

## Item 65. 리플렉션보다 인터페이스를 선호합니다.

- [Reflection ?](https://velog.io/@ptm0304/Java-%EC%9E%90%EB%B0%94-%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98)

<br/>

## Item 66. 네이티브 메서드를 신중하게 사용합니다.

<br/>

## Item 67. 신중하게 최적화합니다.

<br/>

## Item 68. 일반적으로 허용되는 명명 규칙을 준수합니다.
