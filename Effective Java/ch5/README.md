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

## Arrays 보다는 List를 선호합니다.

## Generic 타입을 선호합니다.

## API 유연성을 향상시키기 위해서, 제한된 Wildcards를 사용합니다.

## Generics와 Varargs를 신중하게 결합합니다.

## Typesafe한 Heterogeneous 컨테이너를 고려합니다.
