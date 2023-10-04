# 2장 - SQL 기초

# 6강 SELECT 구문

- 데이터베이스의 핵심은 검색 이다
- 필요한 데이터를 뽑아내는 행위를 질의 또는 추출이라고도 한다
- 검색을 위해 사용되는게 SELECT 구문이다.

```sql
SELECT name,phone_nbr, address, sex, age
FROM Address
```

## SELECT & FROM

- SELECT 구문은 두 부분으로 구성
    - SELECT 구
        - 테이블이 갖고 있는 필드 일 경우, 로 연결해서 사용
        - 수식
    - FROM 구
        - FROM [테이블 이름]
        - 테이블에서 데이터를 검색하는경우 FROM 필수
        - SELECT 1 처럼 상수를 선택하는 경우 불필요
- SELECT 구문에는 데이터를 ‘어떤 방법’으로 선택할지 나타내지 않는다.
사용자가 생각해야 하는 부분은 어떤 데이터가 필요한지 정도

## WHERE 구

```sql
SELECT name, address
	FROM Address
WHERE address = '경기도';
```

- 엑셀의 필터 조건과 같다
- 어디? 가 아니라 ~ 라는 경우 를 나타내는 관계부사다

### WHERE 구의 다양한 조건 지정

| 연산자 | 의미 |
| --- | --- |
| = | ~ 와 같음 |
| < >  | ~와 같지 않음 |
| ≥ | ~ 이상 |
| ≤ | ~ 이하 |
| < | ~ 보다 작음 |
- WHERE 구는 거대한 벤다이어그램이다
- 주소가 서울시 이며, 나이가 30세 이상 을 찾으려면 다음과 같다

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/1dc14d02-9fef-47d5-828a-c667c7d13337/30e307b3-9a80-4aa9-9ef9-e18a157dad8a/Untitled.png)

```sql
SELECT name, address.age
	FROM address
WHERE address = '서울시'
	AND age >= 30;
```

- 교집합

### OR

- 서울시 또는 30세 이상
- 합집합

```sql
SELECT name, address.age
	FROM address
WHERE address = '서울시'
	OR age >= 30;
```

### IN 으로 OR 조건을 간단하게 작성

```sql
SELECT name, address
	FROM Address
WHERE address = '서울시',
OR address = '부산시',
OR address = '인천시';
```

위의OR 문을 아래로 개선할 수 있다.

```sql
SELECT name, address
	FROM Adress
WHERE address IN ('서울시', '부산시', '인천시');
```

### NULL

- 특정 컬럼이 NULL 인 레코드를 찾기 위해서는 =NULL 이 아닌 IS NULL 을 써야 한다

```sql
SELECT name, phone_nbr
	FROM Address
WHERE phone_nbr IS NULL;
```

### SELECT 구문은 절차 지향형 언어의 향수

- 입력을 받고 관련된 처리를 수행 한 뒤에 리턴하는 것.
- SELECT 구문도 테이블이라는 입력을 FROM 구로 받아, 특정 출력을 리턴한다
- SELECT 구문의 입력과 출력의 자료형은 테이블(관계)다.
입력도 출력도 모두 2차원 표
이외에는 어떤 자료형도 존재하지 않는다.

⇒ 폐쇄성(closure property)

## GROUP BY

- 테이블에서 데이터를 선택 + 합계 또는 평균 등의 집계 연산을 표현

```sql
SELECT sex, COUNT(*)
	FROM Address
GROUP BY sex;

sex | count
--------------
남   |  4
여   |  5
```

- SQL 의 대표적인 집계 함수

| 함수 이름 | 설명 |
| --- | --- |
| COUNT | 레코드 수를 계산 |
| SUM | 숫자를 더함 |
| AVG | 숫자의 평균을 구함 |
| MAX | 최대값을 구함 |
| MIN | 최솟값을 구함 |

## HAVING 구

```sql
SELECT address, COUNT(*)
	FROM Address
GROUP BY address
HAVING COUNT(*) = 1;
```

- 살고 있는 사람수(레코드 수) 가 한 명 뿐인 주소 필드(address)를 선택
- WHERE 구가 레코드에 조건을 지정한다면, HAVING 구는 집한에 조건을 지정하는 기능

## ORDER BY 구

- 순서를 지정

```sql
SELECT name, phone_nbr, address, sex, age
FROM Address
ORDER BY age DESC;
```

- 나이순으로 내림차순 정렬
- ASC (오름차순)이 기본값 DBMS 의 공통 규칙

## 뷰와 서브쿼리

- SELECT 구문 중에서도 자주 사용하는 것과 거의 사용하지 않는것이 나온다.
자주 사용하는 SELECT 구문은 따로 저장하고 싶을때 View 기능을 사용할 수 있다.
- 하지만 테이블과 달리 내부에 데이터를 보유하지 않기때문에, SELECT 구문을 저장한 것 뿐이다

### 뷰 만들기

```sql
CREATE VIEW [뷰 이름]([필드이름 1],[필드 이름2] ... ) AS
```

```sql
CREATE VIEW CountAddress(v_address, cnt)
AS
SELECT address, COUNT(*)
	FROM Address
GROUP BY address;

--------------------
SELECT v_address, cnt
	FROM CountAddress;
```

### 익명 뷰

```sql
뷰에셔데이터를선택
SELECT V_address,⊂nt
FROM CountAddress;

SELECT v_address, cnt
FROM(SELECT address AS v_address, COUNT(*) AS ⊂nt
	FROM Address
GROUP BY address) AS CountAddress;

```

- 이렇게 FROM 구에 직접 지정하는 SELECT 구문을 Subquery 라고 한다.

### 서브쿼리로 조건 설정

```sql
SELECT name
	FROM Address
WHERE name IN (SELECT name FROM Address2);
```

Address 2의 결과로 부터 나온 이름 과 일치하는 Address 의 이름

---

# 조건분기 CASE문

```sql
CASE WHEN address = '서울시' THEN '경기'
			WHEN address = '인천시' THEN '경기'
			WHEN address = '부산시' THEN '영남'
			WHEN address = '속초시' THEN '관동';
			WHEN address = '서귀포시' THEN '호남'
			ELSE NULL AND AS distinct
FROM Address;
```

# 집합연산

## UNION 합집합

```sql
SELECT * FROM Address
UNION
SELECT * FROM Address2;
```

- 중복 제외
- 중복 출력을 하려면 `UNION ALL`
- 합치려는 테이블 두개가 스키마가 같아야 한다.

## INTERSECT 교집합

```sql
SELECT * FROM Address
INTERSECT
SELECT * FROM Address2;
```

- 중복 제외

## EXCEPT 차집합

```sql
SELECT * FROM Address
EXCEPT
SELECT * FROM Address2;
```

- `Address - Address2`
- 교환 법칙 미 성립

# 윈도우 함수

## OVER (PARTITON BY address) - 집약없는 그룹화

```sql
SELECT address,
COUNT(*) OVER (PARTITION BY Address)
FROM Adress;

```

- GROUP BY 와 달리 집약 기능 사용 X

## RANK 순위 구하기

```sql
SELECT name,
			age,
			RANK() OVER (ORDER BY age DESC) AS rnk
FROM Adress;

```

- 값이 같을때 순위를 같이 표시하고, 공동 순위를 건너 뛴 만큼  다음 숫자는
- DENSE_RANK 를 사용하면 공동순위를 표시

# 트랜젝션과 갱신

## 갱신 INSERT

```sql
INSERT INTO Person (name, age, nickname)
VALUES ('홍길동', 99, 'gil-dong')
```

- 다중갱신

```sql
INSERT INTO Person (name, age, nickname)
VALUES ('홍길동', 99, 'gil-dong'),
('김두한', 4, 'four-dollar'),
('심영',30,'goza');
```

## 제거 DELETE

```sql
DELETE FROM Person;
```

- 테이블의 모든 레코드 제거

```sql
DELETE FROM Person WHERE name = '김두한';
```

- 특정 필드의 값을 필터링 해서 제거

- 일부 필드값만 가진 레코드 모두 제거는 불가능

## 갱신 UPDATE

```sql
UPDATE Person
SET name = '아무개'
WHERE name = '김두한';
```

- 다중 컬럼 값 갱신
