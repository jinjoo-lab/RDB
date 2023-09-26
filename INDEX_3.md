# Index -3 (Covering Index)

### Covering Index(커버링 인덱스)

- 쿼리를 수행하는데 있어 필요한 모든 데이터를 가지고 있는 **인덱스**
    - 커버링 인덱스를 효과적으로 사용한다면 **추가적인 데이터 접근은 필요하지 않다 !**

> SELECT, WHERE, ORDER BY, GROUP BY 등에 사용되는 모든 컬럼이 인덱스의 구성요소인 경우
>

- 예시 테이블

```
CREATE TABLE students (
	id
	name
	grade
	school
)
```

- id가 PK라고 했을 때 Index가 생성될 것이다.

> grade에도 index를 생성하고 grade =3 인 학생을 조회해보자 !
>

```
SELECT * FROM students WHERE grade = 3;
```

1. grade를 통한 검색 시 grade에 생성한 index를 통해서 빠르게 검색
2. grade가 아닌 name , school 칼럼에 대해서는 PK를 통한 2차 검색 필요
3. 결국 인덱스를 통한 검색 과정이 2번 수행된다 !
- **WHY ?**

  grade를 통해 생성한 index의 리프노드에는 PK만 저장되어 있다. 그렇기 때문에 name , school 등의 컬럼 정보를 가져오기 위해서는 **PK를 통한 2차 검색**이 필요하다.

    - grade를 통해 생성한 인덱스 → 논 클러스터링 인덱스
        - 리프 노드에는 **PK값이 저장**
    - PK로 생성된 인덱스 → 클러스터링 인덱스

![Untitled](https://user-images.githubusercontent.com/84346055/270674762-270ea788-b887-4086-80d5-31f77998140f.png)

- 이것이 문제인 이유 ?

> 앞서 말했지만 데이터 베이스의 성능에 크게 영향을 주는 것은 Disk I/O이다. **PK값을 통한 추가적인 레코드 접근 시 많은 시간**이 소요된다.
>
- **해결 !**

  추가적인 (데이터 블록)에 대한 레코드 접근(Disk I/O) 없이 데이터를 조회할 수 없을까 ? → **커버링 인덱스**


```
SELECT grade FROM students WHERE grade = 3;
```

- 위와 같은 쿼리는 실행 시 grade로 생성한 index 탐색만 하여 추가적인 레코드 접근은 발생하지 않는다.

### 커버링 인덱스 + PK

```
SELECT id FROM students WHERE grade = 3;
```

- **위의 쿼리는 추가적인 INDEX 탐색을 할까 ?**

  NO !! , 한번의 인덱스 탐색으로 id값을 반환한다 → 커버링 인덱스의 사용


**WHY**

- InnoDB의 세컨더리 인덱스의 구조
    - B -Tree 기반의 논 클러스러링 인덱스
    - 리프 노드에는 ROWID가 아닌 **PK** 저장
