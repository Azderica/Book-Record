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

익명 클래스에 비해 람다의 주요 장점은 더 간결한 것입니다. 그러나 자바에서는 메서드 참조를 하는 람다보다 더 간결한 함수 객체를 생성하는 방법이 존재합니다.

```java
// 람다식을 사용한 경우.
map.merge (key, 1, (count, incr)-> count + incr);

// 더 짧은 코드, 메서드 참조
map.merge (key, 1, Integer::sum);
```

메서드에 파라미터가 많을수록 메서드 참조로 제거할 수 있는 상용구가 많아집니다. 다음은 그에 대한 주의사항입니다.

- 람다로 할 수 없는 경우, 메서드 참조로도 할 수 있는 방법은 없습니다.
- IDE로 프로그래밍하는 경우, 가능한 경우에 한해 람다를 메서드 참조로 대체할 수 있습니다.

그러나 항상 이가 옳지는 않습니다. 특히 메서드가 같은 클래스에 있을 때 자주 발생합니다.

```java
// 메서드 참조
service.execute (GoshThisClassNameIsHumongous :: action);

// 람다식 사용 (여기서는 이게 더 좋음.)
service.execute (()-> action ());
```

메서드 참조의 종류는 다음과 같습니다.

| Method Ref Type   | Example                  | Lambda Equivalent                                        |
| ----------------- | ------------------------ | -------------------------------------------------------- |
| Static            | `Integer::parseInt`      | `str -> Integer.parseInt(str)`                           |
| Bound             | `Instant.now()::isAfter` | `Instant then = Instant.now();`<br/>`t->then.isAfter(t)` |
| UnBound           | `String::toLowerCase`    | `str -> str.toLowerCase()`                               |
| Class Constructor | `TreeMap<K, V>::new`     | `() -> new TreeMap<K, V>`                                |
| Array Constructor | `int[]::new`             | `len -> new int[len]`                                    |

이를 정리하면 메서드 참조는 종종 람다보다 간결한 대안을 제공합니다. 따라서, **메소드 참조가 더 짧고 명확한 경우에는 이를 사용하고 그렇지 않은 경우에는 람다를 사용하는 것이 중요합니다.**

<br/>

## Item 44. 표준 기능 인터페이스를 선택합니다.

Java에 람다가 들어오고 난 이후, API 작성 가이드가 변경되었습니다.

대표적으로 Map을 쓰는 경우에는 LinkedHashMap 등을 사용하는 것이 좋습니다. 예를 들어 Map에서 removeEldestEntry를 사용한다고 했을 때, LinkedHashMap 의 경우에는 삭제할 수 있지만 Map의 경우에는 수동적으로 생성해야합니다.

```java
// 불필요한 기능 인터페이스; 대신 표준을 사용하십시오.
@FunctionalInterface interface EldestEntryRemovalFunction <K, V> {
  boolean remove (Map <K, V> map, Map.Entry <K, V> eldest);
}
```

즉, 표준 기능 인터페이스의 기능을 사용하는 경우, **특수 목적으로 만든 인터페이스보다는 표준 기능 인터페이스를 사용하는 것이 중요합니다.**

아래는 기본적인 기능 인터페이스입니다.

| Interface           | Function Signature     | Example               |
| ------------------- | ---------------------- | --------------------- |
| `UnaryOperator<T>`  | `T apply (T t)`        | `String::toLowerCase` |
| `BinaryOperator<T>` | `T apply (T t1, T t2)` | `BigInteger::add`     |
| `Predicate<T>`      | `boolean test (T t)`   | `Collection::isEmpty` |
| `Function<T>`       | `R apply (T t)`        | `Arrays::asList`      |
| `Supplier<T>`       | `T get ()`             | `Instant::now`        |
| `Consumer<T>`       | `void accept (T t)`    | `System.out::println` |

이를 사용해서 여러 변형 케이스로 만들 수도 있습니다. 이를 사용하는 여러 변형 케이스가 존재하지만, 대부분의 표준 기능 인터페이스는 기본 유형에 대한 지원을 위해 존재합니다.

