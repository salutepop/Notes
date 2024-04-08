# I/O Passthru
* paper : https://www.usenix.org/conference/fast24/presentation/joshi
* NVMe passthru (2021) : https://lpc.events/event/11/contributions/989/attachments/747/1723/lpc-2021-building-a-fast-passthru.pdf

## NVMe innovations vs Kernel abstractions
- NVMe 발전에 따른 문제
  - user interface 필요에 의해 NVMe interface 계속 증가 (zone-append 등)
  - 기존 write API를 포함하여 data placement 정보를 전달할 방법이 마땅하지 않음
  - 새로운 HW를 위한 오픈소스 활동에도 불구하고 mainline수준에서 지원하는 user interface 없음
  - IOCTL을 통해 user interface의 부족을 극복해볼 수 있지만, IOCTL은 동기식으로 parallel NVMe를 효과적으로 사용하지 못함

## I/O advances with io_uring
- io_uring : latest asynchronous I/O subsystme
  - user space와 kernel 경계에서 동작
  - user space와 kernel 공유하는 ring buffer 쌍을 통해 통신 (SQ, CQ)
  - Baching : submit multiple I/O requests in single system call
  - SQPoll : syscall-free submissions
  - IOPoll : interrupt-free I/O (completion)
  - Chaining : 여러 명령간에 순서/종속성을 설정하고 entier chain의 명령을 한번의 syscall로 제출

## Limitations of existing NVMe passthrough
- NVME_IOCTL 통해 NVMe passthrough 지원
  - block device와 너무 밀접함
  - blocking interface로 인해 scalability와 efficiency 낮음
  - user space와 kernel space간의 copy overhead
  - root user만 사용가능함

## Design goals
- Block I/O independence : non-block NVMe command set 처리 가능 
- Catch-all user interface : 새로운 NVMe 명령어마다 syscall을 추가하는 문제 해결
- Efficient and scalable
- General accessibility : root user외에도 사용가능
- Upstream acceptance : mainline upstream

## I/O Passthru in Kernel
- io_uring과 새로운 char-interface `/dev/ng0n1`을 통해 NVMe command 전달
- Linux AIO를 선택하지 않은 이유
  - io_uring이 더 효율적이고 기능이 많음
  - upstream 활동이 더 활발함

## Availability : NVMe generic char interface
- NVMe driver에서 char device node를 생성할 수 있도록 수정
- char device는 미래의 새로운 command set도 생성할 수 있음
- operations
  - .open
  - .release
  - .unlocked_ioctl
  - compat_ioctl
  - uring_cmd
  - uring_cmd_iopoll

## Efficiency & Scalability
- Big SQE : double size of SQE, 16 bytes -> 80 bytes
- Big CQE : double size of CEQ, extra 16 bytes
- Command provider `다시 이해`

## Asynchronous processing
- Vectored variants : userspace로 부터 multiple data를 buffer로 전달
- Zero copy : user-space의 Big SQE가 자체적으로 control structure를 생성하기 때문에 copy_form_user 동작 불필요, 결과도 Big CQE를 통해 전달되므로 put_user 동작 불필요
- Zero memory-allocations : io_uring_cmd내에 32 bytes pud array를 활용하여 submit data중 completion까지 유지되어야 하는 정보 저장
- Fixed-buffer : Data 전송 중 buffer pinned/unpinned 비용을 줄이기 위해, buffer를 재사용하는 경우 buffer를 lock하지 않고, 이전 locked region을 그대로 사용하는 기능
- Completion polling : io_uring의 N polled Queue-pair (SQ-CQ)를 통해 interrupt가 아닌 polling `uring_cmd_iopoll` 방식으로 completion처리하여 context swtiching overhead 감소

## Accessibility : from root-only to general
- permission checks
  - (일반)
    - VFS, file open request 시점
    - NVMe driver, file open command issue 시점
  - (passthrough)
    - `CAP_SYS_ADMIN`이 있으면 모든 것 허용, 그렇지 않으면 아래의 command type 검사
    - file-mode write 권한 있으면 write I/O command 가능
    - (아니면?) read I/O command 가능
    - namesapce/controller 식별을 위한 Admin command만 가능

## Block layer
- NVMe passthrough는 block layer bypass를 의미하는 것이 아니라, device 위에 아무런 layer가 없는 것을 의미함
- Block I/O와 Passthrough I/O 비교
  - Abstract device limit : block I/O는 64KB 이상의 read 요청도 명령을 64KB 단위로 나누어서 처리해주지만, Passthrough에서는 device limit에 제한 됨(추상화 되지 않음)
  - I/O scheduler : NVMe에서는 I/O를 merge할 수 있는 I/O Scheduler가 오히려 방해되어 기존에도 default `none`으로 사용하고 있음. 이를 bypass 하는 Passthrough가 유리함.
  - Multi queue : 기존에는 Blk-mq를 통해 MQ를 지원하고, Passthrough도 이와 유사한 구조를 통해 MQ 지원
  - Tag management : block layer는 outstanding command를 관리하기 위해 tag를 사용하고, driver는 신경쓰지 않아도 되었음. Passthrough도 driver에서 신경쓰지 않아도 됨(?)
  - Command-timeout & Abort : block layer는 outstanding command를 abort할 수 있고, Passthrough도 `control structure`의 timeout value를 사용하여 이를 지원함

