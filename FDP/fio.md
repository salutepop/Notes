nvme_setup_cmd : dsmgmt의 65535 (0xffff) 단위로 placement id 설정 됨
plid 0 = 0
plid 1 = 65536 (0x10000)
plid 2 = 131072 (0x20000)
...

offset 1% 마다 slba 9171000 (4GB) 만큼 건너뜀

             fio-98976   [014] ..... 879504.023670: nvme_setup_cmd: nvme1: qid=0, cmdid=28690, nsid=0, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=1, ctrlid=0)
          <idle>-0       [007] d.h1. 879504.023904: nvme_sq: nvme1: qid=0, head=16, tail=16
          <idle>-0       [007] ..s1. 879504.023923: nvme_complete_rq: nvme1: qid=0, cmdid=28690, res=0x0, retries=0, flags=0x2, status=0x0
             fio-98976   [014] ..... 879504.023933: nvme_setup_cmd: nvme1: qid=0, cmdid=28691, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=0, ctrlid=0)
          <idle>-0       [007] d.h1. 879504.024056: nvme_sq: nvme1: qid=0, head=17, tail=17
          <idle>-0       [007] ..s1. 879504.024058: nvme_complete_rq: nvme1: qid=0, cmdid=28691, res=0x0, retries=0, flags=0x2, status=0x0
             fio-98976   [014] ..... 879504.024068: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=15, cmdid=28930, nsid=1, flags=0x0, meta=0x0, cmd=(0x12 cdw10=01 00 00 00 03 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00)          <idle>-0       [014] d.h1. 879504.024284: nvme_sq: nvme1: disk=nvme1n1, qid=15, head=704, tail=704
          <idle>-0       [014] d.h1. 879504.024284: nvme_complete_rq: nvme1: disk=nvme1n1, qid=15, cmdid=28930, res=0x0, retries=0, flags=0x2, status=0x0
             fio-98998   [016] ..... 879504.125407: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=512, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171000, len=0, ctrl=0x20, dsmgmt=65536, reftag=0)
          <idle>-0       [016] d.h1. 879504.125452: nvme_sq: nvme1: disk=nvme1n1, qid=17, head=676, tail=676
          <idle>-0       [016] d.h1. 879504.125454: nvme_complete_rq: nvme1: disk=nvme1n1, qid=17, cmdid=512, res=0x0, retries=0, flags=0x2, status=0x0
           fwupd-99191   [014] ..... 885899.948583: nvme_setup_cmd: nvme1: qid=0, cmdid=32784, nsid=0, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=1, ctrlid=0)
          <idle>-0       [007] d.h1. 885899.948794: nvme_sq: nvme1: qid=0, head=18, tail=18
          <idle>-0       [007] .Ns1. 885899.948801: nvme_complete_rq: nvme1: qid=0, cmdid=32784, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99653   [010] ..... 895027.137550: nvme_setup_cmd: nvme1: qid=0, cmdid=36884, nsid=0, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=1, ctrlid=0)
          <idle>-0       [007] d.h1. 895027.137818: nvme_sq: nvme1: qid=0, head=19, tail=19
          <idle>-0       [007] ..s1. 895027.137840: nvme_complete_rq: nvme1: qid=0, cmdid=36884, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99653   [010] ..... 895027.137851: nvme_setup_cmd: nvme1: qid=0, cmdid=36885, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=0, ctrlid=0)
          <idle>-0       [007] d.h1. 895027.137984: nvme_sq: nvme1: qid=0, head=20, tail=20          <idle>-0       [007] ..s1. 895027.137985: nvme_complete_rq: nvme1: qid=0, cmdid=36885, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99653   [010] ..... 895027.138001: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=11, cmdid=45953, nsid=1, flags=0x0, meta=0x0, cmd=(0x12 cdw10=01 00 00 00 03 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00)
          <idle>-0       [010] d.h1. 895027.138213: nvme_sq: nvme1: disk=nvme1n1, qid=11, head=83, tail=83
          <idle>-0       [010] d.h1. 895027.138213: nvme_complete_rq: nvme1: disk=nvme1n1, qid=11, cmdid=45953, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99675   [016] ..... 895027.238978: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=4608, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171000, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
          <idle>-0       [016] d.h1. 895027.238994: nvme_sq: nvme1: disk=nvme1n1, qid=17, head=677, tail=677
          <idle>-0       [016] d.h1. 895027.238994: nvme_complete_rq: nvme1: disk=nvme1n1, qid=17, cmdid=4608, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99720   [012] ..... 895070.227453: nvme_setup_cmd: nvme1: qid=0, cmdid=53273, nsid=0, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=1, ctrlid=0)
          <idle>-0       [007] d.h1. 895070.227632: nvme_sq: nvme1: qid=0, head=21, tail=21
          <idle>-0       [007] ..s1. 895070.227644: nvme_complete_rq: nvme1: qid=0, cmdid=53273, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99720   [012] ..... 895070.227649: nvme_setup_cmd: nvme1: qid=0, cmdid=53274, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=0, ctrlid=0)
          <idle>-0       [007] d.h1. 895070.227771: nvme_sq: nvme1: qid=0, head=22, tail=22
          <idle>-0       [007] ..s1. 895070.227773: nvme_complete_rq: nvme1: qid=0, cmdid=53274, res=0x0, retries=0, flags=0x2, status=0x0
             fio-99720   [012] ..... 895070.227809: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=13, cmdid=16897, nsid=1, flags=0x0, meta=0x0, cmd=(0x12 cdw10=01 00 00 00 03 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00)
          <idle>-0       [012] d.h1. 895070.228034: nvme_sq: nvme1: disk=nvme1n1, qid=13, head=747, tail=747
          <idle>-0       [012] d.h1. 895070.228035: nvme_complete_rq: nvme1: disk=nvme1n1, qid=13, cmdid=16897, res=0x0, retries=0, flags=0x2, status=0x0
           <...>-99742   [017] ..... 895070.329214: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=18, cmdid=57536, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171000, len=0, ctrl=0x20, dsmgmt=196608, reftag=0)
           <...>-99742   [017] d.h1. 895070.329239: nvme_sq: nvme1: disk=nvme1n1, qid=18, head=304, tail=304
           <...>-99742   [017] d.h1. 895070.329242: nvme_complete_rq: nvme1: disk=nvme1n1, qid=18, cmdid=57536, res=0x0, retries=0, flags=0x2, status=0x0


