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

상속을 위해 클래스를 설계하고 문서화하는 것은 아래를 의미합니다.

- 클래스는 메서드 재정의의 효과를 정확하게 문서화해야합니다. 즉, **클래스는 재정의 가능한 메서드의 자체 사용을 문서화해야합니다.**
- 상속을 위한 디자인은 단순히 자체 사용 패턴을 문서화하는 것 이상을 의미합니다.
- 상속을 위해 설계된 클래스를 테스트하는 유일한 방법은 하위 클래스를 사용하는 방법입니다.
- release 하기 전에 하위 클래스를 작성하여 클래스를 테스트해야합니다.
- 생성자는 재정의 가능한 메서드를 직접 혹은 간접으로 호출하면 안됩니다.
- clone이나 readObject는 직접 또는 간접적으로 재정의 가능한 메서드를 호출할 수 없습니다.
- 안전하게 sub classing 되도록 설계 및 문서화되지 않은 클래스에서 sub classing을 금지하는 것입니다.

<br/>

## 추상 클래스보다는 인터페이스를 선호합니다.

자바에서는 type을 구현하는 두가지 방법은 인터페이스와 추상클래스가 있습니다.

- 기존클래스를 쉽게 개조하여 새 인터페이스를 구현할 수 있습니다.
- 인터페이스는 mixins를 정의하는 것에 이상적입니다.
  - mixin : 클래스가 기본유형에 추가하여 구현할 수 있는 유형이며 선택적 동작을 제공함
- 인터페이스는 nonhierarchical type 프레임워크의 구성을 허용합니다.

```java
public interface Singer {
  AudioClip sing(Song s);
}

public interface Songwriter {
  Song compose (int chartPosition);
}

public interface SingerSongwriter extends Singer, Songwriter {
  AudioClip strum();
  void actSensitive();
}
```

- 인터페이스는 wrapper 클래스를 통해 안전하고 강력한 기능 향상을 가능하게합니다.

### Template Method Pattern

인터페이스와 함께, abstract skeletal 구현 클래스를 제공해서 장점을 결합한 패턴입니다. 인터페이스는 유형을 정의하고, 기본 메소드를 제공하며 skeletal 구현 클래스는 나머지 non-primitive 인터페이스를 구현합니다.

인터페이스 자체에 있는 기본 메소드의 이점을 사용할 수 있고 skeletal 구현 클래스는 구현의 작업을 지원할 수 있습니다,.

```java
// Skeletal implementation class
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
  // Entries in a modifiable map must override this method
  @Override public V setValue(V value) {
    throw new UnsupportedOperationException();
  }

  // Implements the general contract of Map.Entry.equals
  @Override public boolean equals(Object o) {
    if (o == this)
      return true;
    if (!(o instanceof Map.Entry))
      return false;
    Map.Entry<?,?> e = (Map.Entry) o;

    return Objects.equals(e.getKey(), getKey())
      && Objects.equals(e.getValue(), getValue());
  }

  // Implements the general contract of Map.Entry.hashCode
  @Override public int hashCode() {
    return Objects.hashCode(getKey())
      ^ Objects.hashCode(getValue());
  }

  @Override public String toString() {
    return getKey() + "=" + getValue();
  }
}
```

- skeletal 구현은 상속을 위해 설계되었으므로 skeletal 구현에서는 좋은 문서가 절대적으로 필요합니다.

<br/>

## posterity(후세)를 위한 디자인 인터페이스

Java 8 이후로, default method 구성이 추가되었습니다. 또한 주로 람다 사용을 용이하기 위해서 Java 8의 핵심 Collection Interface에 많은 기본 메서드가 추가됩니다. Java의 라이브러리의 기본 메소드는 잘 구현되어 있으며, 대부분 제대로 작동합니다.

그러나 **모든 가능한 구현의 모든 불변을 유지하는 기본 메서드를 작성하는 것이 항상 가능한 것은 아닙니다.**

