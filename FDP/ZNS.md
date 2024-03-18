# ZNS
* Paper : https://www.usenix.org/conference/atc21/presentation/bjorling

## Keyword
- Zone : Zoned storage의 기본 단위이며, read는 자유로우나 write point에 의한 sequential write만 가능

## The Block Interface Tax
- Flash-based SSD : write in page unit, erase in block unit (A block comprising mulitple pages)
- HDD target 하여 만들어진 Block interface를 지원하기 위해 FTL 구현이 복잡해지고 비용 및 성능에 영향을 줌 : in-place upate를 하려면 기존 영역의 일부를 invalid 하고, 새로운 영역에 write 한 뒤 Host Logical address와 Device physical page를 연결(mapping)하고, garbage collection(GC)을 통해 invalid data를 모아서 새로운 영역으로 옮기고, 해당 block을 erase 해야함.
- 문제점
  - GC와 같은 FTL 내부동작으로 인해 성능을 예측할 수 없음.
  - GC를 수행에는 내부 복사가 필요한데, 이를 위한 예비 영역(Over provision)이 최대 28%까지도 요구됨.
  - Logical to Physical address mapping을 위해 GB수준의 DRAM이 요구 됨.

## Existing Tax-Reduction Strategies
- Stream SSD : Write command에 stream hint를 함께 전달하여 Stream hint별로 Erase block 단위로 기록될 수 있도록 함. Data의 수명/속성에 따라 hint를 부여하여 비슷한 데이터가 함께 Erase될 수 있도록 하였으나, 한 Stream 내에 수명이 다른 Data가 섞일 수 있기 때문에 결국 기존의 block-interface SSD와 비슷하게 OP 영역 및 DRAM이 필요함.
- Open-Channel SSD : Host의 연속된 LBA chunk들과 OCSSD의 erase block의 단위를 일치시켜 운용하여 SSD 내부의 GC가 불필요해지고 OP 영역 및 DRAM 비용도 절감할 수있는 장점이 있으나, wear-leveling와 같은 SSD 안정성과 같은 특성을 모두 Host가 관리해야하기 때문에 다양한 SSD를 혼재시키는 환경에서 Host가 이를 모두 관리 및 보장하기가 어려움.
- ZNS는 SSD media와 Device interface 불일치 문제 해결 목적으로 개발

## Tax-free Storage with Zones
- State machine(per zone)
  - EMPTY : begin state
  - OPEN : (EMPTY-OPEN) write 시작 부터 fully written이전
  - CLOSE : (OPEN-CLOSE-OPEN) Open zone limit 이상의 zone이 동시에 OPEN 상태이면, 새로운 zone을 위해 기존의 open zone을 CLOSE로 변경. 다시 OPEN으로 변경 후 이어서 write 가능
  - FULL : (OPEN-FULL) fully writen
- Write pointer (per zone)
  - EMPTY 및 OPEN zone에서만 유효하며, write 성공 시마다 갱신
  - Write pointer에서 시작하지 않거나, FULL state에서 요청된 write는 수행하지 않음
  - reset 시 EMPTY state로 전이 되며, write pointer는 zone의 첫번째 LBA로 갱신 됨. 이전에 기록된 데이터는 더이상 접근할 수 없음.
  - write pointer를 사용하면 last LBA를 추적할 필요 없음. (recovery)
- Writable zone capacity
  - Zone size보다는 작은 Zone capacity를 가질 수 있으며, 실제로 Zone capacity만큼 write 할 수 있음. (zone size는 업계 표준을 따라 2의 제곱 단위로 맞춤)
- Active zone limit
  - 모든 zone을 writable 상태(OPEN/CLOSE)로 유지할 수 있는 SMR HDD와 달리 Flash based media는 program disturb 같은 특징들 때문에 writable zone(OPEN/CLOSE)의 수를 Active zone limit으로 제한

