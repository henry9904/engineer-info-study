# 📗 SQLD 2과목-2 — SQL 활용 (문제풀이용 정리)

> 서브쿼리 · 집합연산자 · 그룹함수(ROLLUP/CUBE) · 윈도우함수 · TOP N. 별표(⭐) = 고빈도.

---

## 1장. 서브쿼리(Subquery) ⭐⭐

### 1-1. 개념
- 하나의 SQL문 안에 포함된 또 다른 SQL문 (메인쿼리를 보조하는 하위 쿼리)
- **위치 가능**: SELECT, FROM, WHERE, HAVING, ORDER BY 절 + INSERT VALUES / UPDATE SET
- ⚠️ **GROUP BY 절에는 서브쿼리 사용 불가**

### 1-2. 사용 시 주의 ⭐
- 반드시 **괄호**로 감싼다
- 단일행 비교연산자(`=,<,>,<=,>=,<>`)와 쓰면 서브쿼리 결과가 **1건 이하**여야 함
- 복수행 비교연산자(IN, ANY, ALL, EXISTS)는 결과 건수와 무관
- 서브쿼리엔 ORDER BY 불가 (단, ORDER BY는 메인쿼리 마지막에 / **예외: TOP-N 분석**)

### 1-3. 동작 방식에 따른 분류
| 종류 | 설명 |
|------|------|
| **비연관(Un-Correlated)** | 서브쿼리가 메인쿼리 컬럼을 **안 가짐** → 서브쿼리 먼저 실행해 값 제공 |
| **연관(Correlated)** | 서브쿼리가 메인쿼리 컬럼을 **가짐** → 메인 각 행마다 서브쿼리 실행 |

### 1-4. 위치에 따른 분류 ⭐⭐
| 종류 | 위치 | 특징 |
|------|------|------|
| **스칼라 서브쿼리** | SELECT 절 | **단일행·단일열** 반환 (하나의 값을 열처럼). OUTER JOIN과 같은 결과. 매칭 없으면 **NULL**(생략 아님!) |
| **인라인 뷰(Inline View)** | FROM 절 | 서브쿼리 결과를 **테이블처럼** 사용. **테이블 별칭 필수**. DB에 저장 안 됨(일회성) |
| **중첩 서브쿼리(Nested)** | WHERE·HAVING | 조건절에서 필터·비교용 |

### 1-5. 반환 형태에 따른 분류 ⭐
| 종류 | 연산자 |
|------|--------|
| 단일행 서브쿼리 | =, <>, >, >=, <, <= |
| 다중행 서브쿼리 | **IN, ANY, ALL, EXISTS** (단일행 연산자 불가) |
| 다중컬럼 서브쿼리 | (col1,col2) IN (…) — 개수·위치 동일해야 / **SQL Server 미지원** |

**다중행 연산자**
| 연산자 | 의미 |
|--------|------|
| IN | 결과 중 하나라도 일치 |
| ANY / SOME | 결과 중 하나라도 조건 만족 (>ANY: 최솟값보다 크면 / <ANY: 최댓값보다 작으면) |
| ALL | 모든 값이 조건 만족 (>ALL: 최댓값보다 / <ALL: 최솟값보다) |
| EXISTS | 결과가 존재하는지 여부 |

> 💡 `> ANY(10,200)` = 10보다 크면 됨(최솟값 기준) / `> ALL(10,200)` = 200보다 커야 함(최댓값 기준)

### 1-6. EXISTS vs NOT EXISTS
- **EXISTS**: 서브쿼리 결과와 겹치는 데이터만 메인쿼리에 출력
- **NOT EXISTS**: 서브쿼리 결과를 제외한 나머지(차집합) 출력

> ⚠️ **인라인 뷰**: 다른 테이블과 조인하려면 서브쿼리 결과 컬럼이 서브쿼리 안에 포함돼 있어야 하고, 집계함수 결과를 WHERE에서 쓰려면 **별칭 필수**(WHERE엔 집계함수 불가하므로).

---

## 2장. 집합 연산자 ⭐⭐

### 2-1. 개념
- 여러 SELECT 결과를 하나의 집합으로 간주해 합/교/차집합 연산
- 조건: **컬럼 수 일치, 각 컬럼 데이터 타입 상호 호환** / 컬럼명·데이터타입은 **첫 번째 집합**이 결정 / 컬럼 사이즈는 달라도 됨

| 연산자 | 기능 | 중복 |
|--------|------|------|
| **UNION** | 합집합 | 중복 **1번만**(내부 정렬 수행) |
| **UNION ALL** | 합집합 | 중복 **모두** 반환(정렬 X, 더 빠름) |
| **INTERSECT** | 교집합 | 중복 제거 |
| **MINUS**(Oracle) / **EXCEPT**(SQL Server) | 차집합 | 앞 집합에만 존재하는 행 |