```java
// Java 8의 Collection 인터페이스에 추가 된 기본 메소드
default boolean removeIf (Predicate <? super E> filter) {
  Objects.requireNonNull(filter);
  boolean result = false;
  for (Iterator<E> it = iterator(); it.hasNext(); ) {
    if (filter.test(it.next())) {
      it.remove();
      result = true;
    }
  }
  return result;
}
```

해당 코드가 removeIf 메소드에 대해 작성할 수 있는 코드이지만, 실제 Collection 구현에서는 실패합니다.

기본 메서드가 있는 경우, 인터페이스의 기존 구현이 오류나 경고없이 컴파일 될 수 있지만 런타임에는 실패합니다.

기본 메소드가 Java 플랫폼의 일부이지만, **인터페이스를 신중하게 디자인하는 것이 여전히 가장 중요합니다.**.

인터페이스 출시 이후에, 몇 가지 인터페이스 결함을 수정하는 것이 가능하지만 이를 믿을 수 없습니다. 따라서 release 하기 전에는 새 인터페이스를 테스트하는 것이 중요합니다.

<br/>

## 인터페이스를 사용해 Types를 정의합니다.

클래스가 인터페이스를 구현할 때, 인터페이스는 클래스의 인스턴스를 참조하는데 사용할 수 있는 type으로 사용됩니다.

상수 인터페이스 패턴은 인터페이스를 제대로 사용하지 못하는 것입니다. 상수 유틸리티 클래스로 다음과 같이 선언할 수 있습니다.

```java
// 상수 유틸리티 클래스
public class PhysicalConstants {
  private PhysicalConstants() {}  // 인스턴스화 방지

  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  public static final double BOLTZMANN_CONST = 1.380_648_52e-23;
  public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

즉, 인터페이스는 type을 정의하는데만 사용해야합니다. 상수를 내보낼 때는 사용해서는 안됩니다.

<br/>

## 태그가 있는 클래스보다 클래스 계층을 선호합니다.

경우에 따라 인스턴스가 둘 이상의 특징으로 제공되는 인스턴스의 특징을 나타내는 tag field를 포함하는 클래스를 실행할 수 있습니다.

```java
// Tagged class - 클래스 계층보다 안좋습니다.
class Figure {
  enum Shape {RECTANGLE, CIRCLE};

  // Tag field : the shape of this figure
  final Shape shape;

  // These fields are used only if shape is RECTANGLE
  double length;
  double width;

  // This field is used only if shape is CIRCLE
  double radius;

  // Constructor for circle
  Figure(double radius) {
    shape = Shape.CIRCLE;
    this.radius = radius;
  }

  // Constructor for rectangle
  Figure(double length, double width) {
    shape = Shape.RECTANGLE;
    this.length = length;
    this.width = width;
  }

  double area() {
    switch(shape) {
      case RECTANGLE:
        return length * width;
      case CIRCLE:
        return Math.PI * (radius * radius);
      default:
        throw new AssertionError(shape);
    }
  }
}
```

이러한 코드는 매우 지저분합니다. 즉, **태그가 지정된 클래스는 장황하고 오류가 발생하기 쉬우며 비효율적입니다.** 이러한 클래스는 클래스 계층 구조를 모방한 것입니다.

이를 클래스 계층으로 나타내면 다음과 같습니다.

```java
// Class hierarchy replacement for a tagged class
abstract class Figure {
  abstract double area();
}

class Circle extends Figure {
  final double radius;

  Circle(double radius) { this.radius = radius; }

  @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
  final double length;
  final double width;

  Rectangle(double length, double width) {
    this.length = length;
    this.width  = width;
  }

  @Override double area() { return length * width; }
}
```

이와 같은 클래스 계층은 태그 지정된 클래스의 모든 단점을 해결하고, 자연스러운 계층 관계를 반영하여 유연성을 높이고 컴파일시 유형 검사를 향상 시킬수 있습니다.

<br/>

## 비정적보다 정적 멤버 클래스를 선호합니다.

<br/>

## 소스 파일을 단일 최상위 클래스로 제한합니다.