## Hardware impact
- Zone sizing
  - Write capacity와 SSD 내부의 erase block은 직접적으로 연관됨
  - 1개의 erase bock은 16~128개 die에 걸쳐 striping
  - Die-level protection(parity)을 지원하고, low I/O Queue depth에서 적절한 성능을 낼수 있는 가능한 small zone을 사용하는 것이 host의 데이터 관리가 용이함.
- Mapping table
  - block-interface SSD는 GC 성능을 위해 fine-grained fully associative mapping table을 관리하지만, table size는 1GB/1TB 수준임.
  - ZNS는 zone단위로 sequential write하기 때문에 보다 coarse grained mapping table로도 관리할 수 있음. (block erase level, hybrid)
  - SSD DRAM의 많은 양을 차지하는 Mapping table을 줄여 DRAM을 아낄 수 있음.
- Device resources
  - SSD 내부의 XOR engines, Power capacitors 등의 한계가 있기 때문에 Active zone의 수는 8-32개 정도로 기대 됨.

## Host software adoption
- Host-side FTL(HFTL)
  - FTL의 역할 중 mapping과 GC에 대한 관리만 수행
  - HFTL을 사용하면 Host-side의 정보를 통합 관리하고, 기존 block interface 장치처렴 동작시킬 수 있음.
  - HFTL로 `dm-zoned`, `pblk`, `SPDK's FTL` 등이 있지만 `dm-zap`만 ZNS를 지원함(2021년).
- File system
  - Zone을 storage stack의 상위 계층과 연결시키면, HFTL 및 FTL과 같은 overhead를 제거할 수 있음.
  - 대부분의 file system은 in-place write를 주로 사용하여 Zoned storage 모델에 적용하기 어려움. 반면 `f2fs`, `btrfs`, `zfs`등은 이미 ZNS를 지원하기도 함
  - `f2fs`, `btrfs`는 ZAC/ZBC에 정의된 zone model을 지원하지만, ZNS는 지원하지 않아 이를 추가함
- End-to-end data placement
  - Application이 data 배치를 관리할수 있도록 하여, 파일시스템 및 기타 계층의 오버헤드를 제거할 수 있음.
  - App.과 ZNS SSD간의 End-to-end data placement를 통해 최고의 성능을 달성할 수 있음.
  - Sequential write pattern을 갖는 App.은 ZNS와 end-to-end integration 가능. `RocksDB`, `CacheLib`, `Ceph SeaStore`
  - RocksDB의 zone storage back-end인 ZenFS 개발

## General Linux support
- Zone capacity (writable capacity)
  - Zone capacity attribute 추가하기 위해 zone descriptor data structure를 확장
  - fio는 zone capacity를 초과하지 않도록 수정
  - f2fs는 여기에 더해서 2가지 세그먼트 종류 추가
    - unusable segment
      - zone의 unwritable 영역의 segment
    - partial segment
      - zone의 writable / unwritable 영역이 교차되는 segment
      - segment chunk size와 zone capacity가 align되지 않은 경우에도 writable capacity를 모두 활용할 수 있음.
  - SMR HDD와 호환성을 위해 zone capacity는 zone size로 초기화 됨
- Limiting Active zones
  - fio에서는 active zone의 제한을 두지 않았으며, 사용자가 판단해야함
  - f2fs에서는 active zone는 동시에 열수 있는 segment 수와 연결 됨
    - file system 생성시 limit 설정 되며, 최대 6개로 제한
    - Slack space recycling(SSR, random write)은 ZNS의 성능 하락문제로 disable함.

## RocksDB Zone 
[RocksDB note](./RocksDB.md)
- RocksDB with ZNS
  - ZenFS(storage backend)를 통해 RocksDB와 ZNS를 end-to-end data placement하도록 구현
  - RocksDB의 Log structured merge tree 구조를 바탕으로 I/O가 sequential한 compaction 과정의 이점을 취함
  - RocksDB는 file system wrapper API를 통해 storage backend 분리를 지원 함

## ZenFS architecture
- Journaling and Data
  - Journal zone : 파일시스템의 상태를 복구하기 위해 사용. super block을 유지하고 WAL과 data file을 zone과 mapping함
  - Data zone : file contents 저장
