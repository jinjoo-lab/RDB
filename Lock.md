# 락

## 동시성 제어

> **다중 사용자 환경**에서 둘 이상의 **트랜잭션이 동시에 수행**될 때 , 일관적 처리를 위해 **데이터의 접근을 제어**하는 것
>

### 목적

1. **트랜잭션의 직렬성** 보장
    - 직렬성 : 여러 트랜잭션이 동시에 병행 수행된 결과 = 하나 씩 수행되는 트랜잭션들의 결과
2. **데이터의 무결성 , 일관성** 보장

### 동시성 제어 기법

1. **(비관적)락**
2. **낙관적 검증 (Validation)**
3. Timestamp Ordering
4. MVCC

## Lock

> 트랜잭션의 동시성 제어를 위해 데이터 베이스의 데이터나 자체의 접근을 제한하는 기법
>
- **트랜잭션의 순차성**을 보장하기 위한 기법
- **Lock을 건다는 것은**

  Lock을 거는 트랜잭션에 대하여 해당 데이터 항목에 대한 **연산 제한의 권한**을 가진다는 것이다.

    - 독점 락을 건 경우 해당 트랜잭션만 **쓰기 , 읽기 연산**이 가능
    - 공유 락의 경우 양립 가능하지만 **쓰기 연산에 대한 제한을 거는 주체**는 해당 트랜잭션이다.

### Lock 연산 종류

- 독점(배타) 락 (Exclusive Lock)
    - 트랜잭션에서 **갱신(수정)**을 목적으로 데이터 항목에 접근 시 요청
    - 독점 락에 대해서는 **읽기 , 쓰기** 연산 모두 불가능
    - 하나의 데이터 항목에 대해서는 하나의 독점 락만 존재할 수 있다.
        - **독점 락은 양립 불가능**
- 공유 락(Shared Lock)
    - 트랜잭션에서 **조회(읽기)**를 목적으로 데이터 항목에 접근 시 요청
    - 공유 락에 대해서는 읽기 연산만 가능, **쓰기 연산 불가능**
    - 공유 락끼리는 양립 가능
        - 하나의 데이터 항목에 대하여 2개 이상의 공유 락이 존재할 수 있다.
- **공유 락이 양립 가능한 이유**

  읽기 연산만 허용하기 때문에 데이터의 변경을 걱정하지 않아도 된다.


**락 연산의 양립성 행렬**

![Untitled](https://user-images.githubusercontent.com/84346055/282013415-f493d159-4189-480a-9c70-eb093f382aa6.png)

### Lock 단위

- 락의 단위는 사용하는 DBMS의 종류에 따라 다르다.

![Untitled](https://user-images.githubusercontent.com/84346055/282013445-109f0739-7399-43d4-9279-bde8bae1c79d.png)

- ←
    - 구현이 용이 , Lock 연산에 의한 오버헤드 감소
    - 동시성 제어가 약하다.
- →
    - 구현이 복잡하다. Lock 연산에 대한 오버헤드 증가
    - 동시성 제어가 강하다.

## 비관적 락

> 데이터 베이스의 락을 사용하여 애초에 데이터에 대한 **접근을 막아 동시성 문제를 막는 것**
>
- 충돌이 발생할 것이라는 **비관적 가정**
- 트랜잭션에 대하여 Lock 연산을 사용
    - 데이터베이스의 락 연산 사용 :  Read Lock + Write Lock
    - 데이터베이스 단에서 동시성 처리

### 장점

- 충돌이 많은 경우 롤백 횟수를 줄여 성능이 우수하다.
- 데이터의 일관성과 무결성을 보장하는 수준이 높다.

### 단점

- 읽기 작업이 많은 경우 동시성이 떨어진다.
- 동시 접근을 통한 자원이 필요할 때 **데드락 발생**할 수 있다.

## 낙관적 락

> 데이터 베이스의 락을 사용하지 않고 **동시성 문제가 발생하면 그때 처리(롤백) 수행**
>
- 충돌이 발생하지 않을 것이라는 **낙관적 가정**
- Lock 연산을 사용하는 것이 아닌 version이라는 것을 사용한다.
    - 애플리케이션 단에서 동시성 처리
    - Version(구분 칼럼)을 이용
        - 쓰기 연산이 수행된 경우 - version의 값이 증가한다. (변경된다)
        - Version의 상태를 보고 충돌을 확인하여 충돌이 발생한 경우 : 롤백 진행

![Untitled](https://user-images.githubusercontent.com/84346055/282013450-0eaaa2f6-2646-4dc7-8a23-970a9f24fd51.png)

**동작 과정**

1. USER_1은 id = 2인 row를 읽음 (karol , 1)
2. USER_2도 id = 2인 row를 읽음 (karol , 1)
3. USER_2가 읽은 row 변경: 쓰기 연산 수행 (Karol , 2) **성공**
4. USER_1이 읽은 row 변경 : 읽은 version값이 변경되었기 때문에 쓰기 연산 수행 불가능 : **실패**

> **같은 row에 대해서 각기 다른 2개의 수정 요청이 있었지만 1개가 업데이트 됨에 따라 version이 변경되었기 때문에 뒤의 수정 요청은 반영되지 않게 되었습니다.**
>

### 장점

- 충돌이 적은 경우 성능이 좋다.
- 트랜잭션이 필요하지 않다.

### 단점

- 충돌이 많은 경우 롤백처리에 대한 비용이 많이 발생한다.
- Application Level에서 동시성 처리를 하기 때문에 수동으로 롤백 처리를 해줘야 한다.
    - 별도의 롤백 처리 로직을 구현해야 한다.
