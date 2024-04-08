# eZNS scripts

0. Begin
   1. FTL의 Garbage collection overhead를 개선한 Zoned namespace SSD가 출시되었습니다. 그러나, 디바이스의 특성을 고려하지 않고 사용하다보니 제 성능을 내지 못하는 문제가 있습니다. 여러 실험을 통해 디바이스 특성을 파악하고, 이를 효율적으로 활용하는 기법을 제안한 논문입니다.
1. Problem
   1. RocksDB에서 fill-random workload를 수행하면서 사용된 Logical zone의 숫자를 나타낸 그래프입니다. Zone은 저장 공간의 단위로 뒤에 좀더 자세히 설명드리겠습니다. 여기서 빨간색 선으로 표기 된 최대 Active zone의 수는 SSD가 갖고 있는 자원입니다. HW spec상, 16개의 Active zone을 갖고 있음에도 불구하고 평균 6-7개 수준의 Zone만 사용하고 있습니다. 이 차이만큼 나머지 Active zone을 더 활용하여 성능을 높일 수 있다면 좋을텐데, 기존의 Zoned interface에서는 Zone 설정이 자유롭지 않기 때문에 이를 활용할 수 없습니다.
2. Zone
   1. 먼저, ZNS의 Zone에 대해서 설명드리겠습니다. Zoned interface에서 zone은 연속된 logical block address의 집합입니다. 중요한 특징으로 Write는 순차적으로만 가능하고, Zone단위로 지워집니다. 물론 Read에 대한 제약은 없습니다. 왼쪽 state diagram에서 데이터가 전혀 기록되지 않거나 Zone전체를 모두 사용하면 Inactive 상태인 Empty/Full state가 됩니다. 그 외에 데이터를 기록하기 시작하거나, 기록중인 Zone은 Active 상태인 Open/Closed state가 됩니다. Write를 하려면 Open state가 되어야 하고, 우측의 빨간 화살표인 Write pointer를 따라서 Write할 수 있습니다. Write pointer는 incremental 하게 증가하고 자연스럽게 순차쓰기가 보장됩닏나. 그리고 reset 시에는 해당 zone의 시작 LBA을 가리킵니다.
3. ZNS
   1. 다음은 ZNS는 어떤 특징이 있는지 설명드리겠습니다. ZNS는 Zoned-namespace SSD로 Namespace마다 Zone을 할당하여 사용하는 SSD입니다. 순차쓰기가 보장되고, Zone단위로 삭제 되기 때문에 Storage 내부에서 Invalid page를 처리하기 위한 Garbage collection이 발생하지 않습니다. FTL GC가 발생하지 않으면서 Host Write 요청만큼 Device Write 발생하여 WAF는 1로 보장되고, FTL GC를 위한 OP영역도 불필요해집니다. 또한, Out-place update를 위해 Page 단위로 관리되던 FTL Map table이 더 큰 단위인 Block 단위로 관리되기 때문에 Map table 크기도 대폭 줄어들고 Map table을 로딩하기 위한 DRAM 사용량도 줄어들게 됩니다. 이 모든 것들이 모두 제품의 비용이기 때문에 ZNS는 기존 SSD보다 저렴하다는 장점이 있습니다. 또한 스토리지 내부 GC 동작으로 인해 Host요청이 지연되는 문제가 없기 때문에 QoS 측면에서도 장점이 있습니다.
4. Zone
   1. 앞서 Zone을 간단히 설명드렸는데, 뒤에 내용을 이해하기 위해서 여러가지 Zone의 정의를 짚고 넘어가겠습니다. 크게 Physical / Logical zone으로 구분됩니다. Physical zone은 1개의 Die내에서 block단위로 구성되어있는 가장 작은 단위의 zone을 의미합니다. Logical zone은 여러개의 Physical zone이 Stripe하게 구성된 zone을 의미합니다. 이런 Logical zone은 구성에 따라 Small zone과 Large zone으로 나눌 수 있습니다.
