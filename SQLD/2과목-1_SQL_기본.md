# 📗 SQLD 2과목-1 — SQL 기본 (문제풀이용 정리)

> 개념 → ⚠️ 시험 함정 → ✅ 확인문제. Oracle 기준, SQL Server 차이는 별도 표기.

---

## 1장. 관계형 데이터베이스 개요

### 1-1. 기본 용어
- **DBMS**: 데이터를 효율적으로 관리·복구하는 시스템 (예: MySQL, ORACLE)
- **RDB(관계형DB)**: 1970년 **E.F. Codd** 논문에서 시작. 데이터를 **테이블**로 관리, **SQL**로 관리
- **테이블**: 행·열의 2차원 구조 저장 객체(=릴레이션)
- **컬럼/열** = 필드(Field), **로우/행** = 레코드(Record) = 튜플(Tuple)

### 1-2. SQL 명령어 종류 ⭐⭐
| 종류 | 명령어 | 설명 | COMMIT |
|------|--------|------|--------|
| **DML**(데이터 조작어) | SELECT, INSERT, UPDATE, DELETE, MERGE | 데이터 조회·변경 | 사용자가 COMMIT 해야 함 → **ROLLBACK 가능** |
| **DDL**(데이터 정의어) | CREATE, ALTER, DROP, RENAME, TRUNCATE | 구조 정의 | **AUTO COMMIT → ROLLBACK 불가** |
| **DCL**(데이터 제어어) | GRANT, REVOKE | 권한 부여·회수 | |
| **TCL**(트랜잭션 제어어) | COMMIT, ROLLBACK, SAVEPOINT | 트랜잭션 제어 | |
| (DQL) | SELECT | 데이터 질의 | |

> ⚠️ **함정 1**: **TRUNCATE는 DDL** → 오토커밋, 롤백 불가. (DELETE는 DML, 롤백 가능!)
> ⚠️ **함정 2**: DDL은 AUTO COMMIT. DML은 사용자가 COMMIT 해야 반영·롤백 가능.
> ⚠️ **함정 3**: MERGE는 DML, SAVEPOINT는 TCL.

### 1-3. SQL 실행 3단계
1. **파싱(Parsing)**: 문법·구문 분석, 결과를 Library Cache에 저장
2. **실행(Execution)**: 옵티마이저가 만든 최적 실행계획대로 수행
3. **인출(Fetch)**: 결과를 사용자에게 전송

### 1-4. 데이터 무결성(Integrity) & 제약조건 ⭐⭐
**관계 모델 4대 제약(무결성)**
| 제약 | 대상 | 핵심 |
|------|------|------|
| **도메인 무결성** | 속성(Attribute) | 값이 원자성 + 도메인 정의 범위 내 |
| **개체 무결성** | 기본키(PK) | PK는 **NOT NULL & UNIQUE** |
| **참조 무결성** | 외래키(FK) | FK는 NULL 이거나, 참조 테이블에 **실제 존재하는 값** |
| **키 무결성** | 릴레이션/테이블 | 테이블은 키(PK)를 가져야 함 |

> ⚠️ **개체 무결성 = PK는 NULL 불가 & 중복 불가.**
> ⚠️ **참조 무결성**: 없는 값을 FK로 참조 → 위반 / 참조되는 값(PK) 삭제 시 → 위반
> 💡 FK는 자기 자신이 속한 릴레이션도 참조 가능(예: 멘토, 상사).

---

## 2장. SELECT 문

