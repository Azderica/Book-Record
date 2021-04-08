# 객체 생성과 삭제

## 생성자 대신 정적 팩토리 메서드 고려

### 정적 팩토리 메서드의 장점

다음과 같이 정적 팩토리 메서드를 통해 생성할 수 있습니다.

```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
StackWalker luke = StackWalker.getInstance(options);
```

#### 1. 정적 팩토리 메서드의 한 가지 장점은 생성자와 달리 이름이 존재한다.

- 정적 팩토리가 사용하기 쉽고, 읽기 쉬운 클라이언트 코드를 제공합니다.
- 여러 생성자가 필요하다고 판단되면, 정적 팩토리 메서드를 사용하는 것이 좋습니다.

#### 2. 생성자와 달리 호출될 때마다 새 개체를 만들 필요가 없음

- 생성된 인스턴스를 캐시하고 불필요한 중복 객체 생성을 방지하고 반복적으로 분배 가능합니다.
- 반복 된 호출에서 동일한 객체를 반환하는 정적 팩토리 메서드의 기능을 통해 클래스는 언제든지 존재하는 인스턴스를 엄격하게 제어 할 수 있습니다.

#### 3. 생성자와 달리 반환 유형의 모든 하위 유형의 객체를 반환할 수 있다.

- 유연성의 한 가지 응용 프로그램은 API가 클래스를 공개하지 않고도 객체를 반환 할 수 있다는 것입니다.
- Java 8에서는 인터페이스에 정적 메서드를 포함 할 수 없다는 제한이 제거되었으므로 편하게 사용할 수 있음

#### 4. 반환 된 개체의 클래스가 입력 매개 변수의 함수로 호출마다 다를 수 있다.

- 구현 클래스의 존재는 클라이언트에 보이지 않기 때문에 RegularEnumSet과 같은 작은 열거 유형에 대한 성능적 이점이 있습니다.

#### 5. 메서드를 포함하는 클래스가 작성될 때 반환 된 객체의 클래스가 존재할 필요가 없다.

- 유연한 정적 팩토리 메소드는 JDBC (Java Database Connectivity API)와 같은 Service provider framework 기반을 형성합니다.
- 서비스 공급자 프레임워크는 세가지 필수 구성 요소가 존재합니다.
  - 구현을 나타내는 서비스 인터페이스 (`a service interface`)
  - 공급자가 구현을 등록하느데 사용하는 공급자 등록 API (`a provider registration APi`)
  - 클라이언트가가 서비스의 인스턴스를 얻기 위해 사용하느 서비스 액세스 API (`a service access API`)
  - (선택적 네 번째 구성 요소) 서비스 제공 업체 인터페이스 (`service provider interface`)

서비스 제공 업체 프레임 워크 패턴에는 다양한 변형이 존재합니다.

- 서비스 액세스 API는 공급자가 제공하는 것보다 더 풍부한 서비스 인터페이스를 클라이언트에 반환 가능 (`Bridge 패턴`)

### 정적 팩토리 메서드의 단점

#### 1. public 또는 protected 생성자가 없는 클래스는 하위 클래스화 할 수 없다.

- Collections Framework에서 편의 구현 클래스를 하위 클래스로 만드는 것은 불가능합니다.
- 프로그래머가 상속(inheritance)보다 합성(composition) 를 사용하는 것을 장려하고, immutable types에 필요합니다.

#### 2. 프로그래머가 찾기 어렵다

- API 문서에서 눈에 띄지 않습니다.
- 생성자가 수행하므로 생상자 대신 정적 팩토리 메서드를 제공하는 클래스를 인스턴스화 하는 방법을 파악하기 어렵습니다.

다음은 대표적인 일반적인 이름입니다.

- `from`
  - 단일 매개 변수를 취하고이 유형 의 해당 인스턴스를 반환하는 유형 변환 메소드
  - `Date d = Date.from(instant)`
- `of`
  - 여러 매개 변수를 사용하고이를 통합하는이 유형의 인스턴스를 반환하는 집계 메서드
  - `Set <Rank> faceCards = EnumSet.of (JACK, QUEEN, KING);`
- `valueOf`
  - from및 of에 대한 보다 자세한 대안
  - `BigInteger prime = BigInteger.valueOf (Integer.MAX_VALUE);`
- `instance` or `getInstance`
  - 매개 변수 (있는 경우)로 설명되지만 같은 값을 가질 수없는 인스턴스를 반환
  - `StackWalker luke = StackWalker.getInstance (옵션);`
- `create` or `newInstance`
  - instance또는 getInstance. 단, 메서드가 각 호출이 새 인스턴스를 반환하도록 보장한다는 점은 예외
  - `Object newArray = Array.newInstance (classObject, arrayLen);`
- `getType`
  - getInstance비슷하지만 팩토리 메서드가 다른 클래스에있는 경우 사용
  - `FileStore fs = Files.getFileStore (경로);`
