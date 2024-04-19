# Cachelib
* https://cachelib.org/
* Cachelib(OSDI '20) : https://www.usenix.org/conference/osdi20/presentation/berg
* I/O Passthru(FAST '24) : https://www.usenix.org/conference/fast24/presentation/joshi

## Cachelib
- Cachelib : Caching을 사용하는 다양한 subsystem이 너무 많아 관리가 어려운 문제가 있어, 이를 통합하여 다양한 서비스에 활용가능한 General-purpose Caching engine
- Lessons
    benchmark tool이나 학술 연구에서 활용된 캐싱모델은 Facebook과 같은 실제 상황을 대변하지 못함. 기존 캐싱 모델이 너무 낙관적이었음. 실제로는 DRAM 제약으로 Caching가능한 데이터가 적고, 그로인해 Hit ratio도 낮음.
- Caching Use cases
  1. CDN
     1. outside : byte miss rate를 줄여 wide-area network 전송량 감소
     2. within data center : object miss rate를 줄여 backend 및 storage query 감소
  2. Look-aside
     1. web app.은 넓은 범위의 caching이 필요하고 RPC를 통해 접근하므로 분산 캐시 서비스가 필요함. 그로인해 앱마다 캐싱 서비스를 관리하는 것은 매우 비효율 적임
  3. In-process
  4. ML model serveing system
  5. Storage backend
  6. Database page buffer
- Challenges
  1. temporal locality가 중요하고, 과거의 access pattern으로 caching policy 예측하는 것은 어려움.
  2. 다양한 object size 및 index overhead (8B~64B/64KB/128KB..)
  3. 높은 트래픽(꾸준히 높음 + 단기적으로 높음:스파이크)
  4. empty result를 반환하는 query가 많고, 이런 query는 backend database 컴퓨팅 비용이 높음
  5. cached data update


## I/O Passthru에서의 Cachelib
1. Cachelib
   1. BigHash : Small size 및 4KB random write 관리
   2. BlockCache : Large and Seq. workload 관리
2. BigHash와 BlockCache에 각각 Placement Identifier를 할당
3. 내장된 Cachebench tool의 write-only KVCache production workload 수행 (turn당 200분씩, 66시간)
   1. KV Cache production workload가 무엇이지?
      1. ssd_perf/flat_kvcache_reg
      2. ssd_perf/kvcache_l2_reg
      3. ssd_perf/kvcache_l2_wc
      4. trace_replay/kvcache
      5. hit_ratio/kvcache_reg
      6. kvcache/202206 # aws S3
      7. kvcache/202401 # aws S3

## Build
1. Build
    ```shell
    git clone https://github.com/facebook/CacheLib
    cd CacheLib
    ./contrib/build.sh -d -j -v
    ```
2. Re-build
    ```shell
    ./contrib/build.sh -d -v -j -O -S # 패키지 설치, git-pull skip
    ```
3. 실행파일 생성 경로
    ```shell
    CacheLib/opt/cachelib/bin/cachebench  
    ```

## Test
```shell
# block device name
DEV="nvme0n1"
# 실행파일 경로
CACHEBENCH="CacheLib/opt/cachelib/bin/cachebench" 
# config는 평가조건에 따라 설정 필요
FILE_CONFIG="CacheLib/opt/cachelib/test_configs/ssd_perf/graph_cache_leader/config.json" 
# 결과 로그파일 경로
FILE_OUTPUT="result.log"

# 0. enable nvme
# enable.sh 스크립트 사용, fdp or cns .. 

# 1. trim device
sudo fio --name=trim --filename=/dev/$DEV --rw=trim --bs=3G

# 2. execute benchmark
sudo $CACHEBENCH -json_test_config $FILE_CONFIG —progress_stats_file $FILE_OUTPUT
```

## config file



## cns re-produce

## fdp enable
"cache_config": {
  "deviceEnableFDP": true,
}
"navyConfig::ioEngine": "sync"
JSONSetVal(configJson, navyEnableIoUring)


## WAF 계산 어떻게?
```shell
The '<device>' may be either an NVMe character device (ex: /dev/nvme0) or an
nvme block device (ex: /dev/nvme0n1).E0417 14:09:34.857293 18798 NandWrites.cpp:116] Failed to run nvme command!
E0417 14:09:34.857343 18798 NandWrites.cpp:150] Unexpected number of fields in line! Got 0 fields, but expected at least 4.
E0417 14:09:34.857467 18798 Cache-inl.h:27] Exception fetching nand writes for nvme0n1. Msg: Failed to get bytes written for device nvme0n1
14:09:34     721.52M ops completed. Hit Ratio  45.79% (RAM  45.51%, NVM   0.51%)
```
double appWriteAmp = pctFn(numNvmBytesWritten, numNvmLogicalBytesWritten) / 100.0;
double devWriteAmp = pctFn(numNvmNandBytesWritten, numNvmBytesWritten) / 100.0; 

out << folly::sformat("NVM bytes written (physical)  : numNvmBytesWritten -> lookup("navy_device_bytes_written");
out << folly::sformat("NVM bytes written (logical)   : numNvmLogicalBytesWritten -> bhLogicalBytes + bcLogicalBytes
out << folly::sformat("NVM bytes written (nand)      : numNvmNandBytesWritten -> fetchNandWrites() - nandBytesBegin_
                                                                                                
double bhLogicalBytes = lookup("navy_bh_logical_written"); 
double bcLogicalBytes = lookup("navy_bc_logical_written"); 
numNvmLogicalBytesWritten = static_cast<size_t>(bhLogicalBytes + bcLogicalBytes);  

isitor("navy_device_bytes_written", getBytesWritten(),
       CounterVisitor::CounterType::RATE);            

       
uint64_t now = fetchNandWrites();                    
if (now > nandBytesBegin_) {                         
▏ ret.numNvmNandBytesWritten = now - nandBytesBegin_;
}

We do not bump logicalWrittenCount_ because logically a
remove operation does not write, but for BigHash, it does
incur physical writes.