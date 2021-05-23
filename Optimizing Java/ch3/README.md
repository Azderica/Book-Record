# 하드웨어와 운영체제

자바 성능을 진지하게 높일려면, 자바 플랫폼의 근간 원리와 기술에 대해 알아야합니다.

## 메모리

무어의 법칙에 따라 개수가 급증한 트랜지스터는 처음에는 클론 속도를 높이는데 쓰였습니다. 그러나 클론 속도가 증가하다 보니 프로세스 코어의 데이터 수요를 메인 메모리가 맞추기 힘들어졌습니다. 즉, 클론 속도가 올라가도 데이터가 도착할 때까지 CPU가 기다리게 됩니다.

### 메모리 캐시

이를 위해서 CPU 캐시가 등장했습니다. 레지스터보다 빠르고 메인 메모리보다 빠르며, 자주 액세스하는 메모리 위치는 CPU 캐시에 보관하자는 아이디어로 등장했습니다. 일반적으로 CPU와 가까운 캐시 순으로 L1, L2, L3 등이 있으며 L1과 L2는 전용 프라이빗 캐시입니다.

![image](https://user-images.githubusercontent.com/42582516/119114712-5a080080-ba61-11eb-8582-56e9cccdf254.png)

이를 통해서 프로세서 처리율이 높아졌으나, 캐시한 데이터를 어떻게 메모리에 다시 써야 할지 결정해야합니다. 이 문제를 해결하기 위해서 `캐시 일관성 프로토콜(cache consistency protocol)` 라는 방법으로 해결합니다.

프로세서의 가장 저수준에서 MESI 프로토콜을 사용하는데 이 MESI 프로토콜은 캐시 라인 상태를 네가지로 정리합니다.

- Modified(수정): 데이터가 수정된 상태
- Exclusive(배타): 이 캐시에만 존재하고 메인 메모리 내용과 동일한 상태
- Shared(공유): 둘 이상의 캐시에 데이터가 들어 있고 메모리 내용과 동일한 상태
- Invalid(무효): 다른 프로세스가 데이터를 수정하여 무효한 상태

정리하자면, 멀리 프로세서는 동시에 공유 상태로 존재할 수 있습니다. 하지만 어느 한 프로세서가 배타나 수정 상태로 바뀌면 다른 프로세서는 강제로 무효상태가 됩니다.

이러한 캐시 스킬을 통해서, 데이터를 신속하게 메모리에서 쓰고 읽을 수 있게 되었습니다.

캐시 하드웨어의 작동 원리 코드를 살펴볼 수 있습니다.

```java
public class Caching {
  private Final int ARR_SIZE = 2 * 1024 * 1024;
  private Final int[] testData = new int[ARR_SIZE];

  private void run() {
    System.err.println("Start: "+ System.currentTimeMillis());
    for (int i = 0; i < 15_000; i++) {
      touchEveryLine();
      touchEveryItem();
    }
    System.err.println("Warmup Finished: "+ System.currentTimeMillis());
    System.err.println("Item Line");

    for (int i = 0; i < 100; i++) {
      long t0 = System.nanoTime();
      touchEveryLine();
      long t1 = System.nanoTime();
      touchEveryItem();
      long t2 = System.nanoTime();
      long elItem = t2 - t1;
      long elLine = t1 - t0;
      double diff = elItem - elLine;
      System.err.println(elItem + " " + elLine +" "+ (100 \* diff / elLine));
    }
  }

  private void touchEveryItem() {
    for (int i = 0; i < testData.length; i++)
      testData[i]++;
  }

  private void touchEveryLine() {
    for (int i = 0; i < testData.length; i += 16)
      testData[i]++;
  }

  public static void main(String[] args) {
    Caching c = new Caching();
    c.run();
  }
}
```

위 코드를 실행하면, `touchEveryItem()` 메서드가 `touchEveryLine()` 메서드보다 더 많은 일을 할 것 같지만, 소요시간은 비슷합니다. 그 이유는 메모리 버스를 예열시키는 부분이 가장 큰 영향이 미쳐서 그렇습니다.

즉, 자바 성능을 논할 때는 객체 할당률에 따른 애플리케이션 민감도가 중요합니다.

<br/>

## 최신 프로세서의 특성

메모리 캐시가 트랜지스터를 활용하는 가장 큰 분야이지만 이외에도 여러 기술들이 나왔습니다.

### 변환 색인 버퍼(TLB)

여러 캐시에서 중요하게 사용하는 장치입니다. **가상 메모리 주소를 물리 메모리 주소로 매핑하는 페이지 테이블의 캐시 역할을 수행**합니다. 이 덕분에 가상 주소를 참조해 물리 주소에 액세스하는 빈번한 작업 속도가 매우 빨라집니다. 현재의 칩에서는 TLB는 거의 필수입니다.

### 분기 예윽과 추측 실행

분기 예측은 최신 프로세서의 고급 기법 중 하나이며 **프로세서가 조건 분기하는 기준 값을 평가하느라 대기하는 현상을 방지합니다.** 요즘의 프로세서는 다단계 명령 파이프라인을 통해서 1사이클을 여러 개발 단계로 나누어 실행합니다.

조건문을 다 평가하기 전까지 분기 이후 명령을 알수 없는 것이 문제가 되는데, 프로게세서는 잉여 트랜지스터를 사용해 발생 가능성이 큰 브랜치를 미리 결정하는 휴리스틱을 형성합니다. 추측이 맞을 때는 CPU는 다음 작업을 진행하고, 틀리면 부분적으로 실행한 명령을 모두 폐기하느 방식으로 갑니다.

### 하드웨어 메모리 모델

서로 다른 CPU가 일관되게 동일한 메모리 주소를 액세스 할 수 있을까라는 질문을 해결하기 위해서 나왔습니다.

단계를 최적화하기 위해 코드 조건에 따라 순서를 바꿀 수 있습니다. JVM은 프로세서 타입별로 상이한 메모리 액세스 일관성을 고려해서 명시적인 약한 모델로 설계되었습니다. 따라서 코드가 제대로 작동하기 위해서는 락과 volatile을 정확하게 알고 사용해야합니다.

<br/>

## 운영체제

### 스케줄러

### 시간 문제

### 컨텍스트 교환

<br/>

## 단순 시스템 모델

<br/>

## 기본 감지 전략

### 가비지 수집

### 입출력

### 기계 공감

<br/>

## 가상화

<br/>

## JVM과 운영체제
