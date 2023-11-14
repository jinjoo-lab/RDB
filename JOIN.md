# JOIN

> 우리는 정규화나 각 로직에 적합하게 테이블을 분리한다. 하지만 원하는 데이터를 추출할 때 2개 이상의 테이블을 사용해야 하는 경우에는 적절한 작업이 필요하다.
>

## Join

- 두 개 이상의 릴레이션으로부터 연관된 투플들의 집합
    - 복수의 테이블을 결합하여 데이터를 조회하는 것
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

### INNER JOIN (동등 조인)

- 테이블 간의 **교집합**을 의미

![Untitled](https://user-images.githubusercontent.com/84346055/282738271-af50ba6b-af79-468b-aac4-c901f6454f07.png)

**기본 구문**

```
SELECT ATTRIBUTE FROM TABLE1 
INNER JOIN TABLE2 ON TABLE1.fid = TABLE2.id
```

- ON 절의 조건을 만족하는 데이터를 추출
    - 왼쪽 테이블과 오른쪽 테이블 중 **기준 테이블에 대한 자유도가 높다.** (교집합을 추출하는 개념)

## OUTER JOIN

- OUTER JOIN 의 연산 결과에는 ON 조건을 만족하지 못하는 데이터가 포함될 수 있다.
- 해당 경우에 속성에는 null값이 채워지게 된다.

**RIGHT OUTER JOIN 연산 후 null 값이 채워지는 경우**

![Untitled](https://user-images.githubusercontent.com/84346055/282738291-a56a1004-a633-4cb9-9370-4f90e13b14d3.png)

**기본 구문**

```
SELECT ATTRIBUTE FROM TABLE1
LEFT | RIGHT | FULL OUTER JOIN TABLE2 ON 조인조건;
```

### LEFT OUTER JOIN

- 왼쪽 테이블을 기준 테이블로 **왼쪽 테이블 + 조인 테이블** 추출

![Untitled](https://user-images.githubusercontent.com/84346055/282738302-9d8ef09d-225e-49bd-b7b5-d23e129a7dbf.png)

**구문**

```
SELECT ATTRIBUTE FROM TABLE1
LEFT OUTER JOIN TABLE2 ON 조인 조건;
```

### RIGHT OUTER JOIN

- 오른쪽 테이블을 기준 테이블로 **오른쪽 테이블 + 조인 테이블** 추출

![Untitled](https://user-images.githubusercontent.com/84346055/282738305-35e010d4-79ea-4b32-b35d-7c5507383455.png)

**구문**

```
SELECT ATTRIBUTE FROM TABLE1
RIGHT OUTER JOIN TABLE2 ON 조인 조건;
```

### FULL OUTER JOIN

- 합집합의 의미 : A테이블과 B테이블의 모든 데이터가 검색된다.

![Untitled](https://user-images.githubusercontent.com/84346055/282738306-5f948ef0-4648-4377-90d6-8a35b885f0bd.png)

**구문**

```
SELECT ATTRIBUTE FROM TABLE1
FULL OUTER JOIN TABLE2 ON 조인조건;
```
