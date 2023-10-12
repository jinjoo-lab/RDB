# MySQL 아키텍처 (InnoDB 스토리지)

## InnoDB 스토리지

### 아키텍처

![Untitled](https://user-images.githubusercontent.com/84346055/274527807-2281cad1-1073-4571-a4b1-a88380397bf2.png)

![Untitled](https://user-images.githubusercontent.com/84346055/274527833-4463cf9b-1701-4639-a1a2-b8a56fadc378.png)

- 레코드 기반의 잠금(트랜잭션)을 지원하는 스토리지 엔진
    - 동시성 처리가 가능하고 안정적이다.

### 클러스터링 인덱스

- InnoDB의 모든 테이블 → **PK를 기준으로 클러스터링**되어 저장 (PK - 클러스터링 인덱스)

> PK값을 기준으로 정렬되어 디스크에 저장 → PK가 물지적인 저장 위치를 결정
>
- 클러스터링 인덱스를 사용하기 때문에 **세컨더리 인덱스는 리프 노드(B-트리)**에는 RowID가 아닌 **PK**가 저장
    - PK값이 새로 추가될 때 물리적 저장 주소가 변경될 수 있기 때문에 RowID를 저장하는 것은 올바르지 않다.
- **장점**
    - PK를 이용한 인덱스 레인지 스캔의 성능이 뛰어나다.
        - 옵티아미저가 실행 계획 설정 시 PK의 비중을 다른 세컨더리 인덱스보다 높게 설정

---

### Undo Log

- 데이터가 변경되었을 경우 **변경 전의 데이터를 보관 (디스크 영역에 로그 파일 존재)**
    - 트랜잭션 롤백
    - MVCC

### MVCC (Multi Version Concurrency Control)

> 해당 기능은 기본적으로 DBMS가 **레코드 단위의 트랜잭션**을 지원해야 한다.
>
- Multi Version : **하나의 레코드에 대해 여러 개의 버전**을 동시에 관리
    - InnoDB 버퍼 풀과 Undo-Log에 데이터를 기록하여 격리 수준에 따라 적절한 값을 반환
    - 이를 통해 **잠금을 사용하지 않고 일관된 읽기 작업**을 수행 가능하게 한다.
- **Undo Log** 이용

**Ex)**

```
INSERT INTO member(m_id, m_name, m_area) VALUES (12,'홍길동','서울');
commit;
```

- 쿼리가 실행되고 난 경우

![Untitled](https://user-images.githubusercontent.com/84346055/274527843-17aa76dd-69c5-4705-8cb3-d9825fc081ef.png)

- **추가 UPDATE 쿼리**

```
UPDATE member SET m_area='경기' WHERE m_id = 12;
```

![Untitled](https://user-images.githubusercontent.com/84346055/274527847-d0141ae4-53b6-4909-bb10-90f6cfb485bd.png)

UPDATE 문장 실행

- 커밋 실행 여부와 관계 없이 **InnoDB 버퍼 풀은 새로운 값으로 UPDATE**
- 변경 전의 값을 **Undo - Log**에 복사
- **다른 사용자가 해당 레코드에 접근한다면 ?**
    - READ_UNCOMMITTED : 커밋 여부에 상관 없이 변경된 값을 반환
    - 상위 단계의 격리 수준 : 커밋되지 않은 경우 Undo - Log 데이터 반환
- ROLLBACK
    - Undo-Log 의 데이터를 기반으로 롤백을 진행한다.

---

### 잠금 없는 일관된 읽기(Non-Locking Consistent Read)

- 위에 언급한 MVCC를 통해 잠금을 걸지 않고 읽기 작업 수행

> **다른 트랜잭션이 가지고 있는 잠금을 기다리지 않고** 읽기 작업 가능
>
- READ_UNCOMMITTED , READ_COMMITTED, REPEATABLE_READ
    - SELECT문은 **잠금을 대기하지 않고 바로 실행** ( **버퍼 풀** or **Undo - Log** )

---

### 자동 데드락 감지

- InnoDB 스토리지 엔진
    - 교착 상태 확인을 위해 잠금 대기 목록을 **Wait-for List**  그래프 형태로 관리
    - 자체적으로 **데드락 감지 스레드**를 가지고 있다.
        - 주기적으로 잠금 대기 그래프를 검사 → 교착 상태에 빠진 트랜잭션을 찾아 강제 종료
        - 종료시킬 **트랜잭션 판단** → Undo - Log의 양 (**적을수록 롤백 대상**)
- **동시 처리 스레드의 개수가 많거나 트랜잭션의 잠금 개수가 많다면**
    - 데드락 감지 스레드의 속도가 느려지고 많은 CPU 자원 소모 → innodb_deadlock_detect 시스템 변수 사용 (ON / OFF)
    - 데드락 감지 스레드를 비활성화하는 경우 → timeout을 줄이도록 하자 !

---

## InnoDB 버퍼 풀

> 메모리에 디스크의 데이터 파일이나 인덱스 정보를 **캐시해 두는 공간**
>
- 지연 쓰기를 위한 버퍼 역할도 수행
    - 한번에 쓰기 작업을 수행할 경우 Random I/O(데이터베이스 성능에 너무나도 큰 기여)를 줄일 수 있다.

### 성능 향상 1 : 캐시

- 버퍼 풀 메모리 공간 → **페이지 크기** 단위로 관리

```
InnoDB 스토리지 엔진에서 데이터 찾는 과정

1. 데이터가 포함된 페이지가 버퍼 풀에 있는지 Check
	* InnoDB 어댑티브 해시 인덱스 -> 페이지 검색
	* 인덱스(B-Tree)를 이용 버퍼 풀에서 페이지 검색
	* **Find Data -> LRU리스트에서 해당 페이지 포인터 -> MRU 방향으로 승급**

2. 디스트에서 데이퍼 페이지 찾아 버퍼 풀에 적재 -> 페이지 포인터 LRU 헤더 추가

3. 데이터 페이지가 읽히면 MRU 쪽으로 이동

4. 버퍼 풀 데이터 페이지 -> 최근 접근에 따라 Age 부여
	* 오랫동안 반복해서 읽히지 않는다면 : 버퍼 풀에서 제거
	* 반복해서 읽힌다면 Age 초기화 -> MRU 쪽으로 이동
```

1. **LRU** 리스트
    - 목적 : 디스크로부터 한번 읽은 페이지를 최대한 오랫동한 InnoDB 버퍼 풀 메모리에 유지 → Disk I/O를 줄이기 위해
    - Least Recently Used + Most Recently Used 의 형태 → Old + New 서브 리스트 영역
    - LRU의 끝으로 밀려난다면 : InnoDB 버퍼 풀에서 삭제
2. **Free** 리스트
    - InnoDB 버퍼 풀에서 실제 사용자 데이터로 채워지지 않은 비어 있는 페이지 목록
3. **Flush** 리스트
    - 디스크로 동기화되지 않은 데이터를 가진 더티 페이지의 변경 시점 기준 페이지 목록 관리
        - 데이터가 변경 → Flush 리스트에서 관리 + 특정 시점에 디스크로 기록
        - 변경 내용 : 리두 로그 + 버퍼 풀 데이터 페이지에 기록

---

### 성능 향상 2 : 쓰기 버퍼링 (지연 쓰기)

- **InnoDB의 페이지**
    1. 클린 페이지(Clean Page) : 디스크에서 읽은 상태로 **전혀 변경되지 않은 페이지**
    2. 더티 페이지(Dirty Page) : INSERT , UPDATE , DELETE 명령으로 **변경된 데이터를 가진 페이지**

> 더티 페이지의 데이터를 디스크에 반영해야 한다.
>
- **리두 로그 엔트리**는 버퍼 풀의 **더티 페이지**와 관계 생성
    - 재사용 가능 공간
    - 재사용 불가능 공간 → 활성 리두 로그

![Untitled](https://user-images.githubusercontent.com/84346055/274527851-67af6555-441c-4ec2-bce3-0b1a9f00be98.png)

- **쓰기 버퍼링**
    - **더티 페이지의 내용들을 디스크에 플러시(반영)**하는 것을 의미한다.

> InnoDB 엔진 → 주기적으로 **체크포인트 이벤트** 발생
>
- 리두 로그와 버퍼 풀(더티 페이지)를 디스크로 동기화
    - 가장 최근 체크포인트 지점의 LSN이 활성 리두 로그 공간의 시작점
- 체크포인트 LSN보다 작은 리두 로그 엔트리의 더티 페이지 → 디스크로 동기화

## 버퍼 풀 플러시

### 플러시 리스트 플러시 (Flush_list Flush)

- **사용 이유**
    - 리두 로그 공간의 재활용을 위해 오래된 리두 로그 엔트리는 삭제 필요
    - 리두 로그 엔트리를 삭제하기 전에 엔트리와 관계가 생성된 더티 페이지를 디스크에 동기화 필요

> InnoDB 스토리지 엔진 플러시 리스트 플러시 함수 호출 → 더티 페이지 디스크 동기화 수행
>
- **클리너 스레드** : 더티 페이지의 내용을 디스크에 반영하는 스레드 ( 버퍼 풀 인스턴스의 개수와 최소한 같도록 설정 필요)

### LRU 리스트 플러시

- **사용 이유**
    - LRU 리스트는 페이지에 대한 최신 접근 빈도를 기준으로 초기화 된다.
    - 추가적인 접근이 발생하지 않은 페이지들은 리스트에서 제거 필요
- 스캔
    - 더티 페이지 → 디스크 동기화
    - 클린 페이지 → 프리 리스트로 이동

### Double Write Buffer

- 하드웨어 오작동 OR 시스템 비정상 시

> 더티 페이지의 내용을 디스크에 반영할 때 **일부만 반영**된다면 ? → 데이터 베이스의 일관성이 깨진다.
>
- Double Write 기법

![Untitled](https://user-images.githubusercontent.com/84346055/274527856-8c4862cf-4949-4327-84f5-7cdb1716ae40.png)

1. 더티 페이지들을 데이터 파일에 기록하기 전 **변경된 페이지들을 한번에 DoubleWrite 버퍼에 기록**
2. 각 더티 페이지들을 데이터 파일에 하나 씩 Random I/O 를 통해 기록
3. **하드웨어 오작동 OR 시스템 비정상 종료**
    - InnoDB 스토리지 재시작 시 DoubleWrite 버퍼의 내용과 데이터 파일 비교 → 동일하지 않다면 DoubleWrite 버퍼의 내용을 복사