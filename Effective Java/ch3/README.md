# 모든 객체에 공통적인 메소드

객체는 콘크리트 클래스이지만, 주로 확장을 위해 설계됩니다. (대표적인 예시로, equals, hashCode, toString, clone, finalize 등이 override되도록 설계되어있습니다.)

nonfinal Object 메소드를 재정의하는 시기와 방법에 대허 설명합니다.

## 'Equals'를 오버라이딩 할 때, 일반적인 룰을 준수합니다.

equals 메서드를 재정의하는 방법은 여러가지가 있지만, 잘못된 사용은 끔찍한 결과를 만듭니다. 따라서 다음의 룰을 준수해야합니다.

- 클래스의 각 인스턴스는 본질적으로 unique합니다.
- 클래스에 대해 `logical equality(지역적 동일성)` 테스트를 제공할 필요가 없습니다.
- 슈퍼 클래스는 이미 equals를 이미 오버라이딩하였으므로, 슈퍼클래스의 동작은 이미 클래스의 적합합니다.
- 클래스는 private나 package-private이므로, 해당 'equals'는 호출되지 않을것이라고 확신합니다.

### equivalence relation의 조건.

equivalence relation 이란, 요소 집합에서 요소가 서로 동일한 것으로 간주하는 하위 집합으로 분할하는 연산자이며 이를 `equivalence class`라고 합니다. 이를 위해서는 5가지의 요구 사항을 지켜야합니다.

- `Reflexivity(반사성)`

  - 객체가 자신과 동일해야합니다.

- `Symmetry(대칭)`

  - 두 객체가 동일한 지 여부에 대해 동의해야합니다.
  - equals 를 위반한 경우, 해당 객체가 다른 객체를 비교하게 되면 어떻게 동작할지 알 수가 없습니다.

```java
// 대칭을 위반한 케이스
public final class CaseInsensitiveString {
  private final String s;

  public CaseInsensitiveString(String s) {
    this.s = Objects.requireNonNull(s);
  }

  // 대칭을 위반한 경우
  @Override public boolean equals(Object o) {
    if (o instanceof CaseInsensitiveString)
      return (s.equalsIgnoreCase((CaseInsensitiveString) o).s);
    if (o instanceof String)  // 단방향 상호 운용성
      return s.equalsIgnoreCase ((String) o);
    return false;
  }
}
```

```java
// 대칭을 준수한 코드
@Override public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString &&
    ((CaseInsensitiveString) o).s.equalsIgnoreCase (s);
}
```

- `Transitivity`

  - 한 객체가 두번째 객체와 같고, 두번째 객체가 세번째 객체와 같으면 첫번째 객체와 세번째 객체가 같아야합니다.

```java
// equals contract를 위반하지 않는 값 구성 요소
public class ColorPoint {
  private final Point point;
  private final Color color;

  public ColorPoint(int x, int y, Color color) {
    point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }

  public Point asPoint() {
    return point;
  }

  @Override public boolean equals(Object o) {
    if(!(o instanceof ColorPoint))
      return false;
    ColorPoint cp = (ColorPoint) o;
    return cp.point.equals(point) && cp.color.equals(color);
  }
}
```

- `Consistency`

  - 두 객체가 같은 경우에, 둘 중 하나가 변경되지 않는 한 항상 동일하게 유지되어야합니다.
  - 신뢰할 수 없는 리소스에 의존하는 경우, equals를 사용하면 안됩니다.
  - 대표적으로 사용하면 안되는 것이, `java.net.url`에서의 equals이며, 이는 IP를 사용하기 때문에 시간이 바뀌면서 바뀔 수 있습니다.

- `Non-nullity`

  - 모든 객체는 null과 같으면 안됩니다.

```java
// Implicit null check - preferred
@Override public boolean equals(Object o) {
  if (!(o instanceof MyType))
    return false;
  MyType mt = (MyType) o;
  ...s
}
```

### 좋은 equals 사용 방법

- `==`를 사용하여 인수가 이 객체에 대한 참조인지 확인합니다.
- `instanceof`를 사용해서 argument의 유형한 타입인지 확인합니다.
- 올바른 유형으로 캐스트합니다.
- 클래스의 각 중요한 필드에 대해 인수의 해당 필드가, 이 객체의 해당 필드와 일치하는 지 확인합니다.

이러한 방법으로 equals를 작성하고 나서는 세가지를 확인해야합니다.

- `symmetric`, `transitive`, `consistent`

그 외의 주의사항은 다음과 같습니다.

- `equals`를 재정의할 때는, `hashCode`를 재정의합니다.
- 너무 영리하게 할 필요가 없습니다. 복잡하게 구성하면 안됩니다.
- `equals`를 선언할 때는, 객체를 다른 타입으로 대체하면 안됩니다.

<br/>

## 'Equals'를 오버라이딩 할때, `Hashcode`를 항상 오버라이딩합니다.

<br/>

## 'ToString'을 항상 오버라이딩합니다.

<br/>

## 신중하게 'Clone'을 오버라이딩합니다.

<br/>

## 'Comparable'을 개발할때 고려합니다.

```

```
