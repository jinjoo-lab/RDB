#  (DML-2)

### INSERT

- 기존 **릴레이션에 투플을 삽입**
    - 참조되는 릴레이션의 투플 삽입 : 참조 무결성 제약조건을 위배하지는 않는다.
    - 참조하는 릴레이션의 투플 삽입 : 외래키 값에 따라 참조 무결성 제약조건 위배 가능

**기본 구문**

``` 
INSERT INTO Student VALUES ('진주원','Jinjooone','1999-11-01');
```

**SELECT 문의 결과를 삽입 시**

1. EMPLOYEE 테이블에서 급여가 3000000 이상인 사원들의 이름 , 직급 , 급여를 HIGH_SALARY 릴레이션에 삽입

``` 
INSERT INTO HIGH_SALARY(EMPNAME , TITLE , SALARY)
SELECT EMPNAME , TITLE , SALARY
FROM EMPLOYEE
WHERE SALARY >= 3000000;
```

### DELETE

- 한 릴레이션으로부터 한 개 이상의 **투플들을 삭제**
    - 참조하는 릴레이션 투플 삭제 : 참조 무결성 제약조건을 위배하지는 않는다.
    - 참조되는 릴레이션 투플 삭제 : 참조 무결성 제약조건 위배 가능
        - 별도의 처리 필요 (ON DELETE CASCADE 등등)

**기본 구문**

``` 
DELETE FROM DEPARTMENT WHERE DEPTNO = 4;
```

### UPDATE

- 한 릴레이션에 들어 있는 **레코드들의 애트리뷰트 값들을 수정**
    - 기본 키나 외래 키의 값 수정 시 참조 무결성 제약조건 위배 가능 ! (주의 필요)

**기본 구문**

``` 
UPDATE EMPLOYEE
SET DNO = 3, SALARY = SALARY * 1.1
WHERE EMPNO = 2106;
```

### 트리거

- 명시된 **이벤트가 발생**할 때마다 **DBMS가 자동적으로 수행**하는 사용자 정의 문 (프로시저)
    - **데이터베이스 무결성 유지** 위한 강력한 도구
    - 테이블 정의시 **비즈니스 규칙을 정의**

**ECA 규칙**

- 트리거의 다른 명칭
- Event → Condition → Action
    - Event : 트리거를 활성화시키는 사건
    - Condition : 트리거 활성화 시 수행 조건
    - Action : 조건 만족시 수행되는 프로시저(구문)

![Alt text](https://user-images.githubusercontent.com/84346055/254495678-832fd214-c676-4cda-aac0-cdb8c5e324c2.png)

- **트리거 형식**

![Alt text](https://user-images.githubusercontent.com/84346055/254495698-ba6f1d08-faf8-441f-ab86-8121209f876c.png)

- **트리거 분류**
    - 이벤트 발생 전의 트리거(BEFORE) or 이벤트 발생 후의 트리거(AFTER)
    - 테이블 수준의 트리거 or 행 수준의 트리거