> ⚠️ **함정 1**: MINUS(EXCEPT)는 **집합 순서에 따라 결과 다름** (A-B ≠ B-A)
> ⚠️ **함정 2**: 개별 SELECT문에 **ORDER BY 불가**(맨 아래 집합의 ORDER BY만 전체에 적용)
> ⚠️ **함정 3**: 컬럼 순서가 달라도 오류는 안 나지만 의미 없는 결과 나옴

**1:1 관계 테이블 특성**
- EXCEPT 결과는 항상 공집합
- INTERSECT = JOIN 결과와 같음, INTERSECT = UNION 결과와 같음
- UNION ALL 결과 건수 = 한 테이블 전체건수 × 2 (UNION 건수는 동일)

---

## 3장. 그룹 함수 ⭐⭐⭐

### 3-1. 3가지 데이터 분석 함수
집계함수(Aggregate) / 그룹함수(Group) / 윈도우함수(Window)

### 3-2. 집계함수 & NULL (복습)
- COUNT/SUM/AVG/MIN/MAX/STDDEV/VARIANCE — **NULL 무시**
- ⚠️ **AVG 주의**: 전체 대상 평균 원하면 `SUM/COUNT(*)` 또는 `AVG(NVL(SAL,0))`
- MIN/MAX는 날짜·문자에도 사용

### 3-3. ROLLUP ⭐
- 그룹핑 컬럼 N개 → **N+1 Level**의 소계(Subtotal) 생성
- ⚠️ **계층 구조 → 인수 순서 바뀌면 결과 달라짐**
- `ROLLUP(A,B)` → A·B별 소계 / A별 소계 / 총계
- 괄호로 묶으면(`ROLLUP(A,(B,C))`) 하나의 집합으로 간주

### 3-4. CUBE ⭐
- 결합 가능한 **모든 경우**에 대해 다차원 소계 생성
- ⚠️ **평등 관계 → 인수 순서 바뀌어도 결과 동일**(정렬 순서만 다름)
- ROLLUP보다 시스템 부담 큼. `CUBE(A,B)` → A·B별 / A별 / **B별** / 총계

### 3-5. GROUPING SETS ⭐
- **평등 관계**(순서 무관), **총계 자동 출력 X**
- `GROUPING SETS(A,B)` → A별 소계 / B별 소계 (총계 없음)
- 총계 원하면 `GROUPING SETS(A, B, ())` 또는 `(A, B, NULL)`

| 표현식 | 출력값 | 순서 |
|--------|--------|------|
| ROLLUP(A,B) | A·B소계 / A소계 / 총계 | 계층(순서 O) |
| CUBE(A,B) | A·B / A / B / 총계 | 평등(순서 X) |
| GROUPING SETS(A,B) | A소계 / B소계 | 평등(순서 X) |

> 💡 세 함수 모두 **UNION ALL로 대체 가능**, 정렬 필요 시 ORDER BY 명시

### 3-6. GROUPING 함수 ⭐⭐⭐
- ROLLUP/CUBE의 **소계 행이면 1, 아니면 0** 반환
- 용도: 소계 행 NULL을 다른 값으로 표시
```sql
CASE WHEN GROUPING(DNAME)=1 THEN 'All Departments' ELSE DNAME END
-- 또는 DECODE(GROUPING(DNAME), 1, 'All Departments', DNAME)
```

---

## 4장. 윈도우 함수(Window Function) ⭐⭐⭐

### 4-1. 개념
- 집계함수와 달리 **각 행을 유지하면서** 그룹 내 연산 수행
- **GROUP BY 없이** 그룹 연산 가능, 조인·서브쿼리 없이 행 간 비교/연산
```sql
윈도우함수(컬럼) OVER (
  [PARTITION BY 컬럼]         -- 그룹 (=GROUP BY 컬럼)
  [ORDER BY 컬럼 ASC|DESC]    -- 순위함수·누적 시 필수
  [ROWS|RANGE BETWEEN A AND B] -- 연산 범위 (SQL Server 미지원)
) AS result
```
> ⚠️ 순위함수는 ORDER BY 필수 / 집계함수는 누적값 출력 시 ORDER BY 사용

### 4-2. ROWS vs RANGE ⭐
- **ROWS**: 값이 같아도 각 **행씩** 연산
- **RANGE**: 값이 같으면 하나의 범위로 묶어 **동시** 연산 (**DEFAULT**)

**연산 범위 (BETWEEN A AND B)**
- 시작(A): CURRENT ROW / **UNBOUNDED PRECEDING(처음부터, DEFAULT)** / N PRECEDING
- 끝(B): **CURRENT ROW(현재까지, DEFAULT)** / UNBOUNDED FOLLOWING(마지막까지) / N FOLLOWING

### 4-3. 순위 함수 ⭐⭐
| 함수 | 동일 값 처리 |
|------|-------------|
| **ROW_NUMBER()** | 무조건 1,2,3… 순차 (동점 없음) |
| **RANK()** | 동점 같은 순위, 다음 순위 **건너뜀** (1,2,2,4) |
| **DENSE_RANK()** | 동점 같은 순위, 다음 순위 **안 건너뜀** (1,2,2,3) |

