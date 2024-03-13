# RocksDB
* Paper : https://www.usenix.org/conference/fast21/presentation/dong
* Ref : http://165.132.172.98/wp-content/uploads/2020/07/space_amplification_7%EC%9B%94.pdf
* 
## Keyword
- Log-Structured Merge (LSM) trees
  - primary data structure to store data
  - 여러 단계의 SSTable로 구성
- MemTable
  - in-memory write buffer
  - 데이터가 최초록 저장되는 공간
  - Skiplist로 구현되어있어 삽입/탐색 시 복잡도, O(log n)
- Write Ahead Log (WAL) : Transaction 발생 전 Log 기록, Recovery(Redo, Undo)를 위해 사용
  - Undo : Transaction 이전 상태로 되돌리는 것
  - Redo : Transaction 이후 상태로 되돌리는 것 (메모리 O, Disk X 상황 복구)
- Sorted String Table(SSTable)
  - 새로운 SSTable은 MemTable flush에 의해 생성 되며, LSM Tree의 Level 0 위치
  - 상위 Level의 SSTable은 Compaction을 통해 생성
- Shards : Large-sacle distributed data services에서 데이터를 여러 서버노드에 분산하여 저장되는 단위, 각 서버노드 당 수십~수백개의 shards로 분리됨 (Load balancing and replication unit)
## Architecture
- Writes
  1) MemTable이 설정 된 수준까지 기록되면, MemTable 및 WAL은 불변상태가 됨(Immutable)
  2) 새로운 MemTable 및 WAL 할당
  3) MemTable의 내용은 SSTable 파일로 변경되어 디스크에 Flush 됨.
  4) Flushed MemTable과 WAL 제거(Discard)
  5) SSTable은 데이터를 일정한 크기의 블록으로 순서를 정렬하여 저장함
     1) 각 SSTable에는 Binary search를 위한 Index block 존재
     2) 각 SSTable block 당 1개의 Index entry 존재
- Read
  1) Last level까지 key를 탐색 (MemTable -> Lv 0 SST -> higher lv SST)
  2) 각 Level 내 에서는 binary search 수행
- Compaction
  1) 삭제 및 중복 데이터를 제거하여 Read 성능 및 Space 최적화
  2) 설정된 Level의 SSTable size가 초과하면 Lv-(L)의 SSTable 일부가 통합되어 Lv-(L+1)에 저장
  3) Level 0 에서부터 last level까지 점진적으로 마이그레이션 수행
- Compaction 종류 `추가 공부`
  1) Leveled Compaction (Space O/H 25% 이상)
  2) Tiered Compaction(Univeral Compaction)
     1) W.A 감소
     2) 낮은 Read 성능
  3) FIFO Compaction
  4) Dynamic Leveled Compaction (Space O/H 13%)
     1) Last level의 size를 기준으로 나머지 Level의 크기를 동적 조정
     2) LSM Tree의 Dead data(deleted, overwritten)를 줄여 Space efficiency 개선
## Resource
- Write amplification
  1) SSD : SSD 자체적으로 발생 (1.1~3)
  2) DB : page size(4, 8, 16KB) 보다 작은 write 발생 시 (~100~)
  3) Leveled Compaction : (10~30)
  4) Tiered Compaction : (4-10)
- Space amplification
  - SSD의 수명 및 성능은 충분하기 때문에 W.A 보다 S.A가 더 중요
  - Dynamic leveled compaction을 통해 space overhead 개선
- CPU Utilization
  - 비용 효율적인 시스템 구성을 위해 CPU 사용량을 고려해야함
  - RocksDB 설정에 따라 CPU 사용량은 변화함
  - 조화로운 시스템 구성하에 RocksDB 운용 시, CPU 사용량은 중요하지 않을 수 있으나, 충분한 SSD 성능으로 인해 Bottleneck point가 Storage -> CPU 바뀜. 반면, High-end CPU 를 사용한다면 SSD 성능을 충분히 활용할 수도 있음. 결국 시스템 구성 비용에 따라 달라짐.
  
## Adapting to newer technologies
- Open-channel SSD, Multi-stream SSD, ZNS : 소수의 App.에서만 개선 됨
- In-storage computing의 효용성 미지수
- Remote(Disaggregated) storage 연구 집중
  - 필요에 따라 프로비저닝하여 CPU 및 SSD resource 모두 최대로 사용하기 쉬움
