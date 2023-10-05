# Index -2 (클러스터링 인덱스)

## 클러스터링 / 논 - 클러스터링 인덱스

### 클러스터링 인덱스

---

> 데이터가 저장되는 물리적인 형태가 저장된 인덱스
>
- 테이블의 레코드를 프라이머리 키(클러스터링 인덱스)를 기준으로 묶어서 저장하는 형태
    - MySQL에서 클러스터링 인덱스는 **InnoDB 스토리지 엔진**에서만 지원
- 테이블의 **프라이머리 키**에만 적용
    - PK 값이 변경된다면 레코드의 **물리적 저장 위치가 바뀐다.**
    - 즉 PK 값에 따라 자동으로 정렬된다.
- 물리적 저장 위치를 결정하기 때문에 테이블에 대해서 **오직 한개만 존재**할 수 있다.
- 인덱스 자체의 리프 페이지가 곧 데이터 → 테이블 자체가 인덱스
    - 별도의 **인덱스 페이지는 필요하지 않다 !**

장점

- PK(클러스터링 인덱스) 기반 검색이 빠르다.

단점

- 세컨더리 인덱스를 통한 겁색 → PK로 다시 한번 검색 (성능이 좋지 않다)
- 레코드의 저장이나 프라이머리 키의 변경에 대한 처리가 느리다.
    - PK 변경 → 레코드를 DELETE + INSERT
    - 위치를 재설정해야 하기 때문 !

![Untitled](https://user-images.githubusercontent.com/84346055/270346084-1446cb9e-a23b-463d-b46f-400d0586cc6e.png)

- 위 그림과 같이 PK(emp_no) 값을 기준으로 리프 노드(페이지)에 데이터가 정렬되어 저장되어 있는 것을 알 수 있다.
    - 클러스터링 인덱스 구조도 결국 B -Tree 기반이다. 하지만 **리프 노드의 데이터가 실제 레코드**라는 것이 중요하다.

      > 클러스터링 테이블은 그 자체가 하나의 거대한 인덱스 구조로 관리되는 것 !
>

![Untitled](https://user-images.githubusercontent.com/84346055/270346118-46d32be8-7ed2-4d8c-bf2b-eb2a17434f39.png)

- **PK가 없는 InnoDB 테이블의 경우 ?**

  다음의 과정을 수행

    1. PK가 있으면 기본적으로 PK가 클러스터링 인덱스
    2. NOT NULL + UNIQUE INDEX 중에서 첫 번째 인덱스를 클러스터링 인덱스로 선택
    3. 자동으로 유니크한 값을 가지도록 증가되는 칼럼을 내부적 추가 → 사용자에게 보여지진 않는다.

![Untitled](https://user-images.githubusercontent.com/84346055/270346134-6f35ac2f-417d-4ac0-94d7-551cf3d5ffce.png)

### 논 -클러스터링 인덱스

---

- 클러스터링 인덱스의 반대 , 군집화 되어 있지 않은 인덱스
- 데이터 행에 독립적이다.
    - 레코드의 **물리적 저장 위치가 변경되지 않는다.**
        - 레코드가 정렬되는 것이 아닌 인덱스 페이지가 정렬되는 것이다.
        - 별도의 인덱스 페이지를 위한 **저장 공간**이 필요하다.
- 한 테이블에 여러개일 수 있다. → 세컨더리 인덱스
- 인덱스 리프 페이지는 데이터가 위치하는 포인터(**RID → Row ID**)
    - 클러스터형보다 속도는 느리지만 데이터의 입력 , 수정 , 삭제는 빠르다.

![Untitled](https://user-images.githubusercontent.com/84346055/270346144-395738b0-bce1-4e37-84bc-5f485afd5515.png)

### InnoDB 세컨더리 인덱스

---

- 클러스터링 인덱스를 사용하지 않는 경우
    - **레코드가 저장된 공간의 위치**는 **인덱스 테이블과 독립적**이다.
    - 레코드가 저장된 주소 → 내부적인 레코드 아이디(ROWID) 역할
    - PK나 세컨더리 인덱스의 키는 그 주소(ROWID)를 이용하여 실제 데이터 레코드를 가져온다.

- InnoDB(클러스터링 인덱스 사용 경우)
    - 클러스터링 인덱스가 **물리적인 저장 위치**를 결정 !
        - 변경되는 경우 → ROWID가 재설정된다.
    - 세컨더리 인덱스의 리프 노드에 ROWID를 저장하는 경우
        - 클러스터링 인덱스가 변경 → 인덱스에 저장된 주솟값(ROWID) 변경

> 오버헤드를 제거하기 위해 InnoDB 테이블(클러스터링 테이블)의 모든 세컨더리 인덱스는 레코드가 저장된 주소가 아니라 **PK값을 저장**
>

![Untitled](https://user-images.githubusercontent.com/84346055/270346153-7cf27493-5a12-42d6-bd52-bdc522b26fe4.png)