write : fio

```
[global]                            
filename=/dev/ng1n1                 
ioengine=io_uring_cmd               
cmd_type=nvme                       
iodepth=32                          
size=4K                             
bs=4K                               
fdp=1                               
                                    
[write-a]                           
rw=write                            
fdp_pli=2                           
buffer_pattern=0xaa / 0x55          
offset=0%    // 0%, 1%, 2% 로 4GB offset설정                                                           
```

read : nvme cli
cdw10 = slba
```
sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=9171000 --cdw11=0 --cdw12=0 -o normal
```



fio-101472  [017] ..... 897457.024452: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=18, cmdid=24774, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171000, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101471  [016] ..... 897457.024452: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=29188, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=0, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101473  [018] ..... 897457.024455: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=19, cmdid=21443, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=18342000, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)


16k
```
fio-101678  [008] ..... 897628.309053: nvme_setup_cmd: nvme1: qid=0, cmdid=8205, nsid=0, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=1, ctrlid=0)
<idle>-0       [007] d.h1. 897628.309203: nvme_sq: nvme1: qid=0, head=11, tail=11
<idle>-0       [007] ..s1. 897628.309205: nvme_complete_rq: nvme1: qid=0, cmdid=8205, res=0x0, retries=0, flags=0x2, status=0x0
fio-101678  [008] ..... 897628.309207: nvme_setup_cmd: nvme1: qid=0, cmdid=8206, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_admin_identify cns=0, ctrlid=0)
<idle>-0       [007] d.h1. 897628.309325: nvme_sq: nvme1: qid=0, head=12, tail=12
<idle>-0       [007] ..s1. 897628.309326: nvme_complete_rq: nvme1: qid=0, cmdid=8206, res=0x0, retries=0, flags=0x2, status=0x0
fio-101678  [008] ..... 897628.309331: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=9, cmdid=61844, nsid=1, flags=0x0, meta=0x0, cmd=(0x12 cdw10=01 00 00 00 03 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00)
<idle>-0       [008] d.h1. 897628.309541: nvme_sq: nvme1: disk=nvme1n1, qid=9, head=424, tail=424
<idle>-0       [008] d.h1. 897628.309541: nvme_complete_rq: nvme1: disk=nvme1n1, qid=9, cmdid=61844, res=0x0, retries=0, flags=0x2, status=0x0
<...>-101701  [019] ..... 897628.411044: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=20, cmdid=29317, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171000, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101700  [018] ..... 897628.411044: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=19, cmdid=25539, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=0, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101702  [016] ..... 897628.411044: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=41476, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=18342000, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101702  [016] d.h.. 897628.411062: nvme_sq: nvme1: disk=nvme1n1, qid=17, head=762, tail=762
fio-101700  [018] d.h1. 897628.411064: nvme_sq: nvme1: disk=nvme1n1, qid=19, head=190, tail=190
fio-101702  [016] d.h.. 897628.411065: nvme_complete_rq: nvme1: disk=nvme1n1, qid=17, cmdid=41476, res=0x0, retries=0, flags=0x2, status=0x0
fio-101700  [018] d.h1. 897628.411066: nvme_complete_rq: nvme1: disk=nvme1n1, qid=19, cmdid=25539, res=0x0, retries=0, flags=0x2, status=0x0
fio-101701  [019] d.h1. 897628.411070: nvme_sq: nvme1: disk=nvme1n1, qid=20, head=0, tail=0
fio-101701  [019] d.h1. 897628.411072: nvme_complete_rq: nvme1: disk=nvme1n1, qid=20, cmdid=29317, res=0x0, retries=0, flags=0x2, status=0x0
fio-101700  [018] ..... 897628.411092: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=19, cmdid=29635, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=1, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101702  [016] ..... 897628.411097: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=45572, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=18342001, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101700  [018] d.h.. 897628.411103: nvme_sq: nvme1: disk=nvme1n1, qid=19, head=191, tail=191
fio-101701  [019] ..... 897628.411103: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=20, cmdid=33413, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171001, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101700  [018] d.h.. 897628.411105: nvme_complete_rq: nvme1: disk=nvme1n1, qid=19, cmdid=29635, res=0x0, retries=0, flags=0x2, status=0x0
fio-101700  [018] ..... 897628.411107: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=19, cmdid=5060, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=2, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101702  [016] d.h.. 897628.411108: nvme_sq: nvme1: disk=nvme1n1, qid=17, head=763, tail=763
fio-101702  [016] d.h.. 897628.411109: nvme_complete_rq: nvme1: disk=nvme1n1, qid=17, cmdid=45572, res=0x0, retries=0, flags=0x2, status=0x0
fio-101702  [016] ..... 897628.411113: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=37381, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=18342002, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101700  [018] ..... 897628.411114: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=19, cmdid=33731, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=3, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101701  [019] d.h.. 897628.411115: nvme_sq: nvme1: disk=nvme1n1, qid=20, head=1, tail=1
fio-101701  [019] d.h.. 897628.411117: nvme_complete_rq: nvme1: disk=nvme1n1, qid=20, cmdid=33413, res=0x0, retries=0, flags=0x2, status=0x0
fio-101700  [018] d.h.. 897628.411118: nvme_sq: nvme1: disk=nvme1n1, qid=19, head=192, tail=193
fio-101700  [018] d.h.. 897628.411119: nvme_complete_rq: nvme1: disk=nvme1n1, qid=19, cmdid=5060, res=0x0, retries=0, flags=0x2, status=0x0
fio-101701  [019] ..... 897628.411120: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=20, cmdid=33414, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171002, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101702  [016] ..... 897628.411121: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=17, cmdid=49668, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=18342003, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101702  [016] d.h.. 897628.411123: nvme_sq: nvme1: disk=nvme1n1, qid=17, head=764, tail=765
fio-101702  [016] d.h.. 897628.411124: nvme_complete_rq: nvme1: disk=nvme1n1, qid=17, cmdid=37381, res=0x0, retries=0, flags=0x2, status=0x0
fio-101700  [018] d.h.. 897628.411125: nvme_sq: nvme1: disk=nvme1n1, qid=19, head=193, tail=193
fio-101700  [018] d.h.. 897628.411125: nvme_complete_rq: nvme1: disk=nvme1n1, qid=19, cmdid=33731, res=0x0, retries=0, flags=0x2, status=0x0
fio-101701  [019] ..... 897628.411129: nvme_setup_cmd: nvme1: disk=nvme1n1, qid=20, cmdid=37509, nsid=1, flags=0x0, meta=0x0, cmd=(nvme_cmd_write slba=9171003, len=0, ctrl=0x20, dsmgmt=131072, reftag=0)
fio-101701  [019] d.h.. 897628.411131: nvme_sq: nvme1: disk=nvme1n1, qid=20, head=2, tail=2
fio-101701  [019] d.h.. 897628.411132: nvme_complete_rq: nvme1: disk=nvme1n1, qid=20, cmdid=33414, res=0x0, retries=0, flags=0x2, status=0x0
<idle>-0       [016] d.h1. 897628.411149: nvme_sq: nvme1: disk=nvme1n1, qid=17, head=765, tail=765
<idle>-0       [016] d.h1. 897628.411151: nvme_complete_rq: nvme1: disk=nvme1n1, qid=17, cmdid=49668, res=0x0, retries=0, flags=0x2, status=0x0
<idle>-0       [019] d.h1. 897628.411152: nvme_sq: nvme1: disk=nvme1n1, qid=20, head=3, tail=3
<idle>-0       [019] d.h1. 897628.411153: nvme_complete_rq: nvme1: disk=nvme1n1, qid=20, cmdid=37509, res=0x0, retries=0, flags=0x2, status=0x0
```

