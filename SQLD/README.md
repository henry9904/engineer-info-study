# 📚 SQLD 개념정리 (문제풀이용 재구성)

2024 개정판 SQLD 개념정리 PDF(103쪽)를 **바로 문제를 풀 수 있게** 세부 재구성한 노트입니다.
각 장은 **개념 → ⚠️ 시험 함정 → ✅ 확인문제** 순서로 구성했고, 별표(⭐)는 출제 빈도가 높은 핵심입니다.

## 목차

### 1과목. 데이터 모델링의 이해
- [1과목_데이터모델링의_이해.md](1과목_데이터모델링의_이해.md)
  - 모델링·3단계·독립성·3단계 스키마 / 엔터티 / 속성 / 관계 / 식별자 / 정규화·반정규화 / 조인·트랜잭션·NULL / 본질vs인조 식별자

### 2과목. SQL 기본과 활용
- [2과목-1_SQL_기본.md](2과목-1_SQL_기본.md)
  - 관계형 DB·무결성·제약 / SELECT 6절·실행순서 / 함수(숫자·문자·날짜·변환·NULL·CASE) / WHERE·연산자 / GROUP BY·HAVING / ORDER BY / 조인 기본
- [2과목-2_SQL_활용.md](2과목-2_SQL_활용.md)
  - 서브쿼리(스칼라·인라인뷰·중첩) / 집합연산자 / 그룹함수(ROLLUP·CUBE·GROUPING SETS·GROUPING) / 윈도우함수(순위·LAG/LEAD·비율) / TOP N·ROWNUM
- [2과목-3_계층질의_관리구문.md](2과목-3_계층질의_관리구문.md)
  - 계층질의·셀프조인 / PIVOT·UNPIVOT / 정규표현식 / DML·MERGE / TCL / DDL·제약조건 / VIEW·SEQUENCE·SYNONYM / DCL·ROLE·권한옵션

## 시험 초빈출 포인트 Top 10
1. **SELECT 실행순서** FROM→WHERE→GROUP BY→HAVING→SELECT→ORDER BY (별칭은 ORDER BY만)
2. **TRUNCATE=DDL(롤백X)**, DELETE=DML(롤백O)
3. **집계함수는 NULL 무시**, COUNT(*)만 NULL 포함
4. **슈퍼키**=유일성O·최소성X
5. **식별자 관계(실선,자식PK) vs 비식별자 관계(점선,자식FK)**
6. **정규화**: 2NF=부분함수종속 제거, 3NF=이행함수종속 제거, BCNF=결정자가 후보키
7. **ROLLUP(계층·순서O) / CUBE·GROUPING SETS(평등·순서X)**
8. **RANK(건너뜀) vs DENSE_RANK(안건너뜀)**
9. **ROWNUM `>` 연산 불가** → 인라인뷰로 정렬 후 ROWNUM
10. **GRANT OPTION(연쇄회수) vs ADMIN OPTION(제3자 남음)**, ROLE은 재접속 필요