### 2-1. SELECT 6개 절과 순서 ⭐⭐⭐
**작성(문법) 순서**
```
SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY
```
**처리(논리적 실행) 순서**
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```
> 💡 **암기**: "FWGHSO(에프-위-그-하-쓰-오)". SELECT는 거의 마지막(ORDER BY 직전)에 실행!
> ⚠️ 이 실행순서 때문에: **컬럼 별칭(ALIAS)은 ORDER BY에서만 사용 가능**. WHERE·GROUP BY·HAVING에서는 별칭 사용 시 에러(SELECT보다 먼저 실행되므로).

### 2-2. SELECT 절
- `*` : 모든 컬럼 / `SELECT 컬럼1, 컬럼2 ...`
- **표현식**: 연산식·함수 변형식 (예: `급여 * 1.1`)
- **ALL**(기본): 중복 포함 모두 출력 / **DISTINCT**: 중복 제거 1건만

### 2-3. AS (별칭, Alias) ⭐
- 컬럼·테이블에 임시 이름 부여, 결과에만 적용
- **ORDER BY절에서만 컬럼 별칭 사용 가능** (WHERE/HAVING에서 사용 시 에러)
- ⚠️ **오라클에서 테이블 별칭 명시할 땐 AS 사용 불가!** (컬럼 별칭엔 AS 가능)
- 큰따옴표(" ") 사용 경우: 공백 포함 / 특수문자 포함 / 별칭 그대로 전달

### 2-4. FROM 절
- 여러 테이블 전달 가능(콤마 구분)
- **조인 조건 없으면 → Cartesian Product**(모든 조합 생성)
- 테이블 별칭 사용 시, 참조할 때 **반드시 별칭 사용**
- ⚠️ **ORACLE은 FROM 절 생략 불가**(23c부터 가능) → 필요 없으면 **DUAL** 테이블 사용
- SQL Server는 FROM 생략 가능

**DUAL 테이블**: SYS 소유, 모든 사용자 접근 가능, `DUMMY`(VARCHAR2(1)) 컬럼에 `'X'` 1건. 단순 연산·함수 조회용.

---

## 3장. 함수

### 3-1. 단일행 vs 다중행 ⭐
- **단일행 함수(Single-Row)**: 각 행마다 1:1로 결과 반환 (문자·숫자·날짜·변환·NULL 함수)
- **다중행 함수(Multi-Row)=집계함수**: 여러 행 → 하나의 결과 (SUM/AVG/COUNT/MAX/MIN). GROUP BY와 궁합.

### 3-2. 숫자 함수
| 함수 | 기능 | 예 → 결과 |
|------|------|-----------|
| ABS(x) | 절댓값 | ABS(-1)=1 |
| SIGN(x) | 양수1/음수-1/0은 0 | SIGN(100)=1 |
| CEIL(x) | 크거나 같은 최소 정수(올림) | CEIL(-5.3)=-5 |
| FLOOR(x) | 작거나 같은 최대 정수(내림) | FLOOR(-1.8)=-2 |
| MOD(x,y) | 나머지 | MOD(9,2)=1 |
| ROUND(x,d) | d자리 반올림(d생략=0) | ROUND(272,-1)=270 |
| TRUNC(x,d) | d자리까지 버림 | TRUNC(1234.5678,-2)=1200 |
| POWER(x,n) | 거듭제곱 | POWER(3,2)=9 |
| SQRT(x) | 제곱근 | SQRT(100)=10 |

> ⚠️ **d < 0**: ROUND/TRUNC는 **정수 자리에서** 반올림/버림. (SQL Server: CEIL→CEILING)

### 3-3. 문자 함수
| 함수 | 기능 | 예 → 결과 |
|------|------|-----------|
| LOWER/UPPER | 소/대문자 | UPPER('aB')=AB |
| ASCII/CHR | 코드↔문자 | ASCII('A')=65, CHR(65)=A |
| CONCAT(s1,s2) | 결합 | CONCAT('A','B')=AB |
| SUBSTR(s,m,n) | m위치부터 n개 추출 | SUBSTR('ABCDE',2,3)=BCD / SUBSTR('ABCDE',-4,3)=BCD |
| INSTR(s,sub,m,n) | 찾는 위치 반환 | INSTR('banana','a')=2 |
| LENGTH(s) | 길이 | LENGTH('AB CD')=5 |
| LTRIM/RTRIM/TRIM | 좌/우/양쪽 제거(생략 시 공백) | LTRIM('#A#B#C#','#')=A#B#C# |
| LPAD/RPAD(s,n,p) | 총 n길이로 채움 | LPAD('abc',5,'0')=00abc |
| REPLACE(s,old,new) | 문자열 치환 | REPLACE('ABBAC','AB','ab')=abBAC |
| TRANSLATE(s,o,n) | 글자 1:1 치환 | TRANSLATE('ABBAC','AB','ab')=abbaC |

> ⚠️ SQL Server: SUBSTR→SUBSTRING, LENGTH→LEN, INSTR→CHARINDEX
> ⚠️ **REPLACE vs TRANSLATE**: REPLACE는 문자열 단위, TRANSLATE는 글자 1:1 매핑.

### 3-4. 날짜 함수
| 함수 | 기능 |
|------|------|
| SYSDATE | 현재 날짜+시간 |
| ADD_MONTHS(d,n) | n개월 후(음수면 전) |
| MONTHS_BETWEEN(d1,d2) | 개월 차이 |
| LAST_DAY(d) | 그 달 마지막 날 |
| NEXT_DAY(d,n) | d 이후 지정 요일 첫 날(1=일…7=토) |
| EXTRACT(year/month/day FROM d) | 년/월/일 추출 |

> SQL Server: SYSDATE→GETDATE(), ADD_MONTHS→DATEADD, MONTHS_BETWEEN→DATEDIFF

### 3-5. 변환 함수
- **TO_NUMBER(문자)**: 숫자로 / **TO_CHAR(대상,포맷)**: 문자로 / **TO_DATE(문자,포맷)**: 날짜로
- TO_CHAR(1250,'9,999')=1,250 / TO_CHAR(1250,'00000')=01250 (0은 자릿수 강제, 9는 표현)
- SQL Server: **CAST**(일반 변환), **CONVERT**(날짜 특정 형식)
- **암시적 형변환**보다 **명시적 형변환(CAST 등)** 권장

### 3-6. 집계함수 & NULL ⭐⭐⭐
| 함수 | 기능 |
|------|------|
| COUNT(*) | NULL 포함 전체 행 수 |
| COUNT(컬럼) | 해당 컬럼 NULL 제외 행 수 |
| SUM/AVG/MAX/MIN/STDDEV/VARIANCE | 합/평균/최대/최소/표준편차/분산 |

> ⚠️ **집계함수에서 NULL은 0이 아니라 무시(제외)된다!**
> 💡 MAX/MIN은 날짜형에도 사용 가능 (가장 최근/오래된 날짜)

### 3-7. NULL 관련 함수 ⭐⭐
| 함수 | 기능 |
|------|------|
| NVL(a,b) | a가 NULL이면 b, 아니면 a |
| NVL2(a,b,c) | a가 NULL이면 c, 아니면 b |
| NULLIF(a,b) | a=b면 NULL, 다르면 a |
| ISNULL(a,b) | (SQL Server) a NULL이면 b |
| COALESCE(a,b,c…) | 첫 번째 NOT NULL 값, 전부 NULL이면 NULL |
| DECODE(대상,값1,결과1,…,ELSE) | 대상=값1이면 결과1 (=CASE 단순 비교) |

> ⚠️ **NVL2 순서 주의**: NULL이면 **세 번째(c)**, 아니면 **두 번째(b)**.

### 3-8. CASE 표현 ⭐
- IF-THEN-ELSE-END 로직 = DECODE와 유사
- **SIMPLE CASE**: `CASE 컬럼 WHEN 값 THEN 결과 ... ELSE 기본 END` (등가 비교)
- **SEARCHED CASE**: `CASE WHEN 조건 THEN 결과 ... ELSE 기본 END` (조건식 평가, 범위 비교 가능)
- ELSE 생략 시 조건 불일치는 NULL 반환

---

## 4장. WHERE 절

### 4-1. 특징 ⭐
- 조건에 맞는 행만 필터링 (각 행을 개별 평가)
- **NULL은 `=`로 조회 불가 → IS NULL / IS NOT NULL 사용** ⭐
- ⚠️ **WHERE 절에 집계함수 사용 불가** (→ 그룹 조건은 HAVING으로)

### 4-2. 연산자 우선순위 ⭐
```
1.괄호  2.NOT  3.비교연산자·SQL연산자  4.AND  5.OR
```
> ⚠️ **AND가 OR보다 먼저** 실행 → 헷갈리면 괄호로 명확히!

### 4-3. 주요 연산자
| 연산자 | 의미 |
|--------|------|
| BETWEEN a AND b | a≤x≤b (a,b 포함, 반드시 a<b) |
| IN (list) | 목록 중 일치 (NULL 무시) |
| LIKE '패턴' | 부분 일치 (**대소문자 구분**) |
| IS NULL / IS NOT NULL | NULL 여부 |
| NOT IN / NOT BETWEEN | 반대 |

### 4-4. LIKE 와일드카드 ⭐
- `%` : 0개 이상의 문자 / `_` : 딱 1개의 문자
- `'S%'` = S로 시작 / `'%S'` = S로 끝 / `'%S%'` = S 포함
- `'_S%'` = 두 번째 글자가 S / `'__S__'` = 5글자며 가운데가 S

> ⚠️ **IS NULL 연산**: NULL과의 수치연산 → NULL / NULL과의 비교연산(`=`,`>`,`<`) → 거짓(FALSE). 그래서 `WHERE 컬럼 = NULL`은 한 건도 안 나옴.

---

## 5장. GROUP BY & HAVING ⭐⭐

### 5-1. GROUP BY
- 데이터를 소그룹으로 묶어 그룹별 통계
- ⚠️ **그룹 조건은 WHERE 불가 → HAVING 사용**
- ⚠️ **별칭(ALIAS) 사용 불가** (GROUP BY는 SELECT보다 먼저 실행)
- ⚠️ **그룹화 기준이 아닌 컬럼은 SELECT에 그냥 쓸 수 없다** (집계함수로 감싸야 함)

### 5-2. HAVING
- 그룹화된 결과에 조건 적용
- ⚠️ **HAVING은 SELECT보다 먼저 실행 → SELECT 별칭 사용 불가**
- WHERE는 그룹화 **전** 행 필터(성능↑), HAVING은 그룹화 **후** 필터
- HAVING은 보통 GROUP BY 뒤 위치

> 💡 **WHERE vs HAVING 구분 문제 단골**: 개별 행 조건 → WHERE / 그룹(집계) 조건 → HAVING

---

## 6장. ORDER BY ⭐

- 정렬 기준 컬럼 명시. **ASC(오름, 기본)** / **DESC(내림)**
- 여러 컬럼 시 **가장 왼쪽 컬럼부터 우선순위** (1차 정렬 동일값에 한해 2차 정렬)
- **SELECT 절 컬럼 별칭 사용 가능** (실행순서상 ORDER BY가 마지막)
- 문자·날짜도 정렬 가능 (날짜는 과거일수록 작은 값)
- ⚠️ **NULL 정렬: ORACLE은 NULL을 최댓값, SQL Server는 최솟값**으로 취급
- ORACLE: `NULLS FIRST / NULLS LAST`로 변경 가능
- ⚠️ GROUP BY 사용 시 ORDER BY에는 **SELECT 절에 있는 컬럼만** 가능

---

## 7장. 조인(JOIN) 기본

### 7-1. 조인 개념
- 여러 테이블을 공통 속성(JOIN KEY, 보통 FK)으로 결합
- ⚠️ **N개 테이블 조인 시 최소 N-1개 조인 조건 필요**
- FROM에 여러 테이블 나열돼도 SQL은 **2개 집합씩** 조인 (A JOIN B → 결과 JOIN C)
- 여러 테이블 조인 시 SELECT 컬럼이 **어느 테이블 것인지 명시** 권장

### 7-2. 조인의 종류
| 구분 | 종류 |
|------|------|
| 조건 형태 | **EQUI JOIN**(동등 `=`) / **NON EQUI JOIN**(비동등) |
| 결과 | **INNER JOIN**(일치만) / **OUTER JOIN**(LEFT·RIGHT·FULL, 불일치도) |
| 기타 | **NATURAL JOIN**(같은 이름 컬럼 자동) / **CROSS JOIN**(모든 조합=곱집합) / **SELF JOIN**(같은 테이블 2번↑) |

### 7-3. EQUI JOIN 문법 ⭐
- **ORACLE**: 조인 조건을 **WHERE 절**에 / 필터 조건도 WHERE에 AND로
- **ANSI/ISO SQL**: 조인 조건을 **ON 절**에, 필터 조건은 WHERE에
```sql
-- ORACLE
SELECT P.PLAYER_NAME, T.TEAM_NAME
FROM PLAYER P, TEAM T
WHERE P.TEAM_ID = T.TEAM_ID AND P.POSITION = 'GK';
-- ANSI/ISO
SELECT P.PLAYER_NAME, T.TEAM_NAME
FROM PLAYER P INNER JOIN TEAM T
ON P.TEAM_ID = T.TEAM_ID
WHERE P.POSITION = 'GK';
```
> ⚠️ 테이블에 별칭 적용 시 다른 절에서도 **본래 테이블명 아닌 별칭** 사용해야 함(안 그러면 오류).
> ⚠️ EQUI JOIN이 반드시 PK↔FK로만 성립하는 건 아니다.

---

## ✅ SQL 기본 확인문제

**Q1.** 롤백이 불가능한 명령어를 모두 고르시오: DELETE, TRUNCATE, UPDATE, DROP
<details><summary>정답</summary>TRUNCATE, DROP (둘 다 DDL, 오토커밋) — DELETE·UPDATE는 DML이라 롤백 가능</details>

**Q2.** SELECT문의 논리적 실행 순서를 쓰시오.
<details><summary>정답</summary>FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY</details>

**Q3.** 컬럼 별칭(ALIAS)을 사용할 수 있는 절은?
<details><summary>정답</summary>ORDER BY (실행순서상 SELECT 다음이라 가능). WHERE·GROUP BY·HAVING은 불가</details>

**Q4.** `ROUND(272, -1)`과 `TRUNC(1234.5678, -2)`의 결과는?
<details><summary>정답</summary>270, 1200</details>

**Q5.** 이름이 5글자이고 가운데(세 번째) 글자가 'S'인 조건의 LIKE 패턴은?
<details><summary>정답</summary>LIKE '__S__' (언더바 2개 + S + 언더바 2개)</details>

**Q6.** 연산자 `A AND B OR C`에서 먼저 평가되는 것은?
<details><summary>정답</summary>A AND B (AND가 OR보다 우선). 즉 (A AND B) OR C</details>

**Q7.** 그룹별 평균 급여가 300만 초과인 부서만 조회할 때 조건을 넣는 절은?
<details><summary>정답</summary>HAVING (집계함수 조건은 HAVING)</details>

**Q8.** ORACLE에서 포지션이 NULL인 선수를 조회하는 WHERE 조건은?
<details><summary>정답</summary>WHERE POSITION IS NULL (= NULL 사용 불가)</details>

**Q9.** NVL2('값', 'B', 'C')의 결과와, NVL2(NULL, 'B', 'C')의 결과는?
<details><summary>정답</summary>'B', 'C' (NULL 아니면 두 번째, NULL이면 세 번째)</details>

**Q10.** 5개 테이블을 조인할 때 필요한 최소 조인 조건 수는?
<details><summary>정답</summary>4개 (N-1)</details>