- Extents
  - block align된 가변 크기의 연속된 영역
  - RocksDB의 data file은 set of extents와 연결되어 write됨
  - Zone 내에서 여러개의 extent를 가질 수 있으나, 여러 zone에 걸쳐있을 순 없음
  - Extent 할당/해제는 in-memory 구조체에 기록되고, fsync call에 의해 영구적으로 저널됨
- Superblock
  - Disk로부터 ZenFS state 초기화 및 복구 시 진입점
  - 파일 시스템 식별을 위한 UUID, Macig number, user option 정보 포함
- Journal
  - Journal은 super block을 관리하고, WAL 및 data file을 zone과 mapping(through extent) 하는 역할
  - Journal state는 device의 첫번째 2개의 non-offline(무결한) zone에 저장 되고, 언제든 1개의 zone이 active journal zone으로 선택되고 저널의 상태를 영구적으로 업데이트 함. 
  - 초기 저널 상태는 외부 유틸리티에 의해 생성되고, first journal zone에 초기 header 정보를 씀
  - RocksDB에 의해 아래의 복구과정을 수행하여 ZenFS를 초기화 한 후 데이터에 접근할 수 있음.
  - Header
    - Journal의 특정 zone 시작부분에 위치
    - Journal의 header가 저장 된 후, 잔여 writable capacity는 journal을 업데이트를 기록하는데 사용
    - Header 정보
      - 새로운 journal zone 초기화 시 증가하는 Sequence number
      - super block data 구조
      - journal state snapshot
  - Journal state 복구 과정
    1) 2개의 Journal zone의 첫번째 LBA를 읽어 각각의 sequence number 결정. (높은 값이 active zone)
    2) Active zone의 head 전체를 읽고, initial super block과 journal state 초기화
    3) Journal update는 header의 journal snapshot에 적용 됨.
  - The amount of updates
    - OPEN/CLOSE state : write pointer value 까지만 journal에 기록
    - FULL state : header 이후의 모든 레코드를 journal에 기록
- Writeable Capacity in Data Zones
  - (Ideal case) File size와 zone의 writable capacity가 align된다면, 최대용량을 사용할 수 있음.
  - (Real case) RocksDB에서 file size는 compression 및 compaction 결과에 따라 달라지기 때문에 writable capacity와 align되기 어려움. user option으로 zone capacity의 finish limit을 설정(5%)하여 Device zone capacity의 95% 크기까지 file size를 채울 수 있도록 함. file size가 limit을 초과하면 zone allocation algorithm에 따라 사용가능한 capacity를 모두 사용
  - 일반적으로 Zone capacity(2GB)는 RocksDB 권장 파일 사이즈인 128MB보다 커서, SST file size를 512MB까지 늘려도 성능 하락은 없었음.
- Data Zone selection
  - WAL과 SST Levels를 write lifetime hint로 구분
  - 파일을 처음 쓸 때, Device의 Data zone 할당
  - write 하려는 file의 lifetime이 zone에 저장되어있는 data의 max lifetime보다 짧으면서 가장 가까운 zone을 할당.
  - 조건에 해당되는 zone이 없으면 empty zone 할당
  - (실험) zone selection algorithm과 user-defined writeable capacity limit을 사용하여 space amp.나 unused zone space를 10% 수준으로 유지할 수 있음
- Active zone limits
  - ZenFS를 사용하기 위해 최소 3개의 Active zone 필요. `journal`, `WAL`, `compaction`
  - (실험) 6개 active zone에서 잘 동작하고, 12개 이상의 active zone은 더이상 성능향상 없음
- Direct I/O and buffered writes
  - Direct I/O
    - SST file
  - Buffered writes
    - 나머지 파일(WAL, ..)
    - file close or full 될 때 flush 수행
    - flush 시, buffer는 block 경계까지 padding되어 유효한 extent만큼 journal에 저장됨.
    - padding으로 인해 약간의 write amp. 증가하지만, ZenFS 뿐아니라 기존의 파일시스템도 동일한 문제를 갖고 있음.