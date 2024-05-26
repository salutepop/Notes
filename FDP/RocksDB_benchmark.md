# RocksDB benchmark
ReadRandom() : readrandom, readmissing, readrandomsmall, 
DoWrite() : fillseq, fillbatch, fillrandom, filluniquerandom, overwrite, fillsync ...

COMPRESSION_TYPE="none"          
USE_O_DIRECT=1  # --use_direct_io_for_flush_and_compaction --use_direct_reads           
                                 
DB_DIR=$1 # $2 ("/home/cm/tmp/") 
shift                            
NUM_KEYS=$1 # $3 (90000000)      
shift                            
CACHE_SIZE=$1 # $4 (6442450944)  
shift                            
DURATION=$1 # $5 (600)           
shift         
OUTPUT_DIR=$1 # $6                   


 fillseq_disable_wal : sequential write
 bulkload(fillrandom) : random write
 readrandom : random read


1. mkfs & mount
    ```shell
    umount /dev/nvme1n1
    sudo /home/cm/fdp/util/nvmeconfig.sh /dev/nvme1 cns
    sudo mkfs.xfs /dev/nvme1n1
    rm -r /home/cm/tmp
    mkdir /home/cm/tmp
    sudo mount /dev/nvme1n1 /home/cm/tmp
    sudo chown cm:cm /home/cm/tmp
    ```
2. run test
    ```shell
    /home/cm/fdp/dbbench/benchmark.sh bulkload | tee /home/cm/fdp/dbbench/result/bulkload/test.log
    ````


## memo
compaction log 중 `file write time` 항목이 있음. ms 단위
rocksdb i/o 구현 참고 : https://github.com/PingCAP-Hackthon2019-Team17/rocksdb/pull/1

## TODO
1. readwhilewrite (RW MIX) 상황에서 Latency 비교
2. Cachelib 재현
3. RocksDB workload별 i/o chunk size비교 (Compaction까지 볼수 있으면 좋을 듯, bulkload 를 분리해서 진행?)