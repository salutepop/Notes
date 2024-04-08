# eZNS
* Paper : https://www.usenix.org/conference/osdi23/presentation/min

## Keyword
- head-of-line blocking(HOL) : network에서 FIFO input buffer를 사용하는 경우 앞선 패킷이 전송되기 전까지 뒤에있는 패킷은 전송할 수 없음
## Flash memory
- 특징
  - no in-place update
  - read/write 성능 비대칭
  - limited lifetime
- FTL
  - Function
    - dynamic LBA to PPA mapping
    - Garbage collection
    - wear-leveling
  - Overhead
    - Mapping table을 위해 많은 양의 DRAM 필요
    - User I/O 처리 중 내부 GC 및 wear-leveling 발생 (WAF 증가)
    - mixed I/O 인 경우 각 I/O 요청간에 성능 영향을 줌
## ZNS
- Namespace
  - NVMe Device 내에서 독립된 address space. 기존에는 여러개의 block으로 구성되었으나, ZNS에서는 여러개의 Zone으로 구성
  - ZNS SSD에서 namespace는 각자의 active resource를 갖고 있음. `active resource ??`
- 장점 3가지
  - coarse-grained mapping
  - device-side GC 제거 + OP 영역 제거
  - deivce channel/die 별로 open zone할당하여 inter-zone interference 제거
- Write cache
  - user I/O size와 NAND program unit 등을 align하기 위한 DRAM 영역
  - write cache의 정보는 전원이 차단되는 상황에서도 NAND에 기록함
  - 최대 Active zone 수는 write cache size에 따라 달라짐
- I/O command of zone
  - read
  - sequential write
  - append : host i/o request에 LBA를 전달하지 않고, response에서 device로부터 LBA를 전달받아 확인
- Small / Large zone
  - Physical zone : 1개 die내에서 1개 이상의 block으로 구성된 가장 작은 단위
  - Logical zone : 여러개의 physical zone을 striping하여 대역폭 향상
  - Large zone : 여러 die에 걸쳐 striping된 large logical zone.
    - 전체 Zone 수가 적기 때문에 할당/관리 용이함
    - Host/App. 이 사용할 수 있는 Zone이 적음
    - Acitve zone수가 적기 때문에 적은 수의 tenant 사용 시 적합
    - fixed striping으로 인해 다양한 워크로드에서 die/channel간 대역폭 활용이 어려움
  - Small zone : physical zone 수준으로 구성 됨
    - multiple blocks on single die
    - 더 많은 Active resource 제공 (실험에서는 256 zones)
    - zone-reclaiming latency 낮음 `zone-reclaiming ?`
    - small zone일 수록 latency 유리한 연구 결과 (ZNS+)
- Limitation
  - zone striping에 따라 성능이 결정 됨
  - 고정된 zone 구성은 다양한 워크로드에서 최적으로 동작하기 어려움
    - Device가 많은 Acitve zone을 지원해도, App.에서는 제한된 수만 활용
    - 1개의 write는 1개의 zone에만 write하여 bandwidth를 충분히 활용하지 못함
    - `이해필요` > tenant당 logical zone 1개만 쓰는 문제(뒤에서 v-zone으로 극복!)
    ```
    file systems like BtrFS [36] and F2FS [25] support ZNS SSDs but write user data to only one zone at a time, resulting in suboptimal utilization of the available active resources. This issue is further exacerbated when the device has multiple namespaces serving different applications.
    ```
  - zone은 성능적으로 완벽히 독립되지 않음.
    - 서로 다른 zone간에 성능적인 영향을 줌. (=각 tenant마다 동일한 성능을 보장하지 못함)
    - OCSSD와 달리 ZNS는 zone allocation 및 wear-leveling을 내부적으로 하기 때문.

## Typical system model
- 5-layers (top-down perspective)
  - layer 1 : 동작중인 여러개의 storage apps. (multi-tenants)
  - layer 2 : 각 tenant는 1개 이상의 namespace 점유
  - layer 3 : namespace는 독립적으로 가변 가능한 logical zones들로 구성
  - layer 4 : logical zone은 여러개의 physical zone으로 구성되어 있으며, namespace에 따라 용량과 병렬성을 설정할 수 있음
  - layer 5 : physical zone은 1개의 channel/die에 위치
- Zoned block deivces(ZBD) layer
  - ZNS의 구조를 추상화 함
  - 3가지 기능
    - namespace/logical zone 상에서 app.과 상호작용
    - app. 요구사항에 따라 logical to physical zone 구성 조율
    - I/O commands 스케쥴링을 통해 device 최대성능 사용 및 `avoid head-of-line blocking`

## Zone striping
- RAID 0와 같이 high throughput을 위한 기법
- data block들이 여러 physical zones에 걸쳐있고, 동시에 접근
- config. parameters
  - Stripe size : 가장 작은 data 배치 단위
  - Stripe width : active state인 physical zone의 수로 bandwidth 제어   
