# 클래스와 인터페이스

Class와 Interface는 추상화의 기본 단위이며, 이를 위해 여러 요소 등을 사용할 수 있습니다.

## 클래스 및 멤버의 접근성을 최소화합니다.

- 정보 은닉은 개발, 테스트, 최적화, 사용, 이해 및 수정에서 큰 용이성을 가집니다.
- **각 클래스 또는 멤버를 가능한 한 액세스 할 수 없게 처리합니다.**

액세스 수준은 다음과 같이 4가지로 구성됩니다.

- private : 선언된 최상위 클래스에서만 액세스 가능
- package-private(default) : 선언된 패키지의 모든 클래스에서 액세스 가능
- protected : 선언된 클래스의 하위 클래스 및 선언된 패키지의 모든 클래스에서 액세스 가능
- public : 어디서나 액세스 가능

추가적으로 지켜야하는 룰은 다음과 같습니다.

- public 클래스의 인스턴스 필드는 public이면 안됩니다.
- **변경가능한 public 필드가 있는 class는 일반적으로 스레드로부터 안전하지 않습니다.**
- 클래스에 public static final array field 또는 이러한 필드를 반환하는 접근자가 있으면 안됩니다.
  - 해결책은 2개가 있습니다.
  - public array를 비공개로 바꾸고, public static 목록에 추가합니다.
  - array를 private로 만들고, public method를 추가합니다.

```java
// 잠재적인 보안 구멍
public static final Thing[] VALUES = {...};
```

**결론적으로, 프로그램 요소의 접근성을 최대한 줄여야합니다.**

<br/>

## public class에서는 public field가 아닌, 접근자 메소드를 사용합니다.

```java
class Point {
  // 이런식으로 짜면, 캡슐화의 이점이 없습니다.
  public double x;
  public double y;
}
```

- 클래스가 패키지 외부에서 액세스 가능한 경우, 접근자 메소드를 접공합니다.
- 그러나, 클래스가 패키지 전용 클래스거나 전용 중첩 클래스인 경우, 데이터 필드를 노출하는데 본질적인 문제는 없습니다.

<br/>

## 변경 가능성을 최소화합니다.

분변 클래스는 단순히 인스턴스를 수정할 수 없는 클래스이며 이는 설계, 구현 및 사용하기에 더 쉬우며 오류 가능성이 적고 더 안전합니다.

클래스를 불변으로 만들려면 5가지 규칙을 지켜야합니다.

- 객체의 상태(state)를 수정하는 메소드를 제공하면 안됩니다.
- 클래스를 확장할 수 없는지 확인합니다.
- 모든 필드를 최종으로 만듭니다.
- 모든 필드를 private로 설정합니다.
- 변경 가능한 구성 요소에 대한 독점적인 액세스를 보장합니다.

변경 불가능한 객체는 이러한 장점을 가지고 있습니다.

- 스레드로부터 안전하며 동기화가 필요하지 않습니다. 그렇기에 이러한 객체는 자유롭게 공유할 수 있습니다.
- 다른 개체를 위해서 좋은 **building block**을 만듭니다.
- 상태는 변경되지 않기 때문에, 일시적인 불일치 가능성이 없습니다.

다만 이러한 단점을 가지고 있습니다.

- 각 고유 값에 대해 별도의 객체가 필요합니다.

클래스를 변경 불가능하게 만들 수 없는 경우, 가능한 변경 가능성을 제한해야합니다. (즉, 모두 setter를 선언할 필요가 없습니다.) 그리고 다른 이유가 없으면, 모든 필드를 private final로 선언해야합니다.

생성자는 모든 불변성을 설정하여, 완전히 초기화된 객체를 만들어야합니다.

<br/>

## Inheritance(상속)보다 Composition(구성)을 선호합니다.

상속은 코드 재사용을 달성하는 좋은 방법이지만, 항상 좋은 방법은 아닙니다.

- 메서드 호출과 달리 상속은 캡슐화를 위반합니다.

좋은 경우는 다음과 같습니다.

```java
// Wrapper Class - 상속 대신 합성을 사용하는 경우.
public class InstrumentedSet <E> extends ForwardingSet <E> {
  private int addCount = 0;

  public InstrumentedSet (Set <E> s) { super(s) }

  @Override public boolean add(E e) {
    addCount++;
    return super.add(e);
  }

  @Override public boolean addAll(Collection<? extends E> c) {
    addCount += c.size();
    return super.addAll(c);
  }

  public int getAddCount() {
    return addCount;
  }
}

// Reusable forwarding class
public class ForwardingSet<E> implements Set<E> {
  private final Set<E> s;
  public ForwardingSet(Set<E> s) { this.s = s; }
  public void clear() { s.clear(); }

  public boolean contains(Object o) { return s.contains(o); }
  public boolean isEmpty()          { return s.isEmpty();   }
  public int size()                 { return s.size();      }
  public Iterator<E> iterator()     { return s.iterator();  }
  public boolean add(E e)           { return s.add(e);      }
  public boolean remove(Object o)   { return s.remove(o);   }
  public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
  public boolean addAll(Collection<? extends E> c) { return s.addAll(c);      }
  public boolean removeAll(Collection<?> c) { return s.removeAll(c);   }
  public boolean retainAll(Collection<?> c) { return s.retainAll(c);   }
  public Object[] toArray()          { return s.toArray();  }
  public <T> T[] toArray(T[] a)      { return s.toArray(a); }
  @Override public boolean equals(Object o) { return s.equals(o);  }
  @Override public int hashCode()    { return s.hashCode(); }
  @Override public String toString() { return s.toString(); }
}
```

위와 같은 코드는 인터페이스를 통해서 클래스의 디자인이 가능하고 매우 유연합니다.

상속은 하위 클래스가 실제로 수퍼 클래스의 하위 유형인 상황에서만 적절합니다. 즉, `is-a` 관계인 경우에만 주로 사용하는 것이 좋습니다.

<br/>

## 상속을 위한 설계 및 문서 또는 금지

<br/>

## 추상 클래스보다는 인터페이스를 선호합니다.

<br/>

## posterity(후세)를 위한 디자인 인터페이스

<br/>

## 인터페이스를 사용해 Types를 정의합니다.

<br/>

## 태그가 있는 클래스보다 클래스 계층을 선호합니다.

<br/>

## 비정적보다 정적 멤버 클래스를 선호합니다.

<br/>

## 소스 파일을 단일 최상위 클래스로 제한합니다.
