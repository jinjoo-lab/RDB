# Undo Log + Redo Log

### InnoDB Storage Engine Architecture

![Untitled](https://user-images.githubusercontent.com/84346055/275507875-09659461-b9ff-4995-919a-5b7ced40c6cf.png)
## Undo-Log

> InnoDB 스토리지 엔진은 **트랜잭션 롤백**과 **격리 수준 보장**을 위해 DML(삽입,변경,삭제)로 변경되기 전의 데이터를 별도로 백업하는데 해당 백업 데이터를 Undo Log라 한다.
>

사용 이유

1. 트랜잭션 롤백 : 트랜잭션 도중 변경된 데이터를 변경 전 데이터로 복구해야 하는데 언두 로그에서 백업해 둔 이전 데이터 사용
2. 격리 수준 보장 : 트랜잭션 격리 수준에 맞게 변경 전의 데이터를 반환해야 하는 경우 Undo Log 데이터 참조

```
UPDATE member SET name = '홍길동' WHERE member_id = 1;
```

- 쿼리가 실행되면 트랜잭션이 커밋되지 않아도 **버퍼의 내용은 변경**된다.
- Undo 영역에는 **변경되기 전의 데이터**가 저장(백업)
- **Undo 영역(records) , Undo Log 의 차이**
    - Undo Log는 디스크에 별도로 저장되는 것을 알 수 있다. 그렇다면 Undo Log가 저장되기 까지의 과정은?

  ![Untitled](https://user-images.githubusercontent.com/84346055/275507925-2f755c12-d690-42d5-bd8b-ef5d9a74a3f6.png)

    - 위 그림에서 Buffer Pool에서 어떤 데이터가 다뤄지는지를 알 필요가 있다.
        - Data page , Index page , **undo records** , adaptive hash index
    1. Undo Record 영역(**memory buffer pool**)에 변경되기 전의 데이터가 저장
    2. CheckPoints 시 Undo Record의 데이터가 **디스크**에 기록

### 저장 위치

- Undo Log가 저장되는 공간을 Undo Tablespace라 한다. (디스크의 영역)
    - 기존에는 **시스템 테이블 스페이스**에 저장(ibdata.ibd)
        - 시스템 테이블스페이스는 서버가 초기화될 때 생성 → 확장에 어려움이 있다.
    - 현재 : 시스템 테이블 스페이스 외부의 **별도의 로그 파일**

![Untitled](https://user-images.githubusercontent.com/84346055/275507936-d7e9c62b-72fe-4da8-a2e3-310ac70d2cba.png)

### Undo Tablespace truncate

- 언두 테이블 스페이스가 별도의 파일로 관리되면서 확장과 축소가 쉬워졌다.
    - 필요한 공간만 남기고 **공간을 운영체제에게 반납**하는 것
1. 자동 모드
    - InnoDB 스토리지 엔지의 **퍼지 스레드**가 주기적으로 불필요한 언두 로그 삭제
2. 수동 모드
    - 언두 테이블 스페이스 자체를 비활성화해서 더이상 사용되지 않도록 한다. (비활성화된 공간을 운영체제에 반납)

## Redo-Log

- **ACID의 D 를 위한 복원**

> Durability(영속성 , 지속성) : 하나의 **트랜잭션이 성공적으로 동작**되었다면 **시스템 오류가 발생**되더라도 영구적으로 **반영**되어야 한다. (로그로 남아야 한다)
>
- 데이터베이스에 Write 작업의 Random I/O는 비용이 큰 작업
    - 그렇기 때문에 InnoDB에서는 Buffer Pool을 사용한다. 하지만 Buffer Pool은 **메모리 (휘발성)**
    - InnoDB memory 영역에는 **Log Buffer**란 공간이 존재 : 해당 공간에 변경된 데이터의 로그를 기록
        - 해당 로그가 **checkpoint** 시 **redo log** 에 기록
- 서버가 비정상적으로 종료되었을 때 데이터 파일(디스크)에 반영되지 않은 데이터를 잃지 않게 해주는 장치

### 복원

1. 커밋 → 디스크에 기록되지 않은 데이터
- Redo Log를 통해 마지막 체크 포인트시 기록되 데이터를 디스크에 복사
1. 롤백 → 디스크 파일에 이미 데이터 기록
- Redo Log와 Undo Log를 같이 사용
    - Redo Log를 통해 변경의 상태 (롤백, 커밋, 도중 중단)을 판단
    - Undo Log의 데이터를 디스크에 복사

### Undo Log , Redo Log 그림

![Untitled](https://user-images.githubusercontent.com/84346055/275507942-1b23f7d6-2965-4fb9-b982-48f287c82854.png)

## 체인지 버퍼

- RDBMS에서 레코드가 삽입되거나 변경될 때 데이터 파일을 변경하는 작업에는 **인덱스를 변경**하는 작업도 포함된다.
- **B+ Tree 인덱스 자료구조**의 특성 상 리프 노드까지의 접근에는 **Random I/O 작업**이 수행
    - 인덱스의 크기가 큰 경우에는 많은 비용이 발생한다.

> 변경되는 인덱스 페이지가 버퍼 풀에 있다면 바로 업데이트 수행, 인덱스 페이지를 **디스크로부터 읽어와야 한다면 즉시 실행하지 않고 임시 공간에 저장** → 해당 공간이 체인지 버퍼
>
1. 중복 여부를 체크해야 하는 **유니크 인덱스는 체인지 버퍼를 사용할 수 없다.**
2. 체인지 버퍼에 저장된 인덱스 레코드는 백그라운드 스레드에 의해 병합 (머지 스레드)

**MySQL 5.5 이후**

- 작업(쿼리)의 종류 별로 체인지 버퍼 활성화 가능
- ***`all` :*** The default value: buffer inserts, delete-marking operations, and purges.
- ***`none` :*** Do not buffer any operations.
- ***`inserts` :*** Buffer insert operations.
- ***`deletes` :*** Buffer delete-marking operations.
- ***`changes` :*** Buffer both inserts and delete-marking operations.
- ***`purges` :*** Buffer the physical deletion operations that happen in the background.

[MySQL :: MySQL 8.0 Reference Manual :: 15.5.2 Change Buffer](https://dev.mysql.com/doc/refman/8.0/en/innodb-change-buffer.html)

## 어댑티브 해시 인덱스

- InnoDB 스토리지 엔진에서 사용자가 자주 요청하는 데이터에 대해 자동으로 생성하는 인덱스
    - InnoDB는 **B+Tree 검색 시간(O(logN))을 줄이기** 위해 도입
- Hash Index
    - 인덱스 키 : 데이터 페이지의 주소
        - **B-Tree 인덱스의 고유 번호** + B-Tree 인덱스의 키 값
- InnoDB 스토리지 엔진에서 어댑티브 해시 인덱스는 하나만 존재 (파티션 기능은 가능)
    - 모든 B-Tree 인덱스를 하나의 어댑티브 해시 인덱스에서 다루기 때문에 어느 인덱스인지 구분 필요
    - 그렇기 때문에 **B-Tree 인덱스의 고유 번호**가 포함된다.
- 데이터 페이지의 주소
    - InnoDB 버퍼 풀로 로딩된 페이지의 주소 (즉 버퍼 풀에 올려진 데이터 페이지만 관리 대상)

### 만능은 아니다….

1. 디스크 읽기가 많은 경우
2. (조인 or Like 패턴) 검색이 많은 경우
    - Hash 자료구조 특성상 범위 검색과 패턴 검색에 취약하다…..

### 효과적인 경우

1. 디스크의 데이터가 InnoDB 버퍼 풀의 크기와 비슷한 경우
2. 동등 조건 검색 , IN 연산이 많은 경우
3. 일부 데이터에 쿼리가 집중되는 경우
