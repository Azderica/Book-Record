# Serialization

아래에서는 자바 직렬화의 위험성과 이를 최소화하는 방법을 중점으로 합니다.

- 자바 직렬화란.

## Item 85. 자바 직렬화의 대안을 찾습니다.

자바의 직렬화는 위험합니다. 자바의 직렬화의 위험한 이유는 다음과 같습니다.

- 공격 범위가 너무 넓습니다.
- 지속적으로도 더 넓어져서 방어하기도 어렵습니다.

`OutputInputStream`의 `readObject` 메서드를 호출하면서 객체 그래프가 역직렬화(deserialization)가 되기 때문입니다.

바이트 스트림을 역직렬화하는 과정에서 `readObject` 메서드는 그 타입들 안의 모드 코드를 수행할 수 있습니다. (즉, 타입들의 코드 전체가 악의적인 공격 범위에 들어갑니다.)

역직렬화 과정에서 호출되어 잠재적인 위험한 동작을 수행하는 메서드를 **가젯(gadget)** 이라고 합니다. 하나의 가젯이 여러개의 가젯이 마음대로 코드를 수행할 수 있기 때문에 아주 신중하게 제작된 바이트 스트림만 역직렬화를 해야합니다.

역직렬화에 시간이 오래걸리는 짧은 스트림을 **역직렬화 폭탄(deserialization bomb)** 이라고 합니다. 아래는 그 예시입니다.

```java
static byte[] bomb() {
  Set<Object> root = new HashSet<>();
  Set<Object> s1 = root;
  Set<Object> s2 = new HashSet<>();

  for (int i = 0; i < 100; i++) {
    Set<Object> t1 = new HashSet<>();
    Set<Object> t2 = new HashSet<>();

    t1.add("foo"); // Make t1 unequal to t2
    s1.add(t1);  s1.add(t2);
    s2.add(t1);  s2.add(t2);

    s1 = t1;
    s2 = t2;
  }
  return serialize(root); // Method omitted for brevity
}
```

이를 호출해버리면, 깊이가 100단계까지 호출됩니다. 이를 역직렬화 하려면 2^100 번 넘게 호출해야합니다.

자바 직렬화는 **크로스-플랫폼 구조화된 데이터 표현 방법** 으로 대체해야 합니다. 예로는 JSON, protocol buffer 등이 있습니다. 프로토콜 버퍼는 이진 표현이라 효율이 훨씬 더 높으며, JSON은 텍스트 기반이라 사람이 읽을 수 없는 장점이 있습니다.

직렬화를 대체할 수 없다면, 반드시 신뢰할 수 있는 데이터만 역직렬화해야합니다. 직렬화를 피할 수 없고, 역직렬화한 데이터가 안전하지 확실할 수 없다면 객체 역직렬화 필터링을 사용하면 됩니다.

다만, 직렬화는 위험 요소가 많습니다. 시간과 노력을 쓰더라도, JSON 등으로 마이그레이션하는 것을 추천합니다.

<br/>

## Item 86. `Serializable`을 구현할지에 대해 신중히 결정합니다.

직렬화 가능한 클래스는 `Serializable`을 구현하면 됩니다. 이를 구현하는 것은 싶지만, 구현을 했을 때의 대가는 매우 비쌉니다. 구현한 순간부터 많은 위험성을 가지게 되고, 확장성을 잃게 됩니다.

### 직렬화 클래스의 단점

#### 1. 릴리즈 이후에는 유연성이 어렵습니다.

`Serializable`을 구현하면 직렬화 형태도 하나의 공개 API가 됩니다. 직렬화 형태는 적용 당시 클래스의 내부 구현 방식에 종속적입니다. 또한 클래스의 private과 package 인스턴스 필드마저 API로 공개되기 때문에 캡슐화도 깨집니다.

클래스의 내부 구현을 수정할 시, 원래의 직렬화 형태와 달라집니다. 구버전의 인스턴스를 직렬화한 후 신버전 클래스로 역직렬화를 시도하면 오류가 발생합니다.

한편 수정을 어렵게 만드는 요소로 `SerialVersionUID`를 뽑을 수 있습니다. 모든 직렬화된 클래스는 고유 식별 번호를 부여받으며, 클래스 내부에 직접 명시하지 않는 경우에 시스템이 런타임에 자동으로 생성됩니다. `SUID`를 생성할 때는 클래스의 이름, 구현하도록 선언한 인터페이스 등이 고려됩니다. 따라서 나중에 수정한다면 `SUID` 값도 변하게 됩니다. 이러한 자동으로 생성된 값은 호환성이 쉽게 깨집니다.

#### 2. 버그와 보안에 취약합니다.

자바에서는 객체를 생성자를 통해서 만듭니다. 그러나 직렬화는 이러한 언어의 기본 방식을 우회하면서 객체를 생성합니다. 역직렬화는 일반 생성자의 문제가 발생하는 숨은 생성자입니다. 역직렬화를 사용하면 불변식이 깨질 수 있으며 허가되지 않은 접근에 쉽게 노출될 수 있습니다.