- 특정 값의 순위: `RANK(값) WITHIN GROUP (ORDER BY 컬럼)` — 이건 윈도우 아닌 일반함수

### 4-4. 행 순서 함수 (SQL Server 미지원)
| 함수 | 기능 |
|------|------|
| **LAG(컬럼, N, 대체값)** | N개 **이전** 행 값 (기본 N=1) |
| **LEAD(컬럼, N)** | N개 **이후** 행 값 |
| **FIRST_VALUE / LAST_VALUE** | 범위 내 처음/마지막 값 |
- LAG/LEAD는 ORDER BY 필수. NULL 대체는 세 번째 인수.
- ⚠️ LAST_VALUE는 기본 범위가 '처음~현재'라 원하는 값이 안 나올 수 있음 → 범위 지정 or DESC 정렬

### 4-5. 비율 함수 (SQL Server 미지원) ⭐
| 함수 | 기능 | 범위 |
|------|------|------|
| **RATIO_TO_REPORT(컬럼)** | 파티션 SUM 대비 행 값 비율 | 0< x ≤1, ORDER BY 불가 |
| **PERCENT_RANK()** | (순위-1)/(총행수-1), 백분율 | 0≤ x ≤1, ORDER BY 필수 |
| **CUME_DIST()** | 현재 행 이하 누적비율 | 0< x ≤1, ORDER BY 필수 |
| **NTILE(N)** | N개 그룹으로 분할(안 나눠지면 앞 그룹이 큼) | ORDER BY 필수 |

---

## 5장. TOP N 쿼리 ⭐

### 5-1. 개념
- 전체 결과에서 **상위 N개 행** 추출 (페이징에 활용)

### 5-2. ROWNUM (Oracle) ⭐⭐
- 출력 데이터에 순차 행 번호 부여 (가상 컬럼)
- ⚠️ **절대적 번호 아님** → 특정 행 지정 불가
- ⚠️ 첫 행이 할당된 이후 증가하므로 **`>` 연산 불가** (`ROWNUM > 1`은 결과 없음!)
- `WHERE ROWNUM <= N` 형태로만 상위 N개 추출 가능
- 정렬된 상위 N개: **인라인 뷰로 먼저 ORDER BY 후 ROWNUM 적용**
```sql
SELECT * FROM (SELECT * FROM EMP ORDER BY SAL DESC) WHERE ROWNUM <= 3;
```

### 5-3. 기타 방법
- SQL Server: `TOP(N)` / `OFFSET-FETCH`
- Oracle 12c+: `FETCH FIRST N ROWS ONLY`
- `ROW_NUMBER() OVER(ORDER BY …)` 활용 (범용)

---

## ✅ SQL 활용 확인문제

**Q1.** 서브쿼리를 사용할 수 없는 절은?
<details><summary>정답</summary>GROUP BY 절</details>

**Q2.** FROM 절에 오는 서브쿼리의 이름은? 필수 조건은?
<details><summary>정답</summary>인라인 뷰(Inline View), 테이블 별칭 필수</details>

**Q3.** UNION과 UNION ALL의 차이는?
<details><summary>정답</summary>UNION은 중복 제거+정렬, UNION ALL은 중복 포함(정렬X, 더 빠름)</details>

**Q4.** `ROLLUP(A, B)`가 만드는 소계 종류를 쓰시오.
<details><summary>정답</summary>A·B별 소계 / A별 소계 / 총계 (N+1=3 Level)</details>

**Q5.** 인수 순서를 바꿔도 결과가 같은 그룹 함수는? (ROLLUP / CUBE / GROUPING SETS)
<details><summary>정답</summary>CUBE, GROUPING SETS (평등 관계). ROLLUP은 계층이라 순서 영향</details>

**Q6.** 급여 순위에서 동점 처리 시 다음 순위를 건너뛰지 않는 함수는?
<details><summary>정답</summary>DENSE_RANK() (1,2,2,3). RANK()는 건너뜀(1,2,2,4)</details>

**Q7.** 이전 행의 값을 가져오는 윈도우 함수는?
<details><summary>정답</summary>LAG() (이후 행은 LEAD)</details>

**Q8.** Oracle에서 `WHERE ROWNUM > 1`의 결과는? 왜?
<details><summary>정답</summary>결과 없음(0건). 첫 행이 ROWNUM=1을 못 받으면 다음 행도 못 받으므로 > 연산 불가</details>

**Q9.** GROUPING(컬럼)이 1을 반환하는 경우는?
<details><summary>정답</summary>ROLLUP/CUBE의 소계(집계) 행일 때</details>

**Q10.** 급여가 높은 상위 3명을 뽑는 Oracle 쿼리 구조는?
<details><summary>정답</summary>SELECT * FROM (SELECT * FROM EMP ORDER BY SAL DESC) WHERE ROWNUM <= 3;</details>
