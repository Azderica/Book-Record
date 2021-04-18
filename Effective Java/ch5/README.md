# 제네릭

Java 5이후로, 제네릭은 언어의 일부였습니다. 제네릭을 사용하면 각 컬렉션에서 허용되는 개체 유형을 컴파일러에 알리고, 자동으로 캐스트를 삽입합니다. 대부분 프로그램이 더 안전하고 명확하지만, collections에만 한정적이지 않기 때문에 신경을 써야하는 부분이 있습니다.

## Raw 타입을 사용하면 안됩니다.

Raw 타입을 잘못 사용한 코드와 잘된 코드는 다음과 같습니다.

```java
// Raw collection type
private final Collection stamps = ...;

// Parameterized collection type - typesafe
private final Collection<Stamp> stamps = ...;
```

Raw 타입을 사용하면, 제네릭의 안전성과 표현력 이점을 잃게 되므로 사용하면 안됩니다.

```java
// 제한되지 않은 와일드 카드 유형을 사용 - typesafe하고, 유연합니다.
static int numElementsInCommon (Set <?> s1, Set <?> s2) {...}
```

클래스 리터럴에는 원시 유형을 사용할 수 있는데, 대표적으로 `instanceof` 가 있습니다.

```java
// raw type의 합법적 사용 - instanceof 연산자
if (o instanceof Set) { // Raw type
  Set<?> s = (Set<?>) o;    // Wildcard type

  ...
}
```

즉, 정리하면 raw type을 사용하면 런타임에 예외가 발생할 수 있기 때문에 사용하지 않는 것이 중요하며, 제네릭 도입 이전의 레거시 코드와의 호환성 및 상호 운용성을 위해서만 사용해야합니다.

<br/>

## 확인되지 않은 경고를 제거합니다.

제네릭으로 프로그래밍할 때 확인되지 않은 캐스트 경고, 확인되지 않은 메서드 호출 경고, 확인되지않은 매개 변수인 vararg 유형 경고 및 다양한 컴파일러 경고가 발생합니다.

이 경우에, **확인되지 않은 모든 경고를 제거해야합니다.**

일부 경고를 제거할 수는 없지만, 경고를 유발한 코드가 typesafe하다는 것을 증명할 수 있는 경우 `@SuppressWarnings("unchecked")` 주석으로 경고를 억제할 수 있습니다. (다만, 이는 가능한 작은 범위에서 사용하는 것이 중요합니다.)

```java
// @SuppressWarnings의 범위를 줄이기 위해 지역 변수를 추가합니다.
public <T> T[] toArray(T[] a) {
  if (a.length < size) {
    // 이 캐스트는 우리가 만들고있는 배열이기 때문에 정확합니다.
    // 전달 된 것과 동일한 유형, 즉 T []입니다.

    @SuppressWarnings("unchecked") T[] result =
      (T[]) Arrays.copyOf(elements, size, a.getClass());
    return result;
  }

  System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
  return a;
}
```

추가적으로, `@SuppressWarnings("unchecked")` 주석을 사용할 때마다, 안전한 이유를 설명하는 주석을 추가하는 것이 필요합니다.

<br/>

## Arrays 보다는 List를 선호합니다.

Array는 제네릭 유형과 두가지 중요한 측면에서 다릅니다.

- 1. 배열은 covariant(함께 변할 수 있고), 제네릭은 erasure(불변)입니다.

```java
// Runtime에 실패함.
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in";  // ArrayStoreException 에러가 발생합니다.

// Compile되지 않습니다.
List<Object> ol = new ArrayList<Long> (); // 호환되지 않음
ol.add("I don't fit in");
```

- 2. 배열은 reified

```java
// 배열 생성이 불법이며, 컴파일되지 않습니다.
List<String>[] stringLists = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
objects[0] = intList;
String s = stringLists[0].get(0);
```

<br/>

## Generic type을 선호합니다.

일반적으로 선언을 매개 변수화하고 JDK에서 제공하는 제네릭 유형 및 메소드를 사용하는 것은 어렵지 않으며, 그만한 가치가 있습니다.

```java
// 객체 기반 컬렉션-제네릭의 주요 후보
public class Stack {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  // 요소 배열에는 push (E)의 E 인스턴스 만 포함됩니다.
  // 이것은 타입 안전성을 보장하기에 충분하지만
  // 배열 의 런타임 타입은 E []가 아닙니다; 항상 Object []입니다!
  @SuppressWarnings ( "unchecked")
  public Stack() {
    elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(E e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public E pop() {
    if (size == 0)
      throw new EmptyStackException();

    // 일반 스택을 실행하는 작은 프로그램
    @SuppressWarnings("unchecked")
    E result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
  }

  public boolean isEmpty() {
    return size == 0;
  }

  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
  }
}
```

다음과 같이 경고 창을 제거할 수 있습니다.

<br/>

## Generic methods를 선호합니다.

클래스가 제네릭일 수 있는 것처럼 메소드도 가능합니다.

```java
// Uses raw types - 허용되지 않습니다.
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}

// Generic method
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<>(s1);
  result.addAll(s2);
  return result;
}
```

이러한 generic method를 사용하는 간단한 코드는 다음과 같습니다.

```java
public static void main(String[] args) {
  Set<String> guys = Set.of("Tom", "Dick", "Harry");
  Set<String> stooges = Set.of("Larry", "Moe", "Curly");
  Set<String> aflCio = union(guys, stooges);
  System.out.println(aflCio);
}
```

식별함 후 디스펜서를 작성하면 다음과 같습니다.

```java
// 일반 싱글 톤 팩토리 패턴
private static UnaryOperator <Object> IDENTITY_FN = (t)-> t;

@SuppressWarnings ( "unchecked")
public static <T> UnaryOperator <T> identityFunction () {
  return (UnaryOperator <T>) IDENTITY_FN;
}
```

컬렉션의 최대 값을 계산하는 코드입니다.

```java
// 컬렉션에서 최대 값을 반환합니다. 재귀 유형 바인딩을 사용합니다.
public static <E extends Comparable<E>> E max(Collection<E> c) {
  if (c.isEmpty())
    throw new IllegalArgumentException("Empty collection");

  E result = null;
  for (E e : c)
    if (result == null || e.compareTo(result) > 0)
      result = Objects.requireNonNull(e);
  return result;
}
```

위의 내용을 요약하면 다음과 같습니다. generic type과 같은 generic methods는 클라이언트가 입력 매개 변수에 명시적 캐스트를 입력하고 값을 반환해야하는 메서드보다 안전하고 사용하기 쉽습니다. 이를 위해 메소드를 캐스트 없이 사용할 수 있는지 확인해야하며, 이는 generic을 의미합니다.

<br/>

## API 유연성을 향상시키기 위해서, 제한된 Wildcards를 사용합니다.

<br/>

## Generics와 Varargs를 신중하게 결합합니다.

<br/>

## Typesafe한 Heterogeneous 컨테이너를 고려합니다.
