# 3장 - 조건 분기

UNION은 성능적인 측면에서 큰 단점을 가지며, 쓸데없이 긴 표현이다. UNION을 사용하는 대신 CASE문을 사용하여 UNION의 문제점을 회피할 수 있다.

# 8강 UNION 을 사용한 쓸데 없이 긴 표현

- UNION은 성능적인 측면에서 굉장히 큰 단점을 가진다
- 외부적으로는 SQL 구문 1개처럼 보이지만, 내부적으로는 여러개의 SELECT가 실행된다
- 테이블에 접근하는 횟수가 많아져서 I/O 비용이 크게 늘어난다.

## UNION & CASE

- 2001 년 이전에는 세금이 제외된 가격표시
- 2002 년 이후에는 세금이 포함된 가격표시

```sql
SELECT item_name, year, price_tax_ex AS price
FROM items
WHERE year <= 2001
UNION ALL
SELECT item_name, year, price_tax_in AS price
FROM items
WHERE year >= 2002;
```

### UNION의 문제점 1

- 쓸때 없이 길다 - 거의 같은 두개의 쿼리를 두번이나 실행한다
읽기 힘들다

### UNION의 문제점 2

- 실행 계획이 복잡해진다.
- items 테이블에 2회 접근하게 된다 ( TABLE ACCESS FULL * 2)
- 테이블의 크기가 커지면 캐시히트율이 낮아지면서 더더욱 성능이 안좋아진다.

## 정확한 판단 없는 UNION 사용 회피

- UNION 은 편리하지만, 성능이 안좋다
- WHERE 구에서 조건분기를 하는 사람은 초보자다

## CASE 문으로 해결

```sql
SELECT item_name, year,
	CASE WHEN yaer <= 2001 THEN price_tax_ex
				WHEN year >= 2002 THEN price_tax_in END as price
FROM items;
```

- items 테이블에 대한 접근이 1회로 줄어든다.
- 성능이 2배 좋아졋따
- 가독성도 좋아졌다

## UNION VS CASE

- UNION을 사용한 분기는 SELECT 구문을 기본 단위로 분기하고 있다.
절차지향형식 발상
- CASE문을 사용한 분기는 “식”을 바탕으로 하는 사고
”구문” → “식” 으로 사고를 변경하는게 SQL 마스터의 길

IF문을 CASE 로 바꿔보는 사고를 해봐라

---

# 집계와 조건 분기

## UNION 방식

```sql
SELECT prefecture, SUM(pop_men) AS pop_men, SUM(pop_wom) AS pop_wom
FROM (SELECT prefecture, pop AS pop_men, null AS pop_wom
			FROM Population
			WHERE sex ='1' -- 남성
			UNION
			SELECT prefecture, NULL AS pop_men pop AS pop_wom
				FROM Population
			WHERE sex = '2') TMP -- 여성
GROUP BY prefecture;
```

- 실행계획
    - Population 테이블에 풀 스캔이 2회 수행된다
        - 만약 sex 필드에 인덱스가 존재한다면, CASE 식으로 수행되는 데이블 풀 스캔 1회 보다, 
        UNION 식 인덱스 스캔 2회가 더 빠를 수 있다

## CASE 방식

```sql
SELECT prefecture,
	SUM(CASE WHEN sex='1' THEN pop ELSE 0 END) AS pop_men,
	SUM(CASE WHEN sex='2' THEN pop ELSE 0 END) AS pop_wom
	FROM Population
GROUP BY prefecture;
```

- CASE 식을 집약 함수 내부에 포함시켜서 남성인구와 여성인구 필터를 만든다
- 실행계획
    - Population 테이블로의 풀스캔이 1회

# 집약 결과로 조건 분기

- 직원과 직원이 소속된 팀을 관리하는 테이블 Employees

- 소속된 팀이 1개라면 해당 직원은 팀의 이름을 그대로 출력한다
- 소속된 팀이 2개라면 해당 직원은 “2개를 겸무” 라는 문자열을 출력한다
- 소속된 팀이 3개 이상이라면 직원은 “3개 이상을 겸무” 라는 문자열을 출력한다.