#### 3. 테스트 부담 요소가 증가합니다.

직렬화 가능한 클래스가 수정되면, 새로운 버전의 인스턴스를 직렬화 한후에 구버전으로 역직렬화가 가능한지 테스트해야합니다. 물론 그 반대 경우도 테스트 해야합니다.

### `Serializable` 구현

`Serializable` 구현 여부는 쉽게 결정할 것이 아닙니다. 클래스를 설계할 때마다 따르는 이득과 비용을 잘 고려해야합니다. 에를 들어 `BigInteger`과 `Instant` 같은 값 클래스와 컬렉션 클래스는 `Serializable`을 구현하였으며 스레드 풀처럼 동작하는 객체를 표현한 클래스는 대부분 구현하지 않았습니다.

따라서 아래의 경우는, `Serializable`을 구현하면 안되는 경우입니다.

#### 상속 목적으로 설계된 클래스와 대부분의 인터페이스는 `Serializable`을 구현하면 안됩니다.

클래스를 확장하거나 인터페이스를 구현하는 대상에게 위험성을 제공합니다. 하지면 `Serializable`를 구현한 클래스만 지원하는 프레임워크를 사용해야한다면 어쩔 수 없습니다. 이러한 경우처럼 직렬화와 확장이 모두 가능한 클래스를 만들어야한다면 하위 클래스에서 `finalize` 메서드를 재정의하지 못하게 해야합니다. 일반적으로는 재정의하고 `final` 키워드를 붙이면 되며, 인스턴스 필드 중 기본값으로 초기화되어서 위배되는 불변식이 있는 경우에는 아래와 같은 메서드를 추가합니다.

```java
private void readObjectNoData() throws InvalidObjectException {
  throw new InvalidObjectException("Stream data required");
}
```

#### 내부 클래스는 직렬화를 구현하면 안됩니다.

기본 직렬화 형태가 명확하지 않습니다. 내부 클래스는 바깥 인스턴스의 참조와 유효 범위에 속한 지역변수를 저장하기 위한 필드가 필요합니다. 이 필드들은 컴파일러가 자동으로 추가를 하는데, 이 필드들이 어떻게 추가될 지 모릅니다. (정적 멤버 클래스는 다릅니다.)

<br/>

## Item 87. 커스텀 직렬화 형태를 고려합니다.

### 커스텀 직렬화가 필요한 이유.

클래스가 `Serializable` 을 구현하고 기본 직렬화 형태를 사용한다면 현재의 구현에 종속적이게 됩니다. 즉, 기본 직렬화 형태를 버릴 수 없게 됩니다. 따라서 유연성, 성능, 정확성과 같은 측면을 고민한 후에 합당하다고 생각되는 경우에 한해 기본 직렬화 형태를 사용해야합니다.

### 이상적인 직렬화 형태

기본 직렬화 형태는 객체가 포함한 데이터 뿐만 아니라, 그 객체를 시작으로 접근할 수 있는 모든 객체와 객체들의 연결된 정보까지 나타냅니다. 이상적인 직렬화의 형태는 물리적인 모습과 독립된 논리적인 모습만을 표현해야합니다. 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 선택해도 무방합니다.

```java
public class Name implements Serializable {

  @serial
  private final String lastName;

  @serial
  private final String firstName;

  @serial
  private final String middleName;

  ...
}
```

이름은 논리적으로 성, 이름, 중간 이름으로 3개의 문자열로 구성되는데 위 클래스의 인스턴스 필드들은 논리적인 구성 요소를 정확하게 반영합니다.

기본 직렬화 형태가 적합해도 불변식 보장과 보안을 위해서 `readObject` 메서드를 제공해야하는 경우가 많습니다. 앞에 있는 코드의 경우, lastName과 firstName 필드는 null이 아님을 `readObject` 메서드가 보장해야합니다.

### 부적절한 직렬화 형태

객체의 물리적 표현과 논리적 내용이 같은 경우, 기본 직렬화 형태를 선택해도 됩니다. 그러나 적절하지 않는 경우도 있습니다.

```java
public final class StringList implements Serializable {
  private int size = 0;
  private Entry head = null;

  private static class Entry implements Serializable {
    String data;
    Entry next;
    Entry previous;
  }

  // ... 생략
}
```

위의 클래스 경우에는 여러 문제점이 있습니다. 논리적으로 문자열을 표현했고 물리적으로는 문자열들을 이중 연결 리스트로 표현했습니다. 이 클래스에 기본 직렬화 형태를 사용하면 각 노드에 연결된 노드들까지 모두 표현하기 때문에 다음과 같은 문제가 발생합니다.

