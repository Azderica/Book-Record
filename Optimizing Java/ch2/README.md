# JVM 이야기

## 인터프리팅과 클래스로딩

자바 가상 머신을 규정한 명세서에 따르면 JVM은 스택 기반의 해석 머신입니다. 레지스터는 없지만 일부 결과를 실행 스택에 보관하며 이 스택의 맨 위에 쌓인 값을 가져와 계산을 합니다.

JVM 인터프리터의 기본 로직은 평가 스택을 이용해서 중간 값들을 담아두고 가장 마지막에 실행된 명령어와 독립적으로 프로그램을 구성하는 `옵코드(opcode, 명령 코드)`를 하나씩 순서대로 처리하는 `while 루프안의 switch문` 입니다.

자바에 특정 명령을 실행시키기 위해서는 자바 `classloading` 메커니즘이 관여합니다. 자바 프로세스가 새로 초기화되면 사슬처럼 줄지어서 연결된 클래스로더가 차례차례 작동합니다. 자바 8 이전에는 jar 파일에서 가져오지만, 자바 9 이후에는 런타임이 모듈화되고 클래스로딩 개념 자체가 많이 달라졌습니다.

- 부트 스트랩 클래스로더의 주 임무는 다른 클래스로더가 나머지 시스템에 필요한 클래스를 로드할 수 있게 최소한의 필수 클래스만 로드합니다.
- 그다음 확장 클래스로더가 생성됩니다. 부트스트랩 클래스로더를 자기 부모로 설정하고 필요할 때 클래스로딩 작업을 부모에게 넘깁니다.
- 애플리케이션 클래스로더가 생성되고 지정된 클래스패스에 위치한 유저 클래스를 로드합니다.

자바는 프로그램 실행 중 처음 보는 새 클래스를 `디펜던시(의존체, dependency)`에 로드합니다. 클래스를 찾지 못한 클래스로더는 기본적으로 자신의 부모 클래스로더에게 대신 찾아보는 역할을 넘깁니다. 이렇게 올라가면서 부트스트랩도 룩업에 실패하면 `ClassNotFoundException` 예외가 발생합니다. 따라서 빌드 프로세스 수립 시 운영 환경과 동일한 클래스패스로 컴파일하는 것이 좋습니다.

보통 환경에서 자바는 클래스를 로드할 때 런타임 환경에서 해당 클래스를 나타내는 Class 객체를 만듭니다. 그런데 똑같은 클래스를 상이한 클래스로더가 두 번 로드할 가능성도 있으므로 주의해야합니다. 한 시스템에서 클래스는 패키지명을 포함한 풀 클래스명과 자신을 로드한 클래스로더, 두 가지 정보로 식별됩니다.

<br/>

## 바이트코드 실행

<br/>

## 핫스팟 입문

### JIT 컴파일이란

<br/>

## JVM 메모리 관리

<br/>

## 스레딩과 자바 메모리 모델(JMM)

<br/>

## JVM 구현체 종류

### JVM 라이선스

<br/>

## JVM 모니터링과 툴링

### VisualVM