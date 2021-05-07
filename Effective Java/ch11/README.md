# Concurrency (동시성)

동시성은 여러 활동을 동시에 진행할 수 있습니다. 동시 프로그래밍은 단일 스레드 프로그래밍보다 어렵습니다. 더 많은 문제가 발생할 수 있고 실패를 재현하기 어렵기 때문에 동시성을 피할 수 없습니다. 아래에서는 명확하고 정확하며 잘 문서화 된 동시 프로그래밍을 작성하는데 도움이 되는 자료입니다.

## Item 78. 공유된 변경 가능한 데이터는 동기화해서 사용합니다.

`synchronized` 키워드는 하나의 스레드가 한번에 방법 또는 블록을 실행함을 보장합니다. 이를 통해서, 어떤 메서드도 객체의 상태가 일관되게 됩니다.

자바 언어는 long과 double 형을 제외하고는 변수를 읽고 쓰는 것은 원자적입니다. 즉, **동기화 없이 여러 스레드가 같은 변수를 수정하므로 항상 어떤 스레드가 정상적으로 저장한 값을 읽어오는 것을 보장**합니다.

하지만, 스레드가 필드를 읽을 때 항상 수정이 완전히 반영된 값을 얻는다 보장하지만, 한 스레드가 저장된 값이 다른 스레드에서 보이는가는 보장하지 않습니다. 즉, **스레드 간의 안정적인 통신과 상호 배제를 위해서는 동기화가 필요합니다**.

이를 표현한 잘못된 코드는 다음과 같습니다.

```java
public class StopThread {
  private static boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

해당 코드의 경우, 스레드가 `start`되고 1초의 sleep후, 루프가 종료될 것으로 예상되지만 종료되지않습니다. 이는 동기화가 되지 않았기 때문입니다. 즉, 동기화가 없어지면 가상 머신이 아래처럼 수정할 수 있습니다.

```java
// 원래 코드
while (!stopRequested)
  i++;

// 최적화한 코드
if (!stopRequested)
  while (true)
    i++;
```

이는 JVM이 적용하는 **끌어올리기(hoisting, 호이스팅)**이라는 최적화 기법이 사용된 경우이며, 이는 종료되지 않습니다. 그렇기 때문에 아래처럼 고쳐서 지속적으로 동작하도록 할 수 있습니다.

```java
public class StopThread {
  private static boolean stopRequested;

  private static synchronized void requestStop() {
    stopRequested = true;
  }

  private static synchronized boolean stopRequested() {
    return stopRequested;
  }

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested())
        i++;
    });
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```

이와 같이 하면 동기화처리가 되며, 동기화는 읽기와 쓰기 모두 필요합니다. 둘 중 하나만 동기화 하는 경우에는 충준하지 않습니다.

### volatile

volatile(휘발성)은 배타적 수행과는 상관이 없지만 항상 가장 최근에 저장된 값을 읽어온다. 이론적으로는 CPU 캐시가 아닌 컴퓨터의 메인 메모리로부터 값을 읽어옵니다. 그렇기 때문에 읽기/쓰기 모두가 메인 메모리에서 수행됩니다.

```java
public class stopThread {
  private static volatile boolean stopRequested;

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

위의 코드처럼 `volatile`을 사용하면 동기화를 생략할 수 있습니다. 그러나 아래의 경우 처럼 문제가 발생할 수 있기에 조심히 사용해야합니다.

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
  return nextSerialNumber++;
}
```

코드 상으로 증가 연산자(++)는 하나지만 실제로는 volatile 필드에 두 번 접근합니다. 먼저 값을 읽고 그 다음에 1을 증가한 값과 동일한 새로운 값을을 다시 작성합니다. 따라서 두 번째 스레드가 첫 번째 스레드의 연산 사이에 들어와 공유 필드를 읽게되며, 첫번째 스레드와 같은 값을 보게될 것입니다.

이처럼 잘못된 결과를 계산해내는 오류를 안전 실패(safety failure)라고 합니다. 이 문제는 메서드에 `synchronized`를 붙이고 `violate` 키워드를 공유 필드에서 제거하면 해결됩니다.

### atomic 패키지

위의 volatile 보다 더 좋은 방법 중 하나는 `java.util.concurrent.atomic`을 사용하는 것입니다. `java.util.concurrent.atomic`의 패키지에는 락 없이도 thread-safe한 클래스를 제공합니다. `volatile`은 동기화 효과 중 통신쪽만 지원하지만, **패키지는 원자성까지 지원**하며 성능도 다른 동기화 버전에 비해 우수합니다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
  return nextSerialNum.getAndIncrement();
}
```

### 결론적으로.

가변 데이터를 공유하지 않는 것이 동기화 문제를 피하는 가장 좋은 방법입니다. 즉, **가변 데이터는 단일 스레드에서만 사용하는 것이 좋습니다.** 그리고 이에 대한 문서화를 하는 것이 중요합니다.