- 공개 API가 현재의 내부 표현 방식에 종속적이게 됩니다.
  - 향후 버전에서 연결 리스트를 사용하지 않더라도, 관련 처리가 필요해집니다.
  - 코드를 제거할 수가 없습니다.
- 사이즈가 큽니다
  - 기본 직렬화를 사용할 때 각 노드의 연결 정보까지 모두 포함될 것입니다.
  - 이는 내부 구현이며 직렬화 형태에 가치가 없으며 네트워크 전송 속도를 느리게 합니다.
- 시간이 많이 걸립니다.
  - 직렬화 로직은 객체 그래프의 위상에 관한 정보를 알 수 없으니, 직접 순회할 수 밖에 없습니다.
- 스택 오버플로를 발생시킵니다.
  - 기본 직렬화 형태는 객체 그래프를 재귀 순회하며, 호출 정도가 많아지면 스택이 감당을 하지 못합니다.

### 합리적인 직렬화 형태

이를 수정해서 합리적인 직렬화 형태는 다음과 같습니다. 단순히 리스트가 포함한 문자열의 개수와 문자열만 있는 것이 좋습니다. 위의 부적절한 코드를 개선한 형태입니다.

```java
public final class StringList implements Serializable {
  private transient int size = 0;
  private transient Entry head = null;

  private static class Entry {
    String data;
    Entry next;
    Entry previous;
  }

  public final void add(String s) { ... }

  private void writeObject(ObjectOutputStream stream) throws IOException {
    stream.defaultWriteObject();
    stream.writeInt(size);

    for (Entry e = head; e != null; e = e.next) {
      s.writeObject(e.data);
    }
  }

  private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException {
    stream.defaultReadObject();
    int numElements = stream.readInt();

    for (int i = 0; i < numElements; i++) {
      add((String) stream.readObject());
    }
  }

  // ... 생략
}
```

위 코드에서 특별한 키워드인 `transient`를 호가인할 수 있습니다. `transient` 키워드가 붙은 필드는 기본 직렬화 형태에 포함되지 않습니다. 클래스의 모든 필드가 `transient`로 선언되어 있더라도 `writeObject` 와 `readObject` 메서드는 `defaultWriteObject`와 `defaultReadObject` 메서드를 호출합니다. 직렬화 명세에서는 이 과정을 무조건 할 것을 요구합니다. 이렇게 함으로써 향후 릴리즈에서 `transient`가 아닌 필드가 추가되더라도 상위와 하위 모두 호환이 가능하기 때문입니다.

신버전의 인스턴스를 직렬화하고 구버전으로 역직렬화할시, 새로 추가된 필드는 무시됩니다. 그리고 구버전의 `readObject` 메서드에서 `defaultReadObject`를 호출하지 않는다면 역직렬화 과정에서 `StreamCorruptedException`이 발생할 것입니다.

기본 직렬화 여부에 관계없이 `defaultWriteObject` 메서들 호출하면 `transient`로 선언하지 않은 모든 필드는 직렬화됩니다. 따라서, `transient` 키워드를 선언해도 되는 필드라면 붙이는 것이 좋습니다. 즉, 논리적 상태와 무관한 필드라고 판단될 때 생략하는 것이 좋습니다.

기본 직렬화를 사용한다면, 역직렬화를 할 때는 `transient` 필드는 기본 값으로 초기화됩니다. 기본 값을 변경해야 하는 경우에는 `readObject` 메서드에서 `defaultReadObject` 메서드를 호출한 다음 원하는 값으로 지정하면 됩니다. 아니면 값을 처음 사용할 때 초기화해도 됩니다.

기본 직렬화 사용 여부와 상관없이 직렬화에도 동기화 규칙을 적용해야합니다. 예를 들어 모든 메서드를 `synchronized` 로 선언하여 스레드에 안전하게 만든 객체에 기본 직렬화를 사용한다면 `writeObject` 도 아래처럼 수정해야 합니다.

```java
private synchronized void writeObject(ObjectOutputStream stream) throws IOExceptions {
  stream.defaultWriteObject();
}
```

어떤 직렬화 형태를 선택하더라도, 직렬화가 가능한 클래스에는 `SerialVersionUID(SUID)` 를 명시적으로 선언해야 합니다. 물론 선언하지 않더라도 자동 생성되지만 런타임에 이 값을 생성하느라 복잡한 연산을 수행해야합니다.

```java
// 무작위로 고른 long 값
private static final long serialVersionUID = 0204L;
```

다만, SUID가 꼭 유니크할 필요가 없습니다. 다만 이 값이 변경되면 기존 버전 클래스와의 호환을 끊게 됩니다. 따라서 호환성을 끊는 경우가 아니라면 SUID 값을 변경해서는 안됩니다.

<br/>

## Item 88. `readObject` 메서드는 방어적으로 작성합니다.

<br/>

## Item 89. 인스턴스 수를 통제해야한다면 `readResolve`보다는 열거 타입을 사용합니다.

<br/>

## Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토합니다.