- Basic performance
  - outstanding I/Os 가 충분히 많으면 최적의 stripe size는 NAND operation 단위(page size)
  - READ
    - stripe width가 클수록 bandwidth 높음
    - stripe size가 page size보다 작으면 bandwidth 급격히 낮아짐
  - WRITE
    - stripe width가 클수록 max. bandwidth 높음
    - stripe size는 큰 영향을 주지 않음 (동일한 physical zone마다 write가 병합되어 I/O 요청 됨)
- Challenge
  - I/O workload에 따라 최적의 stripe size/width를 결정해야 함
  - stripe width 클수록 active zone을 많이 필요로 하고, app.의 동시성을 악화시킴
  - striping된 logical zone은 높은 bandwidth를 가질 수 있지만, total active zone을 고려해서 striping 되어야 함.
  - `이해필요` > stripe size-width trade off 필요
  ```
   An ideal strip size can be the NAND page size, but it also has to be adjusted to the stripe width to provide a consistent full stripe size.
  ```

## Zone allocation and placement
- physical zone 할당 시, 사용가능한 Die를 찾고 wear-leveling을 고려하여 Die내에서 P/E cycle이 적은 block을 선택함
- Basic performance
  - stripe size 16KB 고정, logical zone내의 physical zone수를 증가시킴
  - READ 2MB/QD2 조건은 PCIe BW 제약
  - READ 2MB/QD1 조건은 outstanding I/O 부족
  - READ 4KB/QD32 조건 및 WRITE 2MB 조건은 physical die 성능 제약
  - 실험 결과 Channel/PCIe 대역폭을 최대한 활용하려면 Physical zone수가 40~80개 필요
- Challenge
  - 새로운 영역을 할당할 때 기존 할당내역이나 tenant간의 동작을 고려하지 않고 사용가능한 die를 할당하여 모든 내부 I/O Parallelism을 tanent가 활용하기 어려움.
  - Channel-overlapped placement : 여러 zone이 채널을 중복하여 할당된 경우 채널 병렬성이 제한됨
  - Die-overlapped placement : Namespace내에서 1개 다이를 중복하여 할당한 경우

## I/O execution under ZNS SSDs
- background GC 제거
- ZNS SSD는 물리적으로 파티션되어있지 않고, per die/channel 성능이 낮음
- Logical zone에서 sequential write 및 random read 발생할 때, die/channel contention으로 인해 read 성능 하락(latency spike)
- Basic performance
  - 동일한 HW(Channel/Die 구조)를 가진 CNS/ZNS 비교
    - CNS : 70% filled with 128KB random writes (precondition)
    - ZNs : 128 zones with 16KB stripe size (configuration)
  - 시나리오 : 128KB random read * 8 threads + `fixed rate..?` sequential write * 1 thread
  - ZNS의 경우 Write양을 증가시킬수록 read bandwidth가 부족해져 latency 길어짐
  - CNS의 경우 내부 GC로 인해 write amp.증가하여 read bandwidth 부족해져 latency  길어짐
- Challenge
  - Die 내에서 여러 zone이 불규칙하게 사용됨
  - Die 1개의 성능 낮음
  - HOL blocking문제와 logical zone 성능 저하 발생
  - Write cache를 초과하는 경우 NAND die로 flush하는 program bandwidth 중요함
  - 모든 active zone을 중앙에서 관리 및 zone 별 I/O scheduling을 해야 Die 경합을 줄일 수 있음
  - QD 8 with 2 Zones와 QD 2 with 8 Zones 비교
    - `Figure 9 이해 안됨`


## eZNS (Enabling an Adaptive Zoned NS)
- Overview
  - Zone arbiter
    - Hardware abstraction layer(HAL)
    - 중복없이 zone 할당
    - 동적 Zone 구성 변경
  - I/O scheduler (tenant-cognizant)
    - delay 기반의 read 요청 제어
    - token 기반의 write 요청 제어
- Hardware contract and HAL
  - small zone
    - physical zone : 1개 die내에서 1개 이상의 blocks (erasure unit) 으로 구성
    - max. active physical zone의 수는 die 갯수 * 2배
    - 모든 die가 완전히 사용되면, 모든 다이는 동일한 수의 active zone을 갖게 됨
    - 모든 die에 physical zone을 균등하게 분산 시킴
    - ware-leveling 요구사항에 따라 zone 할당 (모든 die를 순회하며 zone할당)
    - 정확한 channel/zone의 정확한 구조를 알지 못해도 문제 없음
  - HAL(shadow device view)
    - 대략의 data locality를 노출함
    - 3 hardware parameters
      - maximum number of active zones (MAR, maximum active resources)
        - physical die의 배수로 결정
      - NAND page size for striping
        - seq. read에서 stripe를 효율적으로 활용하기 위해
      - physical zone size
        - logical zone과 strip group 결정을 위해
