# InnoDB
* Link : https://dev.mysql.com/doc/refman/8.0/en/innodb-introduction.html

## 특징
- Transactions
  - are atomic units of work that can be committed or rolled back.
- ACID
  - A: atomicity
    - commit, rollback
  - C: consistency
    - doublewrite buffer
    - crash recovery
  - I: isolation
    - transaction isolation
  - D: durability (HW에 따라 지원 범위가 달라짐 - UPS, Network..)
    - fsync
    - write buffer
- Row-level (행 단위) locking
- Clustered Index(Primary key) and Secondary Index
  - 인접한 index의 데이터가 클러스터링 되어있어, disk I/O 최소화


## Multi-versioning
- concurrency & rollback을 위해 변경된 행(row)의 old version 유지
  - undo tablespace에 rollback segment로 저장 됨
- Undo log
  - Insert undo log : transaction rollback에만 필요, commit후 discard(삭제)
  - Update undo log : consistent read를 위해 사용, snapshot 없는 경우만 discard
- Purge : update undo log를 discard할 때, row & index record를 물리적으로 제거

## Undo tablespace
- instance 초기화 시, 기본적으로 2개의 undo tablespace 생성
- tablespace size : 10MiB (~8.0.23), 16MiB(8.0.23~)
- Transaction이 오랫동안 실행 되면, undo log가 커질 수 있고, undo tablespace도 매우 커질지 수 있으므로 적절히 undo tablespace를 추가 하는 것이 좋음
- Drop undo tablespace : undo tablespace가 비워져있어야 하고, inactive mark로 변경하여 새로운 rollback segment가 할당되지 않도록 한 뒤 기존 rollback segment가 할당 된 transaction이 모두 종료되어 empty 상태가 되면 drop 가능
- Moving undo tablespace : 