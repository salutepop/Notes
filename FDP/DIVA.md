# Diva: Making MVCC Systems HTAP-Friendly
* paper : https://dl.acm.org/doi/10.1145/3514221.3526135
make any disk-based MVCC databases more HTAP-freindly.

## 요약
MVCC에서 version index 와 version data를 분리하여 version search/cleaning overhead 개선
  - version index는 p-leaf index pages로 관리
  - version data는 epoch based interval-tree 로 관리 (transaction - version data matchmaking)

## Todo
- DB system 동작 공부하기
  - Logging, Redo, Undo, Recovery, MVCC, Checkpoint, Cleaning,
  - ETL
  - isolation level (READ-COMMITTED)
  - ARIES : https://web.stanford.edu/class/cs345d-01/rl/aries.pdf

## Keywords
- MVCC : Multiversion concurrency control
  - 데이터의 업데이트가 필요할 때, 기존 데이터 항목을 새로운 데이터가 덮어쓰는 대신 데이터 항목의 새로운 버전을 만든다
  - Three concerns : recovery, version searching, version cleaning
  - Solution : Unified management(recovery target, search link, old version)
  - 성능 저하 요인 : low update rate, long query latency, space expansion
  - data version을 prompt cleaning하면 잦은 스토리지 접근, search link 복구가 발생하여 version search에 악형향을 줌
  - 많은 recovery target이 존재한다면, version cleaning과정에서 stale data를 식별하는 시간이 더 오래걸림
  - version searching을 위해 시간효율적인 version index필요
  - version cleaning을 위해 공간효율적으로 stale data를 식별하고 제거하는 방법 필요

- OLTP : Online Transactional Processing
  - 데이터베이스에 정보를 빠르고 안정적으로 저장하는 데이터 기술
  - 데이터 엔지니어는 OLTP 도구를 사용하여 트랜잭션 데이터를 관계형 데이터베이스에 저장

- OLAP : Online Analytical Processing
  - 다양한 관점에서 비즈니스 데이터를 분석하는 데 사용할 수 있는 소프트웨어 기술

- OLTP는 데이터베이스에서 여러 트랜잭션 스트림을 처리하고 저장하는 데 적합합니다. 그러나 데이터베이스에서 복잡한 쿼리를 수행할 수는 없습니다. 따라서 비즈니스 분석가는 OLAP 시스템을 사용하여 다차원 데이터를 분석

- OLCP : OnLine Complex Processing
  - Processing complex queries, long transactions and simultaneous reads and writes to the same record. Contrast with OLTP, in which records are updated in a more predictable manner.

- ARIES (Algorithms for Recovery and Isolation Exploiting Semantics)
  - `공부할 것` https://web.stanford.edu/class/cs345d-01/rl/aries.pdf
  - Simulator : https://mwhittaker.github.io/aries/
  - No-force : 지연 갱신 (Redo 필요)
  - Steal : commit없이 수정 사항 즉시 반영(Undo 필요)
  - Analysis -> Redo -> Undo

- Phantom read : 다른 트랜잭션에서 새로운 레코드를 추가/삭제 하는 경우 동일한 트랜잭션 내에서 조회 결과가 달라짐.
- 기존 해결책 : `ETL 관련해서 학습하고 다시 이해 하기` they build ETL pipelines, or its variants, to convert one data format emitted from OLTP workloads to OLAP- friendly columnar formats. Although such systems often maintain multiple dedicated instances for OLTP/OLAP loads and strive to reduce ETL latency between the instances, they offer the superior capability for performance isolation with HTAP.



## Background
- MVCC
  - Writer가 기록 중인 데이터에 Reader가 접근해도 기존의 데이터를 정상적으로 서비스하기 위해(혹은 반대 상황) 동시에 여러개의 버전을 관리함.
- Transaction recovery (ARIES)
  - (PostgreSQL) Main index에 old version 저장, 빠른 복구 가능, 잦은 index 재구성으로 비용이 큼
  - (MySQL) 기존 index가 유지되는 분리된 table space에 저장
  - 커밋된 데이터를 구성한 방법에 따라 Garbage data cleaning 방법도 다름
- Coupled concerns
  - delayed purging(stale version)으로 인한 공간낭비, version lookup에 영향
  - `undo log와 old version serving 연관관계 생각해보기` From the MVCC perspective, transaction undo logs become natural candidates for serving old versions of data. In this respect, garbage collection can clean both undo logs and stale versions all at once if they are unified.
- Unified management
  - version cleaning은 version lookup을 위해 search links를 복구하는 비용이 추가 됨
  - version lookup을 위해서는 version 제거에 취약한 검색구조를 필요로 함(?)
- Traditional disk-based databases
  - Undo logging
    - 분리된 공간에 기록 (MySQL, Oracle)
    - 메인 table에 기록 (PostgreSQL)
    - MVCC를 위한 unified version 저장 + recovery를 위한 추가적인 정보 기록 (Azure SQL)
