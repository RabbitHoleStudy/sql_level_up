# 2장 - SQL 기초
## 6강 - SELECT 구문

### 기본 문법

```sql
-- 필수 --
SELECT [DISTINCT] 속성이름(들)
FROM              테이블이름(들)
-- 옵션 --
[WHERE            검색조건(들)]
[GROUP BY         속성이름]    -- 일부 DBMS에는 없는 문법 -- 
[HAVING           검색조건(들)] -- HAVING은 GROUP BY가 있을 때만 사용 가능 --
[ORDER BY         속성이름 [DESC]]
```

### WHERE

- `WHERE` 절에 조건으로 사용할 수 있는 술어
    
    
    | 술어 | 연산자 | 예 |
    | --- | --- | --- |
    | 비교 | =, <, >, <=, >= | age > 20 |
    | 범위 | BETWEEN | age BETWEEN 20 AND 30 (20 이상 30 이하) |
    | 집합 | IN, NOT IN | age IN (10, 20, 30) |
    | 패턴 | LIKE | name LIKE ‘yub%’ |
    | NULL | IS NULL, IS NOT NULL | age IS NULL |
    | 복합조건 | AND, OR, NOT | (age > 20) AND (name LIKE ‘yubchoi’) |
- `%`: 0개 이상의 문자
    
    ex) `WHERE name LIKE ‘%yub%’;`:  `yub`을 포함하는 문자열
    
- `_`: 1개의 문자
    
    ex) `WHERE name LIKE ‘_u%’;`: 두 번째 위치에 `u`가 들어가는 문자열
    

### GROUP BY

`GROUP BY`를 통해 그룹을 나누면 집계가 가능하다.

| 집계 함수 이름 | 설명 |
| --- | --- |
| COUNT | 레코드 수 |
| SUM | 숫자의 합 |
| AVG | 숫자의 평균 |
| MAX | 최대값 |
| MIN | 최소값 |

```sql
SELECT floor, COUNT(*)
FROM cabinet
GROUP BY floor;
```

<img width="419" alt="groupby" src="https://github.com/RabbitHoleStudy/sql_level_up/assets/65652094/0c06ccc9-3db2-4a96-a5ca-2460bfa05ae2">

### HAVING

결과 집합에 또다시 조건을 걸어 선택하는 기능.

`HAVING`은 그룹화 또는 집계가 발생한 후 레코드를 필터링하는데 사용되고,
`WHERE`은 그룹화 또는 집계가 발생하기 전에 레코드를 필터링하는데 사용된다.

```sql
SELECT floor, COUNT(*)
FROM cabinet
GROUP BY floor
HAVING COUNT(*) >= 100;
```

<img width="418" alt="having" src="https://github.com/RabbitHoleStudy/sql_level_up/assets/65652094/40dfce43-a344-4e4b-ad37-4f2f73f668dd">


### 뷰와 서브쿼리

- `뷰`: 하나 이상의 테이블을 합하여 만든 가상의 테이블
    
    ```sql
    -- 뷰 생성 --
    CREATE VIEW 뷰이름 [필드이름(들)] AS
    
    -- 뷰 수정 --
    CREATE OR REPLACE VIEW 뷰이름 [필드이름(들)]
    AS SELECT문
    
    -- 뷰 삭제 --
    DROP VIEW 뷰이름[,...]
    ```
    
    - 삭제 후 생성보단 수정을 하도록 하자. 뷰1을 생성한 후, 뷰1로 다시 뷰2, 뷰3, … 를 만들었을 때 삭제 시 문제 발생한다.
    
    ```sql
    CREATE VIEW v_share_ban_history AS
    SELECT * FROM ban_history WHERE ban_type='SHARE';
    ```
    
- `서브쿼리`: 하나의 SQL문 안에 다른 SQL 문이 중첩된(nested) 쿼리
    
    보통 데이터가 대량일 때 데이터를 모두 합쳐서 연산하는 조인보다 필요한 데이터만 찾아서 공급해주는 서브 쿼리가 성능이 더 좋다.
    
    ```sql
    SELECT * FROM user
    WHERE user_id IN (SELECT user_id FROM ban_history);
    
    -- 서브쿼리를 상수로 전개할 수도 있다. --
    SELECT * FROM user
    WHERE user_id IN (1, 2, 3, 4, 5);
    ```
    
    - IN과 서브쿼리를 함께 사용하면 동적으로 상수 리스트가 생성되어 데이터가 바뀌어도 알아서 처리해준다.

## 7강 - 조건 분기, 집합 연산, 윈도우 함수, 갱신

### CASE
`CASE`는 식이므로 SELECT, WHERE, GROUP BY, HAVING, ORDER BY 구와 같이 어디에나 적을 수 있다.
```sql
CASE WHEN 평가식 THEN 식
     WHEN 평가식 THEN 식
     ...
     ELSE 식
END
```

### 집합 연산

- `UNION`(합집합), `INTERSECT`(교집합), `EXCEPT`(차집합)
- 중복된 레코드는 제거된다. 중복을 제외하고 싶지 않다면 `UNION ALL`과 같이 ALL 옵션을 붙인다.

### 윈도우 함수

행과 행 간의 관계를 쉽게 정의하기 위해 만든 함수이다.

```sql
SELECT WINDOW_FUNCTION (ARGUMENTS) OVER
([PARTITION BY 속성이름] [ORDER BY 속성이름])
FROM 테이블이름;
```

- `PARTITION BY`: 전체 집합을 기준에 의해 소그룹으로 나눔
    
    ```sql
    SELECT lent_type, count(*) OVER(PARTITION BY lent_type)
    FROM cabinet;
    ```
    
- `ORDER BY`: 어떤 항목에 대해 순위를 지정할지 기술
    
    ```sql
    SELECT user_id, name, RANK() OVER(ORDER BY name DESC) AS name_rank
    FROM user;
    ```
    

### 트랜잭션과 갱신

SQL의 갱신 작업은 `INSERT`(삽입), `UPDATE`(갱신), `DELETE`(제거)가 있다.

```sql
-- INSERT --
INSERT INTO 테이블이름[(속성리스트)]
VALUES (값리스트);

-- UPDATE --
UPDATE 테이블이름
SET    속성이름1=값1[, 속성이름2=값2, ...]
[WHERE <검색조건>];

-- DELETE --
DELETE FROM 테이블이름
[WHERE 검색조건];
```

- 예시
    
    ```sql
    INSERT INTO lent_history (started_at, cabinet_id, user_id)
    VALUES (NOW(), 1, 1);
    
    UPDATE lent_history SET ended_at = NOW() WHERE lent_history_id = 1;
    
    DELETE FROM lent_history WHERE user_id = 1;
    ```
    
- `DELETE`를 통해 모든 테이터를 삭제해도 테이블은 삭제되지 않는다. 테이블 자체를 삭제하고 싶다면 `DROP TABLE`을 이용한다.
