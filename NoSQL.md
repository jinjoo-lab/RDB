# NoSQL VS SQL

# NoSQL

### NoSQL 주요 특징

- Schema-less
- JOIN 연산의 부재와 반정규화된 데이터 저장 방식
- 트랜잭션 특성이 RDB와 다름
- ACID 특성에 자유롭기에 수평 확장(샤딩)위 용이하다.

### ACID vs BASE

- BASE는 일반적으로 관계형 데이터베이스에서 따르는 트랜잭션 특성(ACID)과 반대되는 특성을 가지는 NoSQL 시스템에서 주로 따른다
    - Basically Available → 기본적으로 가용성을 제공
    - Soft State → 유연한 상태를 가진다
    - Eventually Consistency → 최종적 일관성을 가진다

### RDB vs NoSQL 비교 테이블

|  | NoSQL | RDB |
| --- | --- | --- |
| 장점 | Scale Out 용이, 상대적으로 빠른 성능 | JOIN 연산, 트랜잭션 보장, 정규화를 통한 데이터 중복 감소 |
| 단점 | 트랜잭션 제약X, 반정규화에 따른 중복데이터 증가 | ScaleOut 어려움, 상대적으로 느린 성능 |

## NoSQL 주요 종류

### Document 방식

- 테이블 대신 Document 개념으로 표현하는 데이터베이스
- MongoDB, Couchbase

### Key - Value 방식

- Dictionary 처럼 Key와 Value로 표현하는 데이터베이스
- Memcached, Redis

### Graph 방식

- 객체간의 관계를 그래프로 표현하는 방식의 데이터베이스
- Neo4j, Neptune

### 번외 - NewSQL

- 기존 RDB와 NoSQL의 장점을 합친 NewSQL 이라는 개념의 데이터베이스도 존재
- CockroachDB, Google Cloud Spanner 등이 대표적

[CockroachDB in Production](https://tech.devsisters.com/posts/cockroachdb-in-production/)