- `newType`
  - newInstance비슷하지만 팩토리 메서드가 다른 클래스에있는 경우 사용
  - `BufferedReader br = Files.newBufferedReader (경로);`
- `type`
  - get유형 과 new유형의 간결한 대안
  - `List <Complaint> litany = Collections.list (legacyLitany);`

<br/>

## 생성자 매개 변수가 많은 경우, 빌더를 고려

Static factories 와 생성자는 제한을 고유하므로, 잘 확장되지 않는다.

### telescoping constructor 패턴

- 생성자에 필수 매개 변수만 제공하고, 다른 하나에는 단일 선택적 매개 변수, 다른 하나는 두 개의 선택적 맥개 변수가 있는 등의 방식으로 생성자를 제공하는 패턴입니다.

```java
public NutritionFacts (int servingSize, int servings) { ... }
public NutritionFacts (int servingSize, int servings, int calories) { ... }
public NutritionFacts (int servingSize, int servings, int calories, int fat) { ... }
public NutritionFacts (int servingSize, int servings, int calories, int fat, int sodium) { ... }
```

- 텔레 스코핑 생성자 패턴은 작동하지만 매개 변수가 많으면 클라이언트 코드를 작성하기 어렵고 여전히 읽기가 더 어렵습니다.

### JavaBeans 패턴

이를 해결하는 방법은 setter 메소드 호출(`JavaBeans 패턴`)입니다. (이 경우는 텔레 스코핑 생성자 패턴을 해결하기에는 유리하나 단점입니다.)

- 불일치를 허용하고 가변성을 요구

```java
@Setter
@Getter
public class NutritionFacts {
  private int servingSize = -1; // 필수; 기본값 없음
  private int servings = -1; // 필수; 기본값 없음
  private int calories = 0;
  private int fat = 0;
  private int sodium = 0;
}
```

- JavaBeans 패턴은 구성이 여러 호출로 분할되기 때문에 JavaBean은 구성 과정에서 일관성없는 상태에 있을 수 있습니다.
  - 유효성을 확인하는 것으로 일관성을 유지할 수 있는 옵션이 따로 없습니다.
  - 클래스를 불변으로 만들 가능성을 배제하고 스레드 안전성을 보장하기 위해 노력이 필요합니다.

### Builder 패턴

텔레 스코핑 생성자 패턴의 안전성 + JavaBeans 패턴의 가독성을 결합

- 클라이언트는 필요한 모든 매개 변수를 사용하여 생성자 (또는 정적 팩토리)를 호출하고 빌더 객체를 가져옴
- 그런 다음 클라이언트는 빌더 개체에서 setter와 유사한 메서드를 호출하여 관심있는 각 선택적 매개 변수를 설정함
- 클라이언트는 매개 변수가없는 build메서드를 호출하여 일반적으로 변경할 수없는 개체를 생성함

```java
public class NutritionFacts {
  private final int servingSize;
  ...

  public static class Builder {
    // 필수 매개 변수
    private final int servingSize;

    // 선택적 매개 변수-기본값으로 초기화
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;

    public Builder (int servingSize, int servings) {
      this.servingSize = servingSize;
      this.servings = 서빙;
    }
    ...
  }
}

NutritionFacts cocaCola = new NutritionFacts.Builder (240, 8)
        .calories (100) .sodium (35) .carbohydrate (27) .build ();

```

- 해당 코드는 작성하기 쉽고 읽기 쉽습니다.
  - 스프링은 `@Builder` 어노테이션이 있습니다.

```java
// 클래스 계층 구조를위한 빌더 패턴

public abstract class Pizza {
  public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
  final Set <Topping> toppings;

  abstract static class Builder <T extends Builder <T >> {
    EnumSet <Topping> toppings = EnumSet.noneOf (Topping.class);

    public T addTopping (Topping topping) {
     toppings.add (Objects.requireNonNull (topping));
      return self ();
    }

    abstract Pizza build ();

    // Subclasses must override this method to return "this"
    protected abstract T self ();
  }

  Pizza(Builder<?> builder) {
    toppings = builder.toppings.clone(); // See Item  50
  }
}
```

다음과 같이 추상 self메서드 와 함께 메서드 체이닝이 캐스트 없이도 하위 클래스에서 제대로 작동합니다. 따라서 아래의 장점을 가집니다.

- 빌더 패턴은 매우 유연합니다. (반복 사용을 통해 여러 개체를 빌드할 수 있음)

다만, 이러한 단점이 있습니다.

- 개체를 만들기 위해서는 작성기를 만들어야 하기 때문에, 만드는 비용 및 성능이 중요한 상황에서 문제가 될 수 있습니다. (처음부터 시작하는 경우에 빌더를 선택하면 좋습니다.)

따라서 **빌더 패턴은 생성자 또는 정적 팩토리에 소수 이상의 매개 변수가 있는 클래스를 디자인할 때 장점**을 가집니다.
<br/>