5. Zone
   1. Small zone은 Single die내에 있는 physical zone으로 구성되고, Large zone은 여러 die에 걸쳐 stripe하게 구성되는 것을 의미합니다. small/large zone의 구성은 HW단계에서 고정되기 때문에 서두에 말씀드렸던 것처럼 Zone interface는 유연하지 못하다는 단점이 있습니다. 예를들어 삼성의 ZNS는 Small zone SSD이고, Western Digital의 ZNS는 Large zone SSD 입니다.
6. Observation
   1. 저자는 Small zone ZNS SSD를 바탕으로 Zone 구성에 대한 특성을 알아보기 위해 몇가지 실험을 하였습니다.
7. Zone striping
   1. 먼저 Zone striping입니다. 데이터를 기록하는 최소 단위는 stripe size로 결정되고, 몇개의 physical zone에 걸쳐있을 것인지는 stripe width로 결정 됩니다.
   2. Stripe size와 width를 가변시키면서 평가하한 결과입니다. X 축은 stripe 설정, Y 축은 Bandwidth를 나타냅니다. 전체적으로 Stripe size가 클수록, Width가 클수록 Read Bandwidth가 높아집니다. 특히 Stripe size가 16KB보다 작을 때는 Read 성능이 급격하게 저하 되며, 이는 device의 page size와 같은 크기입니다.
8. Zone placement
   1. 다음은 Zone의 배치를 바꿔가며 Read 성능을 측정하였습니다. 총 16개의 Zone을, 각 채널 당 2개/4개/8개 씩 Zone을 배치하여 채널에 중복된 Zone의 수를 다르게 평가하였습니다. 컬러는 Chunk size를 구분한 것이고, chunk size 조건과 무관하게 Channel에 중복된 zone이 적을 수록 높은 성능을 보이고 있습니다.
   2.  다음은 Channel이 아닌, Die 수준에서 zone을 중복 할당하며 평가하였습니다. Physical zone이 공유하는 die가 없으면 0%, 모든 physical zone이 같은 Die를 공유하면 100% 입니다. 25%만 겹쳐도 성능이 저하되는 이유는 overlapped 된 zone의 앞선 동작으로 인해, 뒤에 요청된 zone의 동작도 함께 지연되기 때문입니다.
9. Zone interference
   1.  Die 1개의 bandwidth가 낮기 때문에 Die내에서 여러 physical zone들의 충돌이 발생하면, logical zone의 성능도 떨어지게 됨. zone간에 독립적인 성능을 보장하지 못하다는 것은 namespace마다 독립적인 성능을 보장하지 못한다는 것이고, 이는 각 tenant 동작에 따라 서로의 i/o 성능에 영향을 줄 수 있다는 것입니다.
10. eZNS Overview
   1.  eZNS는 Logical zone을 추상화하는 software layer 입니다. workload에 따라 동적으로 zone의 구성을 변경하여 최적의 성능을 이끌어낼 수 있습니다.
   2.  eZNS는 크게 2가지 요소로 구성되어있습니다. Zone I/O Scheduler는 Zone과 Die interference를 최소화 하기 위한 것으로 Read Latency가 높아지면 stripe width를 줄여 zone congestion을 감소시키고, write latency가 높아지면 token 생성을 줄여서 write 빈도를 감소시키는 방향으로 조절합니다. (write latency가 높아지는 것은 die의 write cache가 모두 소모된 것을 의미) 그리고 Device resource인 active zone의 사용률을 최대화 하기 위해 동적으로 zone configuration을 변경하는 Zone arbiter가 있습니다.
