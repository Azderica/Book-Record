# Enums 와 Annotation

Java는 두 가지 특수 목적의 참조 유형 패밀리를 지원합니다.

- enum
- annotation

아래에서는 이를 이용하는 좋은 사례입니다.

## Item 34. 상수형 대신 열겨형을 사용합니다.

열거형은 고정 세트로 구성된 유형이며, 이는 형식에 대한 안전성을 제공합니다. Java의 열거 형 유형의 기본 개념은 `public static final field`를 통해서 각 열거 형 상수에 대해 하나의 인스턴스를 내보내는 클래스입니다. 특히 이에 대한 액세스 가능한 생성자이기 때문에 수정할 수 없습니다.

아래는 간단한 열거형(Enum) 타입입니다.

```java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
```

아래는 data와 behavior이 있는 열거형 타입입니다.

```java
// Enum type with data and behavior
public enum Planet {
  MERCURY(3.302e+23, 2.439e6),
  VENUS  (4.869e+24, 6.052e6),
  EARTH  (5.975e+24, 6.378e6),
  MARS   (6.419e+23, 3.393e6),
  JUPITER(1.899e+27, 7.149e7),
  SATURN (5.685e+26, 6.027e7),
  URANUS (8.683e+25, 2.556e7),
  NEPTUNE(1.024e+26, 2.477e7);

  private final double mass;           // In kilograms
  private final double radius;         // In meters
  private final double surfaceGravity; // In m / s^2

  // 범용 중력 상수 (m ^ 3 / kg s ^ 2
  private static final double G = 6.67300E-11;

  // Constructor
  Planet(double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }

  public double mass()           { return mass; }
  public double radius()         { return radius; }
  public double surfaceGravity() { return surfaceGravity; }

  public double surfaceWeight(double mass) {
    return mass * surfaceGravity;  // F = ma
  }
}
```

이러한 코드는 데이터를 Enum 상수와 연결하려면 인스턴스 필드를 선언하고 데이터를 가져와 필드에 저장하는 생성자를 작성해야합니다.

```java
public class WeightTable {
  public static void main (String [] args) {
    double earthWeight = Double.parseDouble (args [0]);
    double mass = earthWeight / Planet.EARTH.surfaceGravity ();

    for (Planet p : Planet.values ​​())
      System.out.printf("Weight on %s is %f %n",
        p, p.surfaceWeight (mass));
  }
}
```

Enum 상수와 메소드를 결합한 코드를 작성할 수도 있습니다.

```java
// 상수 특정 클래스 본문과 데이터가있는 열거 형 유형
public enum Operation {
  PLUS("+") {
    public double apply(double x, double y) { return x + y; }
  },
  MINUS("-") {
    public double apply(double x, double y) { return x - y; }
  },
  TIMES("*") {
    public double apply(double x, double y) { return x * y; }
  },
  DIVIDE("/") {
    public double apply(double x, double y) { return x / y; }
  };

  private final String symbol;

  Operation(String symbol) { this.symbol = symbol; }

  @Override public String toString() { return symbol; }

  public abstract double apply(double x, double y);
}
```

```java
public static void main (String [] args) {
  double x = Double.parseDouble (args [0]);
  double y = Double.parseDouble (args [1]);
  for (Operation op : Operation.values ​​())
    System.out.printf ( "%f %s %f = %f %n", x, op, y, op.apply (x, y));

  // Output
  // 2.000000 + 4.000000 = 6.000000
  // 2.000000 - 4.000000 = -2.000000
  // 2.000000 * 4.000000 = 8.000000
  // 2.000000 / 4.000000 = 0.500000
}
```

전략적으로 사용하는 enum 패턴 코드는 다음과 같습니다. (근로자 근무 급여 계산 메서드)