- SCM 활용 `추가 공부`
    1) extension of DRAM : 어떤 자료구조(block cache, MemTable, ..)에 접목할 것인지와 persistency를 제공함에 따라 발생하는 overhead 고려
    2) Main storage of DB : I/O 보다는 space 와 CPU bottleneck이 중요하기 때문에 큰 의미 없음.
    3) for WALs : SSD로 이동되기 전의 작은 영역을 대체하는 수준으로 효용성 없음

## Lessons on serving large-scale systems
- Resource management
  - 1개의 Storage host에서 많은 RocksDB instances 동작하며, 모든 instances가 Single address space를 사용하거나 각 instance 마다의 address space를 사용할 수도 있음.
  - Instance가 host's resource를 공유하기 때문에, Globally(host)와 Locally(instance) 둘다 관리되어야함.
  - Single processor 에서 multiple instances가 동작할 때 resource limit을 잘 고려해야 하며, size 및 resource를 제한할 수 있는 thread pool을 사용하는 것이 좋음.
  - Global resource limit : 
    - (sub-optimal) 자원 사용을 보수적으로 접근하면 자원부족 문제는 없지만 전체 자원을 fully 사용하지 못함. 
    - (morre challenging) 각 인스턴스간의 자원 사용량을 공유하여 전체 자원 사용을 적절하게 조절함
    ```
    * memory for write buffer and block cache
    * compaction I/O bandwidth
    * compaction threads
    * total disk usage
    * file deletion rate
    ```
- WAL treatment
  - 전통적인 DB에서는 Durability를 위해 WAL을 사용하지만, Large-scale distributed storage system에서는 성능 및 가용성을 위해 데이터 복제함. 결과적으로 RocksDB WAL 불필요한 경우가 있음.
  - 여러 WAL option 제공 : `sync WAL writes`, `buffered WAL writes`, `no WAL writes at all`
- Rate-limited file deltetions
  - file deletion in RocksDB -> issue TRIM command in VFS -> occur GC in SSD internal -> negative impact foreground I/O latency.
  - 위와 같은 문제를 개선하기 위해 Compaction 이후에 여러 파일이 동시에 삭제되는 수준을 제한함.
- Data format compatibility
  - 매달 업그레이드 되는 RocksDB를 원활하게 운용하기 위해 RocksDB의 Backward/Forward compatibility가 중요함.
    - Protocol buffer
- Managing configurations
  - Code embedded 및 Version update에 따라 Configuration option이 바뀌는 경우 기존에 저장된 data file을 정상적으로 불러오지 못하는 문제가 있음.
  - 이를 해결하기 위해 database file과 함께 option file을 사용하며, validation tool과  migration tool을 제공하여 option file을 관리할 수 있음
  - 현실적으로 많은 사용자들은 어떻게 설정해야할지 모르기 때문에 기본 설정을 단순화하고, 성능을 높이려고 노력함. 궁극적으로 automatic adaptivity를 제공하려고 함.
- Replication and backup support
  - RocksDB를 사용하는 App.은 목적에 따라 Replication 및 backup 기능을 직접 구현하고, RocksDB를 이런 경우를 최대한 지원하려고 함
  - Replication
    - Logical copying
      - 모든 key를 source로 부터 읽어서 destination에 기록
      - Source side : Data scan 과정에서 caching 하지 않는 옵션등을 제공하여 동시에 처리중인 query의 영향을 최소화함
      - Dest. side : Bulk loading
    - Physical copying
      - SSTable 및 기타 파일들을 직접 복사
      - Block layer 및 FTL 등의 수준까지 직접 접근하는 것을 통해 RocksDB의 유의미한 성능향상을 기대하긴 어려울 것임.(득보다 실이 많음)
  - Backup
    - Backup이 Replication과 다른점은 Application이 여러 백업을 관리해야 하는 경우가 많다는 것임 `무슨말이지`
    - 대부분의 App.은 각자 목적에 맞는 Backup을 구현하므로, RocksDB는 간단한 Backup 기능만 제공
      - 다양한 Replica에 일관된 순서로 업데이트 -> 성능 문제 발생
      - 한번에 1개의 Write요청만 발생 -> 성능 문제 및 Replica가 변경점을 따라잡지 못하는 문제 발생
      - 궁극적으로 RocksDB는 user timestamp에 따른 Multi-versioning을 지원하지 않기 때문에 App.은 out of order write가 불가하고, 순서대로 snapshot을 읽는 것이 불하다는 한계가 있음. `개선해야할 부분`

