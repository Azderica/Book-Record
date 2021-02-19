# 1장. 오브제트와 의존관계

## DAO

### 용어 정리

- DAO
  - DAO(Data Access Object)는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트
- JavaBean(자바 빈)
  - 비주얼 툴에서 조작 가능한 컴포넌트
  - 두 가지 조건이 필요
    - 디폴트 생성자 : 파라미터가 없는 디폴트 생성자가 필요
    - 프로퍼티 : 자바빈이 노출하는 이름을 가진 속성, 접근자(set/get)을 통해 수정 및 조회 가능

### JDBC 작업의 순서

1. DB 연결을 위한 Connection을 가져온다
2. SQL을 담은 Statement를 만든다
3. 만들어진 Statement를 실행한다
4. 조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 Object에 옮겨준다.
5. 작업 중에 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫는다.
6. JDBC API가 만들어내는 예외를 잡아서 직접 처리하거나, 메소드에 throws를 선언해서 예외가 발생시 메소드 밖으로 던집니다.

### 코드를 통한 리팩토링