read
```
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=0 --cdw11=0 --cdw12=0 -o normal
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=1 --cdw11=0 --cdw12=0 -o normal
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f0000: aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=2 --cdw11=0 --cdw12=0 -o normal                                                             
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=3 --cdw11=0 --cdw12=0 -o normal                                                             
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa aa "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=4 --cdw11=0 --cdw12=0 -o normal                                                             
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=9171000 --cdw11=0 --cdw12=0 -o normal                                                       
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 "UUUUUUUUUUUUUUUU"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=9171001 --cdw11=0 --cdw12=0 -o normal                                                       
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 "UUUUUUUUUUUUUUUU"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=9171002 --cdw11=0 --cdw12=0 -o normal                                                       
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 "UUUUUUUUUUUUUUUU"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=9171003 --cdw11=0 --cdw12=0 -o normal                                                       
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 55 "UUUUUUUUUUUUUUUU"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=9171004 --cdw11=0 --cdw12=0 -o normal                                                       
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=18342000 --cdw11=0 --cdw12=0 -o normal                                                      
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=18342001 --cdw11=0 --cdw12=0 -o normal                                                      
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=18342002 --cdw11=0 --cdw12=0 -o normal                                                      
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=18342003 --cdw11=0 --cdw12=0 -o normal                                                      
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff ff "................"
cm@cm:~/repo/liburing/test$ sudo nvme io-passthru /dev/nvme1n1 --opcode=2 --namespace-id=1 --data-len=16 --read --cdw10=18342004 --cdw11=0 --cdw12=0 -o normal                                                      
IO Command Read is Success and result: 0x00000000
       0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
0000: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 "................"
```