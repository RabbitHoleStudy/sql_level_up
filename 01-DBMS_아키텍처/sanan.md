## DBMS 아키텍처 개요

- `쿼리 평가 엔진`
    
    SQL 구문을 분석하고, 어떤 순서로 접근할지 `실행 계획`을 결정한다.
    
    이 실행 계획에 기반을 두어 `접근 메서드`를 수행한다.
    
- `버퍼 매니저`
    
    특별한 용도로 사용하는 메모리 영역을 관리하는 매니저.
    
- `디스크 용량 매니저`
    
    데이터 I/O를 제어한다.
    
- `트랜잭션 매니저`와 `락 매니저`
    
    데이터의 Lock, Transaction을 관리한다.
    
- `리커버리 매니저`
    
    데이터를 정기적으로 백업, 장애시 복구한다.
    

---

## DBMS와 버퍼


저장량과 속도는 트레이드 오프 관계.

- 기본적으로 DBMS에는 `데이터 캐시`가 있으므로, 히트시에 SELECT의 속도가 빠르다.
- 갱신 관련(INSERT, DELETE, UPDATE, MERGE.. DDL)시에 SQL 구문을 받으면 곧바로 수행하지 않고 `로그 버퍼`에 변경 정보를 보낸 후, 디스크에 변경을 수행한다.
    
    → 갱신은 비동기로 이루어진다.
    

비동기적으로 이뤄지기에, `로그 버퍼`가 날라가는 것 또한 발생할 수 있다는 점을 알아야 한다.

이를 회피하고자 커밋(갱신 처리 확정, 영속화) 시점에 반드시 갱신 정보를 로그 파일에 쓰고, 장애 발생시에도 정합성을 유지하도록 한다.

→ 커밋 때에는 반드시 디스크에 동기적인 접근이 일어난다.

- `워킹 메모리`
    
    SQL에서 정렬 또는 해시가 필요한 때에 사용되고, 해제되는 영역이다. 여러 SQL을 동시에 받아내기 때문에 넘치는 경우에는 Swap을 한다.(이를 위한 별도의 tmp 영역 또한 존재한다)
    

---

## DBMS와 실행 계획

프로그래밍 언어와는 다르게, SQL을 이용하여 DBMS에게 작업을 위임하는 이 방식은 일일히 절차적인 세부사항을 지정해주지 않아도 수행된다.

즉, 사용자는 RDB를 비절차적으로 DBMS를 통해 어떻게(How)를 기술하는 것이 아닌 대상(What)을 기술하여 사용한다.

쿼리 평가 엔진

1. `파서`로 구문 분석을 한다.
2. `옵티마이저`(최적화)에서 데이터 접근법(`실행 계획`)을 설정한다.
    
    → 인덱스 유무 / 데이터 분산 및 편향도 / DBMS 내부 매개변수 등을 고려해서 선택 가능한 `실행 계획들을 작성`하고, 그 `비용을 연산`한다.
    
3. `옵티마이저`가 계획을 세울 때, DBMS의 내부 정보들을 모아놓은 테이블인 `카탈로그`를 이용한다. 테이블, 인덱스의 **통계 정보**가 저장되어 있다.
4. `옵티마이저`가 세운 여러 계획 중, 최적의 실행 결과를 선택하는 것이 플랜 평가이다. 실행 계획은 DBMS가 실행할 수 있는 코드라기보다, 사람이 읽을 수 있는 계획서에 가깝다. 따라서 DBMS는 실행 계획을 절차적 코드로 변환하고, 작업을 수행한다.

사용자는 옵티마이저를 잘 쓰는 것이 중요하다. 따라서, 카탈로그에 대해서 항상 신경써야 할 것이다. 대부분의 옵티마이저가 실패하는 경우는 통계 정보(카탈로그)가 부족하기 때문이다.

카탈로그가 갖는 정보들은 다음과 같은 것들이 있다.

- 각 테이블의 레코드 수
- 각 테이블의 필드 수와 필드의 크기
- 필드의 카디널리티(값의 개수)
- 필드 값의 히스토그램(분포)
- 필드 내부에 있는 NULL의 수
- 인덱스 정보

이러한 정보들이 DDL등으로 인하여 테이블이 변경되었음에도 그 변경사항이 반영되지 않는다면 옵티마이저가 잘못된 계획을 세울 가능성이 높아진다.

---

## 실행 계획이 SQL 구문의 성능을 결정

실행 계획은 DBMS별로 보이는 것이 다르더라도 공통적으로 3가지 요소를 갖는 포맷으로 구성된다.


- 조작 대상 객체(`Shops` / `SHOPS`)
- 객체에 대한 조작의 종류(`Seq Scan` / `TABLE ACCESS FULL`)
- 조작 대상이 되는 레코드 수(`60`)

위 3가지는 거의 모든 실행 계획에 포함되어 있다.

옵티마이저는 실제 테이블을 바라보지 않고, 단순히 통계 정보(카탈로그)를 믿는다.

일반적으로 DBMS에서 JOIN(결합)을 할 때, 세 가지 종류의 알고리즘을 사용한다.

- Nested Loops
- Sort Merge
- Hash
    
    

실행 계획은 일반적으로 트리 구조다. 중첩 단계가 높을 수록 먼저 실행된다.

따라서, 위의 그림을 보아도 인덱스 스캔이 먼저 이뤄지는 것을 알 수 있다.

한편 같은 중첩 단계에서는 위에서 부터 실행한다.

`INDEX UNIQUE SCAN` - `TABLE ACCESS FULL` - `TABLE ACCESS BY INDEX ROWID` - `NESTED LOOPS` - `SELECT STATEMENT`

---

## 실행 계획의 중요성

옵티마이저 자체는 우수하긴 하지만, 항상 잘 하는 것은 아니다.

애초에 옵티마이저에게 정보를 제대로 주지 못하는 경우(인덱스를 안쓰거나 조인 순서를 이상하게 쓰거나) 있을 수도 있다.

이런 경우에 `실행계획을 수동으로 변경`하는 최후의 수단으로 사용할 수 있다. 이를 `쿼리 힌트`라고 한다.
