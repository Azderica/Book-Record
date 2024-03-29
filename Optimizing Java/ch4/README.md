# 성능 테스트 패턴 및 안티패턴

해당 챕터에서는 성능 테스트에 좋지 않은 영향을 끼치는 몇 가지 일반적인 안티패턴을 설명하고, 이가 문제가 되지않도록 리팩터링하는 방법을 설명합니다.

## 성능 테스트 유형

일반적으로는 아래의 성능 테스트가 존재합니다.

- 지연 테스트(Latency test)
  - 종단 트랜잭션에 걸리는 시간은
- 처리율 테스트(Throughput test)
  - 현재 시스템이 처리 가능한 동시 트랜잭션 개수는
- 부하 테스트(Load test)
  - 특정 부하를 시스템이 감당할 수 있는가
- 스트레스 테스트(Stress test)
  - 이 시스템의 한계점은 어디까지인가
- 내구성 테스트(Endurance test)
  - 시스템을 장시간 실행할 경우, 성능 이상 증상이 나타나는가
- 용량 계획 테스트(Capacity planning test)
  - 리소스를 추가한만큼 시스템이 확장되는가
- 저하 테스트(Degradation)
  - 시스템이 부분적으로 실패할 경우 어떤일이 벌어지는가

### 지연 테스트

- 가장 일반적인 성능 테스트
- **고객이 트랜잭션을 얼마나 기다려야하는지 측정**
- 제대로 정의하지 않는다면, 목적을 잃어버립니다.

### 처리율 테스트

- 지연 테스트 다음의 일반적인 성능 테스트
- 어떤 측면에서 처리율은 지연과 동등한 개념입니다
- 지연 분포가 갑자기 변하는 점을 한계점이라고 하며, 이를 최대 처리율이라고 합니다.
- **시스템 성능이 급락하기전 최대 처리율 수치를 측정하는 것이 목표입니다.**

### 부하 테스트

- 시스템이 이정도 부하는 견딜 수 있을까에 y/n을 경정하는 과정입니다.
- 새로운 시장이나 신규 고객 유치 때, 트래픽이 상당할 때를 예측해서 진행합니다.

### 스트레스 테스트

- 시스템 여력이 어느 정도인지 알아보는 수단입니다.
- 일정한 수준의 트랜잭션(특정 처리율)을 시스템에 계속 걸어놔서, 점점 더 동시 트랜잭션이 증가하고 시스템 성능이 저하됩니다.
- **측정값이 나빠지는 시작하기 직전의 값이 바로 최대 처리율입니다.**

### 내구성 테스트

- 메모리 누수, 캐시 오염, 메모리 단편화 등은 결국 CMF가 발생합니다.
- 보통 이러한 문제를 해결하는 방법은 내구 테스트입니다.
- 평균 사용률로 시스템에 일정 부하를 계속 줘서 모니터링하다가 갑자기 리소스가 고갈되거나 시스템이 깨지는 지점을 찾습니다.
- 일반적으로 빠른 응다을 요구하는 시스템에서 많이 사용합니다.

### 용량 계획 테스트

- 스트레스 테스트와 여러모로 비슷합니다. (현재와 미래의 차이)
- 업그레이드한 시스템이 어느 정도 부하를 감당할 수 있을지 알아보는 것입니다.
- 예정된 계획의 일부분으로 실행하는 경우가 많습니다.

### 저하 테스트

- 부분 실패 테스트라고 합니다.
- 저하테스트 도중에 봐야하는 측정값은 트랜잭션 지연 분포와 처리율입니다.
- 대표적인 하위 유형으로 카오스 멍키라고 있습니다.
  - 진짜 복원성이 있는 아키텍처에서는 어느 한 컴포넌트가 잘못돼도 다른 컴포넌트까지 연쇄적으로 무너뜨리는 일은 없어야합니다.
- 운영 환경에서 라이브 프래세스를 랜덤으로 죽여보면서 테스트하며 검증합니다.

<br/>

## 기본 베스트 연습 예제

<br/>

## 성능 안티패턴 개요

<br/>

## 성능 안티패턴 카탈로그

<br/>

## 인지 편향과 성능 테스트