- Serial zone allocator
  - 3 guarantees
    - stripe group은 연속된 physical zone으로 구성됨
    - stripe group내에서 die collison 없음
    - active zone이 모든 die에 걸쳐 완전 사용되며 write 발생하는 경우만 stripe groups간에 die collision 있음.
      - Channel bandwidth는 die전체의 program bandwidth보다 크기 때문에 channel collision은 고려하지 않음
  - Allocator 동작
    - device 당 request queue를 갖고 있고, 모든 logical zone의 OPEN command를 버퍼링함
    - OPEN command가 physical die의 zone할당을 보장하지 않기 때문에, zone reservation 개념을 도입
      - zone reservation : one block 강제로 flush 하여 die와 zone 연결
    - 일정량의 reserved zones을 순서대로 stripe group에 할당
    - 할당이 완료되면 allocation history를 metadata block(영구적인 영역)에 기록
    - 결과적으로 logical zone간 channel overlapped 문제 발생하지 않음.
- Zone ballooning
  - v-zone
    - logical zone과 비슷하게 physical zone으로 구성되어있으나, 각 physical zone이 1개 이상의 stripe groups으로 나뉘어있다는 점이 다름 (RAID 0 같은?)
    - zone I/F 표준을 따르기 위해 zone size는 2의 제곱승의 크기를 가짐
  - Zone ballooning
    - 다른 namespace의 사용량이 낮으면 stripe width를 빌려 vzone 확장
    - stripe group을 모두 기록하거나 FINISH/RESET 명령어가 요청되면 vzone 반환
  - Initial resource provisioning `다시 정리`
    - types of physical zone group
      - essential group : SSD 최대 write bandwidth 낼 수 있는 최소 active physical zone의 수
      - spare group : essential group을 제외한 나머지 active physical zone의 수
  - Local overdrive : zone expanding `다시 정리`
  - Global overdrive : namespace expanding `다시 정리`
    - 전체 drive의 write 수준에 따라 발동
  - reclaim : releasing overdriven vzone
    - 일반적인 성능 영향 최소화를 위해 reclaim은 2번의 global overdrive 기간 동안 아무런 write 동작이 없는(read-only state) namespace만 background로 수행
- Zone I/O Scheduler
  - read/write 동작에 대해 zone 간 contention을 최소화 하여 공평한 bandwidth 분배와 device 사용량을 최대화하는 방향으로 제어함
  - Congestion-avoid read scheduler
    - read latency를 기준으로 혼잡도를 판단하고 congestion window(cwnd) 관리
  - Cache-aware write admission control
    - global write latency 모니터링 및 token-based addmission control

## Evaluation
- Zone ballooning
- Zone I/O fairness
## 메모
- write latency : (4.5.2 마지막) physical zone마다 buffered write 하기때문에 write 직후 read latency는 낮을수도 있지 않나?
- Channel 마다 컨트롤러 할당 (read/write 처리)
- Way는 1개 컨트롤러 내에서 inter-leaving(pipelining)
- YCSB에서 operation 단위 (read, write,..?)

Summary
CNS vs. ZNS
  1. hardware 동일
  2. ZNS is a new interface in NVMe
  3. ftl역할 중 gc/l2p translation 역할 빠짐
  4. Write Constraints
     - Write strictly sequentially within a zone (Write Pointer)
     - Explicitly reset write pointer
  5. Goal of ZNS
     - Reduce WAF
     - Reduce OP
     - Reduce DRAM usage
     - Facilitate consumption of QLC
small vs. large zone zns
problem
  1. the zoned interface is static and inflexible
  2. workload over the RocksDB
Performance Characterization of a ZNS SSD
  1. Figure 4: Read bandwidth varying the stripe size for different types of zones.
     1. page align되는 16KB 성능이 가장 좋음
  2. Figure 5: Read/Write bandwidth varying the number of physical zones.
     1. physical zone이 많이 할당 될 수록 bandwidth 상승 (최대 PCIe 3.0 BW, 3.2GB/s)
  3. Figure 6: Read bandwidth under three channel overlapping (OL) allocations.
     1. channel에 zone이 집중되면 성능하락 발생
  4. Figure 7: Bandwidth and tail latency varying with the die overlapping ratio.
     1. namespace내에서 die가 집중되면, 성능하락 발생. 25%만 overlapped 성능하락이 큼.
eZNS
  1. Overview
     1. Figure 10-a: eZNS System Architecture.
  2. Zone ballloning
     1. Local overdrive : Zone expanding
     2. Global overdrive : Namespace expanding
     3. Reclaim : Zone/Namespace compaction
  3. I/O Scheduler
    1. Congestion control (aware read latency)
    2. Admission control (aware write latency)
Evaluation