11. Zone ballooning
   1.  zone arbiter를 통해 v-zone의 크기가 커지고 작아지는 것을 zone ballooning이라고 합니다. 특정 v-zone에 I/O가 집중되면 Local/Global overdrive를 통해 v-zone이 커지고, Idle상태일 때 Reclaim과정을 통해 다시 줄어듭니다.
   2.  Zone ballooning 과정을 그림으로 설명드리겠습니다. 먼저 App 3개가 있고, 여기서 App은 namespace와 같은 범주입니다. 각 App 내에 4개의 logical zone으로 구성된 경우입니다. 파란색은 write 가 발생중인 zone이고, 주황색은 Idle 상태입니다. static zone은 각 zone이 구성이 바뀌지 않기 떄문에 그림과 같은 상태에서 전체 24개 Active zone 중에서 실질적으로 6개만 사용하게 됩니다.
   3.  Local drive를 하게 된다면, App 내에서 Idle 상태의 logical zone으로부터 active zone을 넘겨 받아 Write 발생 중인 logical zone에 더 많은 active zone이 할당됩니다. 이때 각 logical zone은 언제 요청될지 모르는 i/o를 처리하기 위해 최소 1개의 active zone을 갖고있어야 합니다.
   4.  나아가 Global drive를 하게 된다면, Idle 상태인 App으로 부터 active zone을 넘겨 받아 write 발생중인 logical zone을 더욱 확장시킵니다. 그리고 idle 상태의 active zone은 namespace마다 공평하게 나눠갖게 됩니다.
12. Evaluation
   1.  다음은 실험 입니다. 저자는 PCIe Gen3 장치를 사용하였고, namespace마다 essential 및 spare zone을 각각 32개씩 할당하고, stripe width는 2, size는 32KB로 설정하였고 이런 namespace를 4개 구성하였습니다. 대조군으로 사용할 static ZNS 조건은 stripe width 4, stripe size 16KB로 설정하였으며 logical zone이 골고루 활용되었을 때 device의 성능을 최대로 끌어낼 수 있는 조건입니다.
13. Zone ballooning
   1. namespace내에서 Logical zone의 수를 증가시키면서, local overdrive에 따른 bandwidth를 비교한 실험입니다. 4개의 logical zone에 write하는 경우에도 overdrive를 통해 vzone을 확장하면서 width를 8까지 증가시키며 개일 때와 준하는 수준으로 성능을 향상시킬 수 있었습니다.
      1. `다시 보기` v-zone도 physical zone이 16이라는게 무슨말이지, 8/16 writers일 때에도 physical zone의 수가 더이상 늘지 않는 이유는 뭐지?
   2. Global overdrive의 효과를 보여주기 위해 3개의 namespace는 각각 2개의 writer를 동작시키고, 하나의 namespace에는 8개의 writer를 동작시켜 i/o를 집중시켰습니다. 100초간의 실험 중에서 30초 부터 80초 구간동안은 2개의 writer가 동작하는 namespace 3개를 중단했습니다. 그러자 해당 구간에서 8개의 writer가 동작하는 namesapce의 bandwidth가 더욱 증가하는 것을 볼 수 있습니다. 그리고 이 때 사용된 Spare zone도 증가하는 것을 볼 수 있습니다. 이처럼 Global overdrive를 통해 vzone이 확장되어 bandwidth를 높일 수 있습니다.
14. RocksDB multi-tenant
   1. write-heavy workload를 수행하는 namespace 2개와 read only workload를 수행하는 namesapce 2개를 동시에 수행하며 i/o scheduler의 효과를 확인하였습니다. read 동작의 tail latency가 20~70% 감소되었으며, throughput도 17% 가량 상승했습니다. 또한 write latency및 throughput도 소폭 개선되었는데, 이는 read-only namespace에 있던 spare zone이 write-heavy namespace로 전달되었기 때문입니다. (global drive)
15. RocksDB with YCSB
    1.  YCSB workloads에서 평가한 결과입니다. A, B, C, F Workload를 크게 write most workload와 read intensive workload로 구분할 수 있습니다. Global overdrive를 통해 write most namespace의 bandwidth를 향상시키고, I/O scheduler를 통해 read latency가 개선 되었습니다.
16. Conclusion


Memo
Local overdrive : Zone을 다 쓰고나서 새로운 Zone을 open할 때, 이전에 opened된 v-zones수의 히스토리를 바탕으로 spare zone이 남아있는 만큼 공평하게 나누어 추가 할당
Global overdrive : Write 가 집중되는 것을 모니터링하고 (I/O scheduler) 이에 따라 조절