```java
// 전략 enum 패턴
enum PayrollDay {
  MONDAY (WEEKDAY), TUESDAY (WEEKDAY), WEDNESDAY (WEEKDAY),
  THURSDAY (WEEKDAY), FRIDAY (WEEKDAY),
  SATURDAY (WEEKEND), SUNDAY (WEEKEND);

  private final PayType payType;

  PayrollDay (PayType payType) {this.payType = payType; }

  int pay (int minutesWorked, int payRate) {
    return payType.pay (minutesWorked, payRate);
  }

  // 전략 enum 유형
  private enum PayType {
    WEEKDAY {
      int overtimePay (int minsWorked, int payRate) {
        return minsWorked <= MINS_PER_SHIFT? 0 :
          (minsWorked-MINS_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      int overtimePay (int minsWorked, int payRate) {
        return minsWorked * payRate / 2;
      }
    };

    abstract int overtimePay (int mins, int payRate);
    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay (int minsWorked, int payRate) {
      int basePay = minsWorked * payRate;
      return basePay + overtimePay (minsWorked, payRate);
    }
  }
}
```

열거형의 Switch는 상수 특정 behavior을 enum types을 늘리는데 유용합니다.

컴파일 타임에 멤버가 알려진 상수 집합이 필요할 때마다, 열거 형을 사용하는 것이 좋습니다. 다만, 열거형 유형의 상수 집합이 항상 고정되어 있을 필요는 없습니다.

이를 정리하면 다음과 같습니다. int 상수에 비해 열거형 유형의 장점은 강력합니다.
열거 형은 더 읽기 쉽고 안전하며, 강력합니다.

<br/>

## Item 35. Ordinals 대신에 인스턴스 필드를 사용합니다.

많은 열거형은 Int와 관련되어 있으며, ordinal 등을 통해서 위치를 반환할 수 있습니다. 그러나 이를 남용하면 유지보수 및 관리에서 문제가 생길 수 있습니다.

```java
// 문제가 되는 코드, 순서를 바꾸면 망함.
public enum Ensemble {
  SOLO,   DUET,   TRIO, QUARTET, QUINTET,
  SEXTET, SEPTET, OCTET, NONET,  DECTET;

  public int numberOfMusicians() { return ordinal() + 1; }
}
```

이러한 코드를 해결하는 방법은, 파생하지 않고 **인스턴스 필드에 저장**하는 것입니다.

```java
public enum Ensemble {
  SOLO (1), DUET (2), TRIO (3), QUARTET (4), QUINTET (5),
  SEXTET (6), SEPTET (7), OCTET (8), DOUBLE_QUARTET (8),
  NONET (9), DECTET (10), TRIPLE_QUARTET (12);

  private final int numberOfMusicians;
  Ensemble (int size) {this.numberOfMusicians = size; }
  public int numberOfMusicians () {return numberOfMusicians; }
}
```

<br/>

## Item 36. 비트 필드 대신에 `EnumSet`을 사용합니다.

비트 필드 표현을 통하면 비트 연산을 통해서 합집합이나 교차 집합 연산을 효율적으로 계산할 수 있습니다. 그러나 이러한 방법 들은 int 열거 형 상수 등의 단점이 있기때문에, `java.util` 패키지의 `EnumSet`을 사용하는 것이 중요합니다.

이를 사용한 코드는 다음과 같습니다.

```java
public class Text {
  public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

  // 모든 세트를 전달할 수 있지만 EnumSet은 분명히 가장 좋습니다
  public void applyStyles (Set <Style> styles) {...}
}
```

```java
text.applyStyles (EnumSet.of ( Style.BOLD, Style.ITALIC) );
```

즉, 이를 요약하면, 열거형이 집합에서 사용되기 때문에 비트 필드로 표현할 이유가 없습니다.

<br/>

## Item 37. Ordinals Indexing 대신 `EnumMap`을 사용합니다.

<br/>

## Item 38. 인터페이스로 확장 가능한 열거형을 모방합니다.

<br/>

## Item 39. Naming Patterns 보다 Annotation을 선호합니다.

<br/>

## Item 40. Override Annotation을 일관되게 사용해야합니다.
