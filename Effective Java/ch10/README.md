# Exceptions

## Item 69. 예외적인 조건에서만 예외를 사용합니다.

예외를 사용할 때는 몇가지 준수사항이 있습니다.

- 예외는 예외적인 상황을위해 설계되었기 때문에, JVM 구현자가 명시적으로 빠르게 할 필요가 없습니다.
- `try-catch` 블록 내에 코드를 배치하면 JVM 구현이 수행할 수 있는 특정 최적화가 금지됩니다.
- 배열을 반복하는 표준 관용구가 반드시 중복 검사를 발생시키는 것이 아닙니다.

일반적으로 예외 기반 관용구는 표준 관용구에 비해 2배정도 느립니다. 즉, **예외는 이름에서 알 수 있듯이 예외적인 조건에서만 사용됩니다. 일반적인 제어 흐름에서는 절대로 사용하면 안됩니다.**

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

`checked exception` 혹은, `runtime exception`를 사용할지 여부를 결정하는 기본 규칙은 다음과 같습니다. 호출자가 합리적으로 복구할 것으로 예상되는 조건에 대해 `checked exception`를 사용합니다. `checked exception`를 throw하면 호출자가 `catch` 문에서 예외를 처리하거나 외부로 전파하도록 강제해야합니다.

확인되지 않은 throwable은 `runtime exception`와 두가지 종류의 오류가가 있습니다. 이러한 에외는 일반적으로 복구가 불가능하고, 계속 실행하게되면 현재 스레드가 종료될 수 있습니다. 이 경우는 `runtime exception`을 사용해서 프로그래밍 오류를 나타낼 수 있습니다. 대부분의 `runtime exception`는 `precondition violation(전제조건 위반)`을 의미합니다.

Java 언어에서는 이가 필요하지는 않지만, JVM에서 리소스 부족, invariant 실패, 실행을 할 수 없는 기타 조건을 나타내기 위해서 오류를 예약하는 강력한 규칙이 존재합니다. 따라서, 구현이 되었으나 확인되지 않는 `throwable`은 `RuntimeException`의 하위 클래스를 만들어야합니다. 다만, Error 하위 클래스는 정의하면 안됩니다.

즉, 복구 가능한 조건에 대해서는 `checked exception`를 써야하고, 프로그래밍 오류에 대해서는 `runtime exception`를 던져야합ㄴ니다. 확실하지 않는 경우에서는 `unchecked exceptions`를 throw 합니다. `checked exception`이나 `runtime exception`가 아닌 경우, `throwable`을 정의하면 안됩니다.

<br/>

## Item 71. 체크된 예외의 불필요한 사용을 피합니다.

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
