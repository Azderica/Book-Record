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

## Arrays 보다는 List를 선호합니다.

## Generic 타입을 선호합니다.

## API 유연성을 향상시키기 위해서, 제한된 Wildcards를 사용합니다.

## Generics와 Varargs를 신중하게 결합합니다.

## Typesafe한 Heterogeneous 컨테이너를 고려합니다.