## I/O Passthru support
- Kernel 6.2 이상 모든 기능 지원
- xNVME에 io_uring_cmd 지원하도록 구현
- SPDK에 새로운 xNVMe bdev추가. 단일 xNVMe bdev는 AIO, io_uring, io_uring_cmd를 전환할 수 있음.
- nvme-cli : character interface를 지원하도록 수정 (block interface동작도 동일하게 지원)
- fio : io_uring_cmd engine 추가. `cmd_type` 지정해야하며, NVMe passthrough는 `nvme`로 지정
- Liburing : io_uring app애 대해서 big-SQE,CQE 지원 추가

## Flexible data placement(FDP)
- 특징
  - 기존 GC단위와 유사한 개념의 RU(Reclaim unit) 추가
  - write command에 PID함께 전달, PID(placement identifier)에 따라 특정 RU에 기록
  - 서로 다른 PID로 쓰여진 데이터는 LBA가 섞이지 않음
- block layer의 write-hint기능이 제거되었고, FDP와 같이 placement hint가 필요한 경우 I/O Passthru가 그 역할을 대신 할 수 있음.

## Computational storage
- Computational sotrage를 위해 새로운 NVMe namespace 표준화 됨
- character device를 통해 kernel 변경 없이 SLM 및 Compute namespace를 지원할 수 있음 `다시 이해`
  - Memory namespace : SSD Data를 byte-addressable 하게 사용하기 위한 subsystem-local-memory(SLM). 그리고 SLM과 데이터를 복사하기 위한 namespace
  - Compute namespace : SLM에 있는 데이터에서 실행되는 다양한 프로그램


## End-to-End data protection
- metadata buffer와 length를 함께 전달하기 때문에 buffer alignment check 불필요
- fio io_uring_cmd를 통해 DIF, DIX를 지원
  - DIF : meta data가 data buffer와 함께 전달
  - DIX : meta data가 data buffer와 분리 전달

## Efficiency characterization
- Peak performance
  - `character device`만 적용되어도 `block device`보다 빠른 이유는?
  - (condition) 512b/random read/QD 128/batch 32
  - Fixed-buffer vs. Polling
    - Fixed-buffer는 mapping overhead 개선
    - Polling은 context switching overhead 개선
    - Fixed-buffer + Polling 조합의 성능 향상이 더 큼
  - io_uring passthrough vs. io_uring block
    - passthrough으 경우 split/merge/scheduling을 하지 않기 때문에 io_uring handler Execution 시간 짧음(209ns -> 144ns)
- Scalability(QD)
  - 높은 QD가 될 수록 passthrough의 처리량이 높아짐
- Cpu utilization and submission-latency
  - logical record size가 커질 수록 I/O buffer로 사용되는 physical page들을 더 많이 lock, DMA mapping해야하기 때문에 수행시간이 더 오래걸림. 그래서 4KB record일 때 cpu사용률이 가장 높음. 그리고 Fixed-buffer는 submission rate와 CPU cost를 감소시킴
- SQPoll and batching
  - SQPoll과 batching은 모두 syscall cost를 줄이지만, SQPoll은 I/O submit하는 app. thread와의 충돌을 피하기 위해 별도의 core가 필요함

## Cachelib with FDP
- SSD data handling을 위한 2개의 I/O engines
  - BigHash: small size data
  - BlockCache : large item, issue seq. flash-friendly
- 다양한 placement identifier를 구분하기 위해 Cachelib에서 I/O Passthru I/F를 사용할 수 있게 수정하였고, FDP 명령어를 통해 SSD에 hint를 보내 mixed data를 방지
- 기존 SSD의 WAF는 2를 상회하였으나, hint를 통해 1에 가까운 WAF 달성.

## Comparison against SPDK
- multiple core에서는 io_uring_char와 SPDK 둘다 최대 성능(10MIOPS)을 보여주지만, single core에서는 kernel config 최적화를 하더라도 SPDK 성능을 따라잡지는 못함. 그래도 기존 io_uring_block 보다는 높은 성능을 보여줌
- SPDK 를 못따라 잡는 이유
  - SPDK는 단일 사용자로 NVMe device의 독점적인 권한이 있어, multiple device/user를 위한 sync, sharing등의 추가적인 코드가 없음
  - I/O Passthru는 HW queue abstraction, tag manage 같은 block layer의 기능들을 갖고 있다보니 추가적인 처리비용 발생
  - preemption scheduling, timer freq. 등의 kernel config에 의한 성능 overhead가 있음