## Lessons on failure handling
- Major lessons
  - Data corruption은 조기에 발견해야 데이터 유실을 최소화하고, 오류 발생 위치를 정확히 찾을 수 있음.
  - Integrity protection이 전체 시스템에서 보장되어야 silent corruption을 방지할 수 있음.
  - Error는 차별화된 방식으로 처리되어야 함
- Frequency of silent corruptions
  - 종합적으로 약 100 PB 데이터마다 3개월에 한번씩 corruption 발생했고, 그 중 40%는 replica로 전이 됨.
  - CPU/Memory corruption은 매우 드물게 발생
  - Data transfer 과정에서 software bug로도 corruption 발생가능 (약 17 checksum mismatch per PB)
- Multi-layer protection
  - 다양한 레벨에서 file data checksum을 통해 하위 레벨의 corruption 탐지
  - Block integrity : 데이터를 읽을 때마다 SSTable block or WAL fragment의 checksum 확인
  - File integrity : Data를 전송할 때마다 SSTable의 checksum 확인 (WAL file은 아님)
  - Handoff integrity : Write corruption을 조기에 탐지하기 위해 write data + checksum을 함께 file system에 전달하고, file system과 같은 lower layer에서 checksum 확인. (SSTable은 해당되지 않고, WAL만 해당)
  - Remote storage 에서 동작한다면 내부 ECC와 연결되어 checksum을 수행하도록 write API를 변경할 수 있고, write 시점에 검증하기 때문에 read time이 지연되는 경우는 극도록 적을 것임
- End to end protection
  - 위 방법의 한계점
    - File I/O layer 상위에서는 data를 보호할 수 없음(MemTable, block cache)
    - flush 및 compaction은 corrupted data를 영구적으로 저장하게 됨
  - Key-value integrity : key/value가 복사될 때, key-value checksum을 함게 전송하고, 대체 보호방법이 있는 파일은 이를 생략함

## Lessons on the key-value interface
- Benefit of Key-value interface
  - Generic : Key/Value에 자유롭게 정보를 저장할 수 있음. (parsing 은 App. 역할)
  - Portability : Key-value system간의 migration 쉽고 대부분의 경우 최적의 성능을 달성함.
- Versions and timestamps `추가 공부`
  - RocksDB는 Client write에 의해 순차적으로 증가하는 56-bit sequence number로 KV-pair의 버전을 관리
  - App.에 의한 Snapshot을 허용하고, App.에 의해 snapshot을 해제하기 전까지 snapshot 시점의 모든 KV-pair를 보존해야 하므로, Sequence number는 다르지만 동일한 key를 가진 KV-pair가 여러개 존재 할 수 있음.
    - 특정 시간을 지정하는 API가 없어 과거의 snapshot 지원하지 않음 
    - 특정 시간의 Read도 충분히 지원되지 않음
  - Instance마다 Sequence number를 할당하고, snapshot은 instance마다 얻을 수 있어 RocksDB instance인 복제된 Shard가 있는 App.은 버전관리가 복잡함.
  - App.은 이런 한계를 극복하기 위해 Key or Value에 timestamp를 encoding 하지만 성능 희생이 동반됨
    - Encoding within key : lookup 성능 하락
    - Encoding within value : 동일 key에 out-of-order write가 발생하여 성능하락 및 old version read 복잡해짐
  - 이를 해결하기 위해 Application-specified timestamp 기능 지원 `추가 공부`
    - Key/Value 외의 별도의 메타영역에 timestamp를 저장(?)


## Open questions
1. How can we use SSD/HDD hybrid storage to improve
efficiency?
2. How can we mitigate the performance impact on readers
when there are many consecutive deletion markers?
3. How should we improve our write throttling algorithms?
4. Can we develop an efficient way of comparing two replicas to ensure they contain the same data?
5. How can we best exploit SCM? Should we still use LSM
tree and how to organize storage hierarchy?
6. Can there be a generic integrity API to handle data handoff between RocksDB and the file system layer?


## 이해 못한 부분
- (Compaction) Level-0 SSTables have overlapping key ranges, as each SSTable covers a full sorted run. Later levels each contain only one sorted run so the SSTables in these levels contain a partition of their level’s sorted run

. 