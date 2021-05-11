# Serialization

아래에서는 자바 직렬화의 위험성과 이를 최소화하는 방법을 중점으로 합니다.

## Item 85. 자바 직렬화의 대안을 찾습니다.

자바의 직렬화는 위험합니다. 자바의 직렬화의 위험한 이유는 다음과 같습니다.

- 공격 범위가 너무 넓습니다.
- 지속적으로도 더 넓어져서 방어하기도 어렵습니다.

`OutputInputStream`의 `readObject` 메서드를 호출하면서 객체 그래프가 역직렬화(deserialization)이 되기 때문입니다.

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

자바 직렬화는 **크로스-플랫폼 구조화된 데이터 표현 방법** 으로 대체해야 합니다. 예로는 JSON, protocol buffer 등이 있습니다. 프로토콜 버퍼는 이진 표현이라 효츌이 훨씬 더 높으며, JSON은 텍스트 기반이라 사람이 읽을 수 없는 장점이 있습니다.

직렬화를 대체할 수 없다면, 반드시 신뢰할 수 있는 데이터만 역직렬화해야합니다. 직렬화를 피할 수 없고, 역직렬화한 데이터가 안전하지 확실할 수 없다면 객체 역직렬화 필터링을 사용하면 됩니다.

다만, 직렬화는 위험 요소가 많습니다. 시간과 노력을 쓰더라도, JSON 등으로 마이그레이션하는 것을 추천합니다.

<br/>

## Item 86. `Serializable`을 구현할지에 대해 신중히 결정합니다.

<br/>

## Item 87. 커스텀 직렬화 형태를 고려합니다.

<br/>

## Item 88. `readObject` 메서드는 방어적으로 작성합니다.

<br/>

## Item 89. 인스턴스 수를 통제해야한다면 `readResolve`보다는 열거 타입을 사용합니다.

<br/>

## Item 90. 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토합니다.