- Modern key-value store
  - RocksDB : version search와 version removal 분리 됨
  - WiredTiger : no-undo 정책을 적용하여 version 관리의 recovery문제만 분리 (memory pressure 취약)
- Improvements to the space-time tradeoff
  - vDriver : 오랫동안 남아있는 트랜잭션을 MVCC의 recovery로부터 분리. space 관리에 집중되어 있고, version search를 덜 고려됨.
  - vWeaver : frugal skip list를 통해 large version set에서의 scan 동작을 향상시켰으나 구조가 바뀌지는 않음(coupled/coupled)
  - KVell+ : MVCC-Recovery와 Version search-cleaning 모두 분리
    - `이해안됨` OLCP query concept that propagates old versions directly to the concerned active transactions
- In-memory database engine
  - committed data를 제외한 모든 동작에서 I/O 비용이 없도록 하고, version 관리에 집중함.
- HTAP solution
  - 높은 가용성을 위해 데이터를 복제함.
  - analytic query로 인한 성능 하락을 방지하기 위해 복제본 사용(RC isolation)
  - `trace MVCC가 무슨 의미?` the systems [7, 68, 88] with the RC isolation seem to trade MVCC to reduce latent space overhead, despite phantom reads

## Design overview
  - Separation of recovery and MVCC
    - 가장 최근에 committed data를 version store에서 분리하여, recovery 과정에서 redo log만 관리하고 version store 접근 불필요 (SIRO-versioning)
  - Separation of search and reclamation
    - MVCC내에서 version index와 관련된 data version을 분리하여 관리
    - asynchronous cleaning (`원래 안되던건데 가능해진건가?`)
  - Design rationale for version indexing
    - data version은 임시 데이터 스트림이고, data lifetime은 snapshot이 겹치는 transaction과 연관됨
      - inode-like provisional version index 사용(내부 파편화 감소)
  - Design rationale for version cleaning
    - version cleaning을 위해 reference tracking을 하지 않도록 time interval 간격으로 version과 transaction을 관리하여, interval 내에 있는 모든 transcation이 모두 committed되면 interval 단위로 일괄 삭제함

## Provisional version indexing
- inode 유사점
  - indirect pointer를 통해 다양한 크기의 visible data version set을 접근할 수 있음
  - 사전 할당 된 index node pool 사용
- inode 다른점
  - 내부 파편화를 줄이기 위해 multi-granularity index array를 사용
  - short/long lived transaction에 따라 version의 window도 좁고 넓어짐

## Architecture
- Multi-granular index array
  - p-leaf는 2^2 ~ 2^8 수준의 index array capacity를 가짐
  - direct/in-direct locator가 있고, in-direct level 제한없음. 하지만 single in-direction으로 (2^8)^2 = 2^16 live transaction을 다룰 수 있어 2번의 I/O 동작을로 version locator를 찾을 수 있음
- Quasi-durable index space
  - version index는 failure상황에서 보존/복구될 필요없음 `(?)왜`
  - user concurrent stacks (lock-free?)
- Managing version index space
  - Resource allocation
    - record update 발생하면, 가장 최근 record version은 version storage로 이동됨.
    - primary index의 record에 이미 p-leaf array가 있으면 이거 활용해서 version pointer를 저장하고
    - 그렇지 않으면(처음 version store 하는 경우) 가장 작은(2^2) p-leaf array를 할당. array를 모두 사용하면 기존보다 2배 용량의 array를 새로만들어 old array를 모두 복사하고 old array는 재사용을 위해 p-leaf page로 되돌림(p-leaf page pool?)
    - array가 최대 사이즈를 초과하면, 가장 작은(2^2) indirect array를 만들어 첫번째 indirect locator는 최대사이즈를 초과한 기존 array를 가리키고, 두번째 locator는 새로운 array를 가리켜 더 많은 버전을 관리할 수 있도록 확장
  - Space compaction
    - live transaction이 garbage data version에 접근하는 일은 없으므로, version index만 고려
    - Macro compaction : most space is filled with stale locators
      - garbage page로 꽉찬 index file에 대해서만 수행
      - begin transition : 새로운 index file 생성
      - grace period : 이전 index file은 read-only가 되고, 새로운 index file에 index 생성/조회 함.
      - end trasition : 이전 index file 삭제 (이전 index file에 접근하는 live transaction이 없으면)
    - Micro compaction : dead space inside an index array
      - array 사용량이 적으면 더 작은 array로 복사한 뒤 기존의 array는 pool로 반환
  - Managing concurrency
    - contention between writers
      - (race condition) data version을 array에 처음 저장하거나 resize될 때, 새로운 index array를 위한 p-leaf page를 획득하는 writer와 free index array를 탐색하기 위해 동일한 page의 bitmap에 접근하는 writer간 경합
      - (solution) concurrent stack을 통해 p-leaf page를 pop 성공한 writer만 접근
    - contention between reader and wirter
      - (race condition) reader가 p-leaf array 순회하는 동안 writer가 동일 array에 micro-compaction 수행하거나, index array resize후에 p-leaf array locatoer를 수정하는 경우
      - (solution) reader-writer lock per record