## UNION

```sql
SELECT emp_name, MAX(team) AS team
	FROM employees
	GROUP BY emp_name
HAVING COUNT(*) = 1
UNION
SELECT emp_name, '2개를 겸무' AS team
	FROM employees
	GROUP BY emp_name
HAVING COUNT(*) = 2
UNION
SELECT emp_name, '3개 이상을 겸무' AS team
	FROM employees
	GROUP BY emp_name
HAVING COUNT(*) >= 3;
```

- 조건 분기가 레코드값이 아닌, 집합의 레코드 수에 적용이 된다
- 조건 분기가 WHERE 절이 아닌 HAVING 구에 지정되었다. ⇒ WHERE 구를 쓸때와 크게 다르지 않다.
- 실행 계획
    - Employees 테이블에 대한 접근이 3번
    - HASH GROUP BY 3회

## CASE 식을 이용한 조건 분기

```sql
SELECT emp_name,
		CASE WHEN COUNT(*) = 1 THEN MAX(team)
					WHEN COUNT(*) = 2 THEN '2개를 겸무'
					WHEN COUNT(*) >= 3 THEN '3개 시앙을 겸무'
			END AS team
	FROM Employees
GROUP BY emp_name;
```

- UNION 에 비해 비용 1/3
- GROUP BY 의 HASH 연산도 3회 → 1회
- COUNT 함수의 리턴값을 CASE 식의 입력으로 사용했기때문.
- COUNT 또는 SUM 와 같은 집약 함수의 결과는 1개의 레코드로 압축된다.
집약 함수의 결과가 스칼라로 압축 ⇒ CASE 식의 매개변수에 집약함수를 넣은것
- 실행계획
    - 테이블 풀스캔 1회
    - HASH GROUP BY 회

WHERE, HAVING 구 에서 조건 분기를 하는 사람은 초보자

# 그래도 UNION 사랑하시죠?