즉, 기본 기능 인터페이스 대신 다른 요소(boxed primitives)가 있는 인터페이스를 사용하는 것은 좋지않습니다.

목적에 맞는 **인터페이스가 필요한 경우**에는 아래의 조건인 경우인지를 잘 생각해봐야합니다.

- 일반적으로 사용되며 설명이 포함된 이름이 도움이 될 수 있는 경우.
- 관련된 강력한 결합(contract)가 있는 경우.
- 사용자 커스텀 메소드가 이점을 가지고 있는 경우.

다만 이러한 경우에는 신중하게 설계가 필요합니다. 그리고, `@FunctionalInterface`와 같이 기능적 인터페이스에는 어노테이션을 추가해야합니다.

요약하면, Java에서는 람다가 있기 때문에 이를 생각하고 API를 생계하는 것이 중요합니다.

<br/>

## Item 45. 스트림을 신중하게 사용합니다.

Stream API는 대량 작업을 순차적으로 또는 병렬적으로 수행하는 작업을 쉽게하기 위해서 Java 8에 추가되었습니다.

Stream API는 다음의 특징을 가집니다.

- Stream pipeline은 source stream과 0개 이상의 intermediate operation, 하나의 terminal operation으로 나눠집니다.
- Stream pipeline은 lazily하게 평가됩니다. (호출될 때까지 시작되지 않으며, 필요없는 데이터 요소는 계산되지 않습니다.)
- Stream API는 유연합니다. (모든 호출이 단일 표현식으로 연결 가능합니다.)
- Stream pipeline은 순차적으로 실행됩니다.
- Stream API는 다재다능하지만, 항상 이렇게 해야하지는 않습니다. (잘못 사용하면, 유지보수성이 떨어집니다.)

Stream을 남용한 코드는 다음과 같습니다.

```java
// Overuse of streams - don't do this!
public class Anagrams {
  public static void main(String[] args) throws IOException {
    Path dictionary = Paths.get(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try (Stream<String> words = Files.lines(dictionary)) {
      words.collect(groupingBy(word -> word.chars()
        .sorted()
        .collect(StringBuilder::new, (sb, c) -> sb.append((char) c),
          StringBuilder::append).toString()))
        .values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .map(group -> group.size() + ": " + group)
        .forEach(System.out::println);
    }
  }
}
```

이 경우, 프로그램을 읽고 유지하기가 매우 어렵습니다. Stream을 잘사용한 케이스는 다음과 같습니다.

```java
// 세련된 스트림 사용으로 명확성과 간결함 향상
public class Anagrams {
  public static void main(String[] args) throws IOException {
    Path dictionary = Paths.get(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try (Stream<String> words = Files.lines(dictionary)) {
      words
        .collect(groupingBy(word -> alphabetize(word)))
        .values().stream()
        .filter(group -> group.size() >= minGroupSize)
        .forEach(g -> System.out.println(g.size() + ": " + g));
      }
  }
// alphabetize method is the same as in original version
...
}
```

이와 같이 표현하면, 프로그램을 이해하기 어렵지 않습니다. 그리고, 람다 매개 변수의 이름도 신중하게 선정해야합니다. 명시적 유형이 없기 때문에, 스트림 파이프파린의 가독성을 높이기 위해서라도 이름 선정은 중요한 역할을 가지고 있습니다.

또한, **파이프 라인에서는 명시적인 유형 정보와 임시 변수가 없기 때문에 도우미 메서드를 사용하는 것은 반복 코드보다 스트림 파이프라인에서 가독성을 위해 매우 중요합니다.**

예를 들어, 아래 코드는 이러한 스트림을 잘못 사용한 케이스입니다.

```java
"Hello world!".chars().forEach(System.out :: print);
// output : 721011081081113211911111410810033 (type을 모르므로)

"Hello world!".chars().forEach(x-> System.out.print ((char) x));
// 수정한 케이스, but 이렇게 사용하는 것은 좋지않음
```

이와 같이 스트림 파이프라인에서 사용할 수 없는 몇가지 작업이 있습니다.