## Version garbage collection
- batch deletion
  - 삭제할 대상(version) 식별 : interval based matchmaking (version-transaciton)
  - cleaning I/O를 줄이기 위한 구조 : 계층화된 버전
- Matchmaker
  - binary time-interval tree에서 고정된 time interval(epoch)마다 새로운 leaf는 시간순서의 오른쪽에 생기고, 부모 노드는 자식 노드의 범위를 모두 포함하고 있음. 각 leaf는 해당 interval 동안 시작된 live transaction의 수를 알고(bookkeep), null 참조인 경우 interval 내에 있는 모든 버전이 불필요함을 보장함. leaf의 ref. counting이 null 되기까지 기다렸다가 data version을 제거하기 때문에, 삭제할 대상을 식별하기 위한 비용이 들지 않음
  - background worker : macro-compaction p-index, modify interval tree(insert/delete)
- Rotation-free interval tree
  - 삽입은 오른쪽에서만, 삭제는 어디에서든
  - lock-free insertion
    - 오른쪽부터 순회하며 오른쪽 subtree보다 넓은 범위를 커버할 수 있는 subtree를 찾음
    - 해당 노드와 오른쪽 subtree 사이에 새로운 부모노드 삽입하고, 부모노드의 왼쪽에 오른쪽 subtree를 연결
    - tree의 균형이 잘맞으면, root에 새로운 부모노드 생성
  - lock-tree deletion
    - 왼쪽/오른쪽 subtree 모두 live transaction의 참조가 없는 경우 삭제(삭제 전 proxy)
    - child to grand parent node가 연결되고, parent node는 child가 proxy 함
  - Interval height analysis
    - log2(epochs spans), 4 epochs(leaf node)인 경우 height는 2를 넘지 않음(worst case)
- Sift-Bind-Trim version cleaning
  - version sifting `잘 이해안됨` : 비슷한 lifetime version 끼리 clustering
  - Transaction binding
    - transaction 시작 시, 현재 interval에 transaction을 연결하고, ref count 증가
    - termination 시, ref count 감소시키고, node를 null ref로 설정하여 background worker에 의해 삭제 될 수 있게 함
  - Garbage trimming : Sift-Bind phase(matchmaking)덕분에 reference 추적이 불필요 해짐. background worker에 의해 모인 garbage queue에서 dequeue하는 것으로 버전클리닝이 수행 됨.

## Design principles in action
- Provisional version index in practice
  - p-leaf index와 interval tree 공간 불일치를 지표로 삼아 macro-compaction 수행
  - interval tree는 동적으로 커지거나 작아지는데 반해, p-leaf index space는 계속 증가하기만 함.
- Version garbage collection in practice
  - version이 time interval의 걸치는 위치에 따라 gc 시점이 달라져서, root to leaf node까지 한번의 스캔으로 version을 이동시킴
  - version이 ancestor time interval 전체에 걸쳐있으면, ancester node에 위치
  - version이 자손 노드에 걸쳐있으면 subtree를 회귀적으로 탐색하여 node
  - long running query(analyic)인 경우 상위 레벨 노드에 위치한 버전은 gc가 지연될 수 있기 때문에 sift가 의미있음 `이해한게 맞나?`

## Experiment
- Index / Version 분리를 통해 더 적은 메모리를 사용하면서도 더 많은 트랜잭션을 처리할 수 있음.
- vanilla MySQL의 경우 sysbench에서 data version이 계속 증가하여 buffer capa.(40GB)를 초과하면 version search과정에서 read overhead로 인해 latency가 매우 길어짐

## Memo
- Log-structured 기반이 아닌 database에서도 sequential read/write 및 stream 분리의 장점을 살릴 수 있다면?
- memory pressure 상황을 storage가 커버할 수 있는 부분은 없을까? (redo-only)
- Recovery가 자주 일어나는 것인가?
- version cleaning 할 때, 가장 최근 데이터(페이지?)만 메모리로 읽어두고, 나머지 삭제(RU 단위로)
- array size마다 page pool은 사전에 정해진건가? 아니면 그때그때 array를 생성해서 쓰는건가? (array 갯수가 bitmap과 연관될 것 같아아서 사전에 할당되어야 할 것 같음.)


## 질문
- version garbage collection은 undo log + stale version을 모두 삭제 하는 것?
  - undo log / stale version은 다른 것?! 같은 것?
- Disk based DBMS 반대는 in-memory DBMS?
- version visibility range의 의미 (live transaction이 참조중인 version ?)
- (Ch 1)
  Data versions are con- tinuous and visible for a sliding time window, fitting the working set model, and they would never survive failure but be durable while alive, evading the duty of database logging and recovery.
- (Ch 6) long running query(analytic)인 경우 상위 레벨 노드에 위치한 버전은 gc가 지연될 수 있기 때문에 sift가 의미있음