![m_20200423090452_vttngyed](https://github.com/RabbitHoleStudy/sql_level_up/assets/13278955/eeee22c5-8fee-41e8-b376-abfd3d98fed6)


- 머지 대상이 되는 SELECT 구문들에서 
사용하는 테이블이 다른 경우
여러개의 테이블에서 검색한 결과를 머지하는 경우

```sql
SELECT col_1
	FROM Table_A
WHERE col_2 = 'A'
UNION ALL
SELECT col_3
	FROM Table_B
WHERE col_4 = 'B';
```

- CASE 로 해결할 수 있지만, 필요없는 결합이 발생하게 된다.
- 실행 계획을 확인해보고 어떤게 더 좋은지 명확하게 확인

## UNION 이 더 좋은 경우

인덱스와 관련된 경우

UNION을 사용했을 때 좋은 인덱스(압축을 잘 하는 인덱스)를 사용하지만, 
이외의 경우는 테이블 풀 스캔이 발생한다면 
UNION을 사용한 방법이 성능적으로 더 좋을 수 있다.

```sql
SELECT key,name
				date_1, flg_1,
				date_2, flg_2,
				date_3, flg_3
		FROM ThreeElements
	WHERE date_1='2013-11-01'
		AND flg_1 = 'T'
UNION
SELECT key, name,
				date_1, flg_1,
				date_2, flg_2,
				date_3, flg_3
		FROM ThreeElements
	WHERE date_2 = '2013-11-01'
		AND flg_2 = 'T'
UNION
SELECT key, name,
				date_1, flg_1,
				date_2, flg_2,
				date_3, flg_3
		FROM ThreeElements
	WHERE date_3 = '2013-11-01'
		AND flg_3 = 'T';
```

- 머지되는 3개의 SELECT 구문에서 다른건 WHERE 구 뿐이다
- 이 쿼리를 최적의 성능으로 수행하려면 
다음과 같은 필드 조합에 인덱스가 필요하다

```sql
CREATE INDEX IDX_1 ON ThreeElements (date_1, flg_1);
CREATE INDEX IDX_2 ON ThreeElements (date_2, flg_2);
CREATE INDEX IDX_3 ON ThreeElements (date_3, flg_3);
```

- WHERE 구에서 (date_n, flg_n) 라는 필드 조합을 사용할 때 빠르게 만들어준다.
- 실행 계획
    - 3개읠 SELECT 구문 모두 INDEX RANGE SCAN이 일어난다.
    - ThreeElements 테이블의 레코드 수가 많고, 
    각각의 WHERE 구의 검색 조건에서 레코드 수를 많이 압축할수록, 
    테이블의 풀 스캔 보다도 훨씬 빠른 접근 속도를 기대할 수 있다.

## OR 을 사용한 방법

- UNION 을 쓰지 않는다면?

```sql
SELECT key, name,
			date_1, flg_1,
			date_2, flg_2,
			date_3, flg_3,
	FROM ThreeElements
WHERE (date_1 = '2013-11-01' AND flg_1 = 'T')
	OR (date_2 = '2013-11-01' AND flg_2 = 'T')
	OR (date_3 = '2013-11-01' AND flg_3 = 'T')
```

- UNION 과 결과는 같지만 실행 계획이 달라진다
    - ThreeElements 테이블에 대한 접근이 1회로 줄었다.
    - 하지만 인덱스가 사용되지 않고, 테이블 풀 스캔이 수행된다
    - WHERE 구문에서 OR 사용시 해당 필드에 부여된 인덱스를 사용할 수 없다.

⇒ 3회의 인덱스 스캔 VS 1회의 테이블 풀 스캔 중 어떤게 더 빠른지에 대한 문제

- 테이블 크기와 검색 조건에 따른 선택 비율( 레코드 히트율) 에 따라 답이 달라진다.
    - 테이블이 크고, WHERE 조건으로 선택되는 레코드의 수가 충분히 작으면 UNION

## IN 을 사용한 방법

```sql
SELECT key, name
			date_1, flg_1,
			date_2, flg_2,
			date_3, flg_3
	FROM ThreeElements
WHERE ('2013-11-01', 'T')
		IN ( (date_1, flg_1),
					(date_2, flg_2),
					(date_3, flg_3) );

```

- 다중 필드 (multiple fields)  또는 , 행식(row expression) 기능을 사용한 방법
- IN 매개변수로는 스칼라뿐 아니라, (a,b,c) 와 같은 값의 리스트(배열)을 입력할 수 있다.
- OR 을 사용할때와 성능 문제가 같다.

## CASE 를 사용한 방법

```sql
SELECT key, name
			date_1, flg_1,
			date_2, flg_2,
			date_3, flg_3
	FROM ThreeElements
WHERE CASE WHEN date_1 = '2013-11-01' THEN flg_1
					WHEN date_2 = '2013-11-01' THEN flg_2
					WHEN date_3 = '2013-11-01' THEN flg_3
					ELSE NULL END = 'T';

```

- 실행계획은 OR, IN 사용할때와 같다 ⇒ 성능적으로 같다
- 비즈니스 룰을 조금 변경하면 UNION, OR, IN 을 사용할때와 다른결과가 나온다.

# 절차 지향형과 선언형

원래 UNION이 조건 분기를 위해 만들어진 것이 아니다.

반대로 CASE 식은 조건 분기를 위해 만들어졌으므로 CASE 식을 사용하는게 더 자연스럽다.

## 구문 기반 식 기반

- SQL 초보자는 절차 지향적인 세계에 살고있다.
생각의 기본 단위는 구문(statement) 이다.
- SQL 중급자 이상은 선언적인 세계에 살고있다.
기본 단위는 식(expression)  이다.
- 이 두 세계에서는 기본적인 생각의 체계(Scheme) 가 다르다.

초보자가 UNION 을 사용해 조건 분기를 하는 이유

UNION 이라는 것 자체가 구문을 바탕으로 하는 절차 지향적인 체계를 사용한다

UNION 으로 연결하는 대상은 SELECT 구문 이다.

SQL 구문의 각 부분(SELECT, FROM, WHERE, GROUP BY, HAVING, ORDER BY) 에 작성하는 것은 모두 식이다.
SQL 구문 내부에는 식을 작성하지, 구문을 작성하지는 않는다.