한 스레드가 데이터를 수정한 후에 다른 스레드에 공유할 때는 해당 객체에 공유하는 부분만 동기화해도 됩니다. 다른 스레드에 이런 객체를 건네는 행위를 `안전 발행(safe publication)`이라고 합니다. 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드 혹은 보통의 락을 통해 접근하는 필드 그리고 동시성 컬렉션에 저장하면 안전하게 발생할 수 있습니다.

여러 스레드가 변경 가능한 데이터를 공유할 때 데이터를 읽거나 쓰는 각 스레드는 동기화를 수행하는 것이 중요합니다.

<br/>

## Item 79. 과도한 동기화는 피합니다.

동기화를 하지 않으면 문제가 발생합니다. 하지만 과도한 동기화는 성능 저하, 데드락, 비결정적 동작을 유발할 수 있습니다.

### 외계인 메서드 (alien method)

이러한 응답 불가 및 안전 문제를 줄이기 위해서는, 동기화된 메서드 또는 블록 내에서 클라이언트에게 제어권을 넘기면 안됩니다. 즉, 동기화된 영액 내에서 재정의되도록 설계된 메서드 또는 클라이언트가 함수 개체의 형태로 제공하는 메서드를 호출하면 안됩니다. 이러한 **메서드는 무슨 일을 할지 모르기 때문에 예외를 발생시키거나, 교착상태를 만들거나 데이터를 훼손 할 수 있으며 이러한 메서드를 외계인 메서드(`alien method`)라고 합니다.**

```java
// Broken - 동기화 된 블록에서 외계인 메서드를 호출한 경우.
public class ObservableSet<E> extends ForwardingSet<E> {
  public ObservableSet(Set<E> set) { super(set); }

  private final List<SetObserver<E>> observers = new ArrayList<>();

  public void addObserver(SetObserver<E> observer) {
    synchronized(observers) {
      observers.add(observer);
    }
  }

  public boolean removeObserver(SetObserver<E> observer) {
    synchronized(observers) {
      return observers.remove(observer);
    }
  }

  private void notifyElementAdded(E element) {
    synchronized(observers) {
      for (SetObserver<E> observer : observers)
        observer.added(this, element);
    }
  }

  @Override public boolean add(E element) {
    boolean added = super.add(element);
    if (added)
      notifyElementAdded(element);
    return added;
  }

  @Override public boolean addAll(Collection<? extends E> c) {
    boolean result = false;
    for (E element : c)
      result |= add(element);  // Calls notifyElementAdded
    return result;
  }
}
```

```java
@FunctionalInterface
public interface SetObserver <E> {
  // 관찰 가능한 집합에 요소가 추가 될 때 호출됩니다.
  void added (ObservableSet <E> set, E element);
}
```

위 코드는 집합에 원소가 추가되면 알림을 받는 관찰자 패턴을 사용한 예제 코드입니다. 해당 코드는 `addObserver` 메서드를 호출해서 알림을 구도하고, `removeObserver` 메서드를 호출해서 구독을 취소합니다.

이를 통한 잘못된 코드는 아래와 같습니다.

```java
set.addObserver (new SetObserver <> () {
  public void added (ObservableSet <Integer> s, Integer e) {
    System.out.println(e);
      if (e == 23)
        s.removeObserver (this);
  }
});
```

<br/>

## Item 80. 스레드보다는 executors, task, stream을 애용하라.

스레드를 직접 다룰 수 있지만, `concurrent` 패키지를 이용하면 간단하게 코드를 작성할 수 있습니다.

<br/>

## Item 81. waite와 notify보다는 동시성 유틸리티를 애용하라.

이제는 `wait`와 `notify`보다 더 고수준디면 편리한 동시성 유틸리티를 사용합니다. `java.util.concurrent` 패키지의 고수준 유틸리티는 크게 실행자 프레임워크, 동시성 컬렉션, 동기화 장치로 나눌 수 있습니다.

<br/>

## Item 82. 스레드 안전성 수준을 문서화하라.

`synchronized`는 문서화와 관련이 없습니다. 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 필요하며, 또 어떤 락을 얻어야 하는지 클라이언트가 알 수 있어야합니다. 무조건 스레드 안전 클래스를 작성할 때는 비공개 락 객체를 사용해야 한다.

<br/>

## Item 83. 지연 초기화는 신중히 사용하라

"필요할 때까지는 하지말라"라는 것이 결론입니다. 지연 초기화는 양날의 검입니다. 지연 초기화(lazy initialization)은 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법인데 주로 최적화 용도로 사용됩니다.

<br/>

## Item 84. 프래그램의 동작을 스레드 스케줄러에 기대지 말라.

### 성능과 이식성이 좋은 프로그램

### 스레드는 절대 바쁜 대기 상태가 되면 안됩니다.
