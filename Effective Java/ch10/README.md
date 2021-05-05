# Exceptions

## Item 69. 예외는 진짜 예외 상황에서만 사용합니다.

예외는 꼭 필요한 경우에만 사용해야합니다.아래는 잘못 사용한 케이스입니다.

```java
try {
  int i = 0;
  while(true)
    range[i++].climb();
} catch (Exception e) {
  ...
}
```

예외를 사용할 때는 몇가지 준수사항이 있습니다.

- 예외는 예외적인 상황을위해 설계되었기 때문에, JVM 구현자가 명시적으로 빠르게 할 필요가 없습니다.
- `try-catch` 블록 내에 코드를 배치하면 JVM 구현이 수행할 수 있는 특정 최적화가 금지됩니다.
- 배열을 반복하는 표준 관용구가 반드시 중복 검사를 발생시키는 것이 아닙니다.

따라서, 예외는 반드시 예외 상황에서만 사용해야하며 일반적인 제어 흐름에서는 절대로 사용하면 안됩니다. 이를 위해 상태 검사 메서드 등을 제공하거나, Optional, 또는 특정 값을 반환하면 안됩니다.

이는 API 설계에도 적용되는 규칙입니다. 잘 설계된 API는 클라이언트가 일반 제어 흐름에 예외를 사용하도록 강요하면 안됩니다.

아래와 같은 코드는 매우 잘못된 코드입니다.

```java
// 컬렉션 반복에 이런 코드는 최악입니다.
try {
  Iterator <Foo> i = collection.iterator ();
  while (true) {
    Foo foo = i.next ();
    ...
  }
} catch (NoSuchElementException e) { ... }
```

위와 같은 코드는 매우 잘못된 코드입니다. 따라서, 예외는 예외적인 조건을 위해 설계되었습니다. 일반적으로 사용하는 제어문에 사용하면 안되며, 다른 사람들이 그렇게 하도록 강요하는 API를 작성하면 안됩니다.

<br/>

## Item 70. 복구 가능한 조건에는 체크된 예외를 사용하고, 프로그래밍 오류에는 런타임 예외를 사용합니다.

자바는 `checked exception`, `runtime exception`, `errors`의 3가지 종류의 throwable을 제공합니다. 각 종류의 throwable을 사용하는 것이 적절한 시기에 대해 프로그래머 간에 약간의 혼동이 있습니다.

검사 예외(`checked exception`)와 비검사 예외(`unchecked exception`)를 구분하는 기본 규칙은 간단합니다.

호출하는 쪽에서 복수할 수 있다고 생각된다면 `checked exception`를 사용합니다. `checked exception`를 던지면, `try-catch`로 처리하거나 `throw`를 이용해서 더 바깥쪽으로 전파하도록 강제합니다

`unchecked exception`은 `runtime exception`와 `errors`가 있습니다. 이 이러한 경우는 프로그램에서 잡을 필요가 없거나 잡아도 득보다 실이 많은 경우입니다. 또한 `throwable`의 경우 직접 구현이 가능한데, `Exception`, `RuntimeException`, `Error` 클래스를 상속하지 않는 구현은 좋지 않습니다.

> [Exception에 대한 글](https://madplay.github.io/post/java-checked-unchecked-exceptions)

즉, 복구 가능한 조건에 대해서는 `checked exception`를 써야하고, 프로그래밍 오류에 대해서는 `runtime exception`를 던져야합ㄴ니다. 확실하지 않는 경우에서는 `unchecked exceptions`를 throw 합니다. `checked exception`이나 `runtime exception`가 아닌 경우, `throwable`을 정의하면 안됩니다.

<br/>

## Item 71. `checked exceptions`의 불필요한 사용을 피합니다.

많은 Java 프로그래머는 `checked exceptions`를 싫어하지만, 제대로 사용하게 되면 API와 프로그램을 향상시킬 수 있습니다. 반환 코드 및 확인되지 않은 예외와는 달리 프로그래머가 문제를 처리하도록하여 안정성을 향상시킵니다.

<br/>

## Item 72. 표준 예외 사용을 선호합니다.

<br/>

## Item 73. 추상화에 적합한 예외를 던집니다.

<br/>

## Item 74. 각 메소드가 던진 모든 예외를 문서화합니다.

<br/>

## Item 75. 세부 메시지에 실패 캠처 정보를 포함합니다.

<br/>

## Item 76. 실패 원자성을 위해 노력하라

<br/>

## Item 77. 예외를 무시하면 안됩니다.
