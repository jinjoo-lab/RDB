#  (DML -1 : SELECT)

## 데이터 조작어 (DML)

- 데이터 쿼리 언어 (SELECT)
    - RDB에 저장된 데이터를 검색하는데 사용
- RDB의 테이블에 **데이터를 저장하거나 수정 , 삭제**하는데 사용

![Alt text](https://user-images.githubusercontent.com/84346055/254004351-a712adab-d944-4b74-b95b-0351899bed5d.png)

### SELECT

- RDB에서 데이터를 검색하는 문
    - 기본 질의 + 중첩 질의

```
SELECT [DISTINCT] Attribute
FROM Table
WHERE 조건
[ GROUP BY Attribute ]
[ HAVING 조건 ]
[ ORDER BY Attribute [ASC|DESC] ];
```

### 별칭(Alias)

- 칼럼, 테이블, 서브 쿼리 , Where절에서 별도의 이름 지정
- 예약어 : AS

**실습**

![Alt text](https://user-images.githubusercontent.com/84346055/254004382-bf516d1b-9413-42da-831d-e347874349e9.png)

1. 전체 부서의 모든 애트리뷰트들 검색

```
SELECT * FROM DEPARTMENT;
```

1. 모든 부서의 부서번호 , 부서 이름

```
SELECT DEPTNO, DEPTNAME FROM DEPARTMENT;
```

### 중복값 제거 - DISTINCT

```
SELECT DISTINCT TITLE FROM EMPLOYEE;
```

### 조건 적용 - WHERE

1. 2번 부서에 근무하는 사원들의 모든 정보

```
SELECT * FROM EMPLOYEE WHERE DNO = 2;
```

**문자열 비교 → % 사용**

1. 이씨 성을 가진 사원들의 이름 , 직급 , 소속 부서번호 검색

```
SELECT EMPNAME,TITLE,DNO FROM EMPLOYEE WHERE EMPNAME LIKE '이%';
```

1. 직급이 과장이면서 1번 부서에 근무하는 사원들의 이름 , 급여

```
SELECT EMPNAME,SALARY FROM EMPLOYEE WHERE TITLE = '과장' AND DNO = 1;
```

1. 5번에서 만약 1번 부서가 아닌 사람들이라면? → <>

```
SELECT EMPNAME,SALARY FROM EMPLOYEE WHERE TITLE = '과장' AND DNO <> 1;
```

[My 5.6 한글메뉴얼](http://www.innodbcluster.com/?depth=120501)

**범위 사용**

1. 급여가 3000000 이상이고 4500000 이하인 사원들의 이름 , 직급 , 급여 검색

```
SELECT EMPNAME,TITLE,SALARY FROM EMPLOYEE WHERE SALARY >= 3000000 
AND SALARY <= 4500000;

SELECT EMPNAME,TITLE,SALARY FROM EMPLOYEE WHERE SALARY BETWEEN
3000000 AND 4500000;
```

1. 1번 부서나 3번 부서에 소속된 사원들에 관한 모든 정보

```
SELECT * FROM EMPLOYEE WHERE DNO IN (1, 3);
```

- **SELECT 절에 산술 연산자 사용 시 !**

  값이 변경되는 것이 아닌 결과만 변경된 값이 보여지는 것


```
SELECT EMPNAME, SALARY , SALARY * 1.1 AS NEWSALARY
FROM EMPLOYEE WHERE TITLE = '과장';
```

![Alt text](https://user-images.githubusercontent.com/84346055/254004391-286b0bc0-e8e7-4188-9230-f4b6c1d413c3.png)

- NULL값에 대해서는 연산 결과는 NULL
- DNO = NULL ( 결과가 모두 거짓으로 적용 ) → DNO IS NULL , DNO IS NOT NULL

```
SELECT EMPNAME FROM EMPLOYEE WHERE DNO IS NOT NULL;
```

### 정렬 - ORDER BY

- 질의 결과를 정렬하여 사용자에게 제공
- SELECT 문에서 가장 마지막에 사용 !
    - DEFAULT : ASC(오름차순)
    - DESC(내림차순)
- 정렬 조건을 여러 개 적용 가능 , **SELCET절에 명시한 Attribute만 사용 가능**
    - ORDER BY DNO, SALARY DESC

---

1. 2번 부서에 근무하는 사원들의 급여 , 직급, 이름을 검색하여 급여의 오름차순 정렬

```
SELECT SALARY,TITLE,EMPNAME FROM EMPLOYEE WHERE DNO = 2 
ORDER BY SALARY ASC;
```

**집단 함수**

- 검색 결과에 대해 적용 가능 → 단일 값을 반환
- **SELECT** 절과 **HAVING** 절에서만 사용 가능
    - COUNT(*)를 제외한 집단 함수는 널값을 제거한 후 남아있는 값에 대해 연산 수행
    - DISTINCT와 함께 사용 시 → 집단 함수 적용 전에 먼저 중복 제거

![Alt text](https://user-images.githubusercontent.com/84346055/254004396-f5659d66-90ca-4fc4-b5ad-ddc11877b85a.png)

1. 모든 사원들의 평균 급여와 최대 급여

```
SELECT AVG(SALARY) as AV , MAX(SALARY) as MA FROM EMPLOYEE;
```

### 그룹화 - GROUP BY

- GROUP BY 절에 사용된 **애트리뷰트에 동일한 값을 갖는 투플들이 하나의 그룹**으로 묶임
    - 각 **그룹마다 한개의 투플 !**
- GROUP BY 사용 시 각 그룹마다 하나의 값을 갖는 애트리뷰트 , 집단 함수 , 사용된 애트리뷰트들만 결과 릴레이션에 나타남
1. 모든 사원들을 부서 번호별로 그룹화 → 부서 번호 , 평균 급여 , 최대 급여

```
SELECT DNO , AVG(SALARY) as AV , MAX(SALARY) as MA FROM EMPLOYEE 
GROUP BY DNO;
```

![Alt text](https://user-images.githubusercontent.com/84346055/254004401-fb03e219-cdf2-4ffb-8b6a-2ddb3f605ba6.png)

### 그룹 선정 조건 - HAVING

- 질의 결과에 나타날 그룹 조건을 명시할 수 있다.
    - HAVING절에는 GROUP BY 절에 나타나는 애트리뷰트 , 집단 함수 , 그룹마다 유일 값 애트리뷰트
1. 모든 사원 → 부서 번호 그룹화 , 평균 급여 2500000 이상 부서 → 부서번호 , 평균 급여 , 최대 급여

```
SELECT DNO, AVG(SALARY) as AV , MAX(SALARY) as MA FROM EMPLOYEE 
GROUP BY DNO 
HAVING AVG(SALARY) >= 2500000;
```

### 조인

- 두 개 이상의 릴레이션으로부터 연관된 투플들의 집합
- 기본 형태
    - SELECT문과 같이 FROM절에서 두 개 이상의 릴레이션 열거
    - 릴레이션에 속하는 애트리뷰트 비교 조건 → WHERE
- 조인 조건 생략 시 → 카티션 곱
- 두 릴레이션의 조인 애트리뷰트 이름 동일 → **구분 필요 (릴레이션 이름 or 별칭)**
1. 모든 사원의 이름과 이 사원이 속한 부서 이름 검색

```
SELECT EMPNAME , DEOTNAME
FROM EMPLOYEE , DEPARTMENT
WHERE EMPLOYEE.DNO = DEPARTMENT.DEPTNO
```

### 자체 조인

- 한 릴레이션 내의 애트리뷰트들 끼리 조인 !
    - 한 릴레이션에 대하여 별칭을 **두 개 사용**해야 함
1. 모든 사원에 대하여 사원 이름 , 직속 상사 이름

```
SELECT A.EMPNAME , B.EMPNAME
FROM EMPLOYEE AS A,B
WHERE A.EMPNO = B.MANAGER;
```

1. 모든 부서 → 소속 부서 이름 , 사원 이름 , 직급 , 급여
    1. 부서 이름 → 오름 차순 , 같은 경우 SALARY 내림 차순

```
SELECT DEPTNAME,EMPNAME,TITLE,SALARY
FROM EMPLOYEE AS E,DEPARTMENT AS D
WHERE E.DNO = D.DEPTNO
ORDER BY DEPTNAME ASC , SALARY DESC
```

```
SELECT EMPNAME , DEPTNAME
FROM EMPLOYEE AS E, DEPARTMENT AS D
WHERE  E.DNO = D.DEPTNO AND E.TILTE = '과장';
```

### 중첩 질의

- 외부 질의의 WHERE 절에 다시 SELECT FROM WHERE 절 포함
- INSERT , DELETE , UPDATE 문에도 사용 가능

![Alt text](https://user-images.githubusercontent.com/84346055/254004404-f53f0506-f96a-4964-a08a-a65b864a4d23.png)

1. 박영권과 같은 직급을 갖는 모든 사원들의 이름 , 직급

```
SELECT EMPNAME , TITLE FROM EMPLOYEE AS E
WHERE TITLE = (
	SELECT TITLE FROM EMPLOYEE WHERE EMPNAME = '박영권'
);
```

- 데이터 정제를 위해 외부 WHERE 절에 IN , ANY , ALL ,EXISTS 사용
- 중첩 질의는 단일 질의로 쪼갤 수 있다. → 대부분의 경우 JOIN형태로 나타남

### 상관 중첩 질의

- 중첩 질의의 WHERE절에 있는 프레디키트에서 외부 질의에 선언 된 릴레이션의 일부 애트리뷰트를 참조하는 질의
1. 자신이 속한 부서의 사원들의 평균 급여보다 많은 급여를 받는 사원들에 대해서 이름 , 부서번호, 급여

```
SELECT EMPNAME , DNO , SALARY
FROM EMPLOYEE E
WHERE SALARY >= (SELECT AVG(SALARY) FROM EMPLOYEE WHERE E.DNO = DNO);
```

![Alt text](https://user-images.githubusercontent.com/84346055/254004409-0bca1b63-c06e-4030-9893-84b8cb8652f7.png)