- 코드 블록에서는 범위의 모든 지역 변수를 읽거나 수정이 가능하지만, 람다에서는 최종만 읽을 수 있기때문에 지역변수를 수정할 수 없습니다.
- 코드 블록 return에서 둘러싸는 메서드 break, continue, 예외 throw 등이 가능하지만 람다에서는 불가능합니다.

그러나 스트림에서 적절한 동작은 다음과 같습니다.

- 요소 시퀀스를 균일하게 변경
- 요소 시퀀스 필터링
- 단일 작업을 사용하여 요소 시퀀스 결합(요소 추가 및 최소값 계산)
- 요소의 시퀀스를 컬렉션으로 합쳐서, 공통 속성별로 그룹화
- 일부 기준을 충족하는 요소에 대한 요소 시퀀스를 검색

이러한 경우에 매우 좋습니다.

스트림을 사용하면 아래처럼 이쁜 코드를 구성할 수 있습니다.

```java
// 가장 큰 메르센 소수 20개를 출력하는 코드.
public static void main(String[] args) {
  primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
    .filter(mersenne -> mersenne.isProbablePrime(50))
    .limit(20)
    .forEach(System.out::println);
}
```

그러나 항상 Stream을 쓸지, 혹은 iteration을 사용할지는 애매한 경우가 많습니다. 이러한 경우에서는 개발자의 취향에 가깝습니다. 즉, **어떤 작업이 Stream이나 Iteration에 의해 더 나은건지 확실하지 않기 때문에, 두가지 모두를 시도하고 어떤 것이 나을지 고르는 것이 중요합니다.**

<br/>

## Item 46. 스트림에서 부작용이 없는 함수를 선택합니다.

스트림은 단순한 API가 아니라, 함수형 프로그래밍을 기반으로 하는 패러다임입니다. 이러한 스트림이 제공해야하는 표현성과 속도, 경우에 따라 병렬화 가능성을 얻기위해서는 패러다임과 API를 채택해야합니다.

스트림 패러다임의 가장 중요한 부분은 각 단계의 결과가 이전 단계의 `pure function`에 최대한 가까운 변환 시퀀스로 계산을 구성하는 것입니다. (`pure function`은 결과가 입력에만 의존하는 함수입니다. 즉, 변경 가능한 상태에 의존하지 않으며 어떤 상태도 업데이트가 없습니다.)

```java
Map <String, Long> freq = new HashMap <> ();

// 잘못 스트림을 사용한 경우.
try (Stream <String> words = new Scanner (file) .tokens ()) {
  words.forEach (word-> {
    freq.merge (word.toLowerCase (), 1L, Long::sum);
  });
}

// 스트림 API를 잘 사용한 경우.
try (Stream <String> words = new Scanner (file) .tokens ()) {
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

위와 같이 `forEach` 연산은 동작 연산을 수행하지 않게 **스트림 결과를 보고하는 목적으로 사용**해야합니다.

```java
// freq 테이블에서 상위 10 개 단어 목록을 가져 오는 파이프 라인
List <String> topTen = freq.keySet().stream()
    .sorted (comparison(freq::get).reversed()).limit(10)
    .collect (toList ());
```

그리고 해당 코드처럼, Collectors의 member를 정적으로 가져오는 것이 스트림 파이프라인을 더 읽기 쉽게 만듭니다.

이외에도 여러개의 Collectors들이 존재합니다. 스트림 파이프 라인 프로그래밍의 본질은 부작용이 없는 함수 객체를 구성하는 것입니다. 이는 스트림 및 관련 개체에 전달된 많은 함수 개체 모두에 적용됩니다.

- `forEach` : 계산을 수행하는 것이 목적이 아니라, 스트림에 의해 수행된 계산 결과를 보고해야하는 곳에 사용해야합니다.
- `toList`, `toSet`, `toMap`, `groupingBy`, `joining` 등의 Collector factories를 사용하는 것이 중요합니다.

<br/>

## Item 47. Return 타입으로 스트림보다는 컬렉션을 선택합니다.

<br/>

## Item 48. 스트림을 병렬로 만들 때 주의합니다.
