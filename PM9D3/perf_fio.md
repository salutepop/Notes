# 기본 성능 측정 (CNS Mode)
WRITE: bw=1462MiB/s (1533MB/s)
READ: bw=1948MiB/s (2043MB/s)


```c
<WRITE>
cm@jyha-test:~/f2fs_perf$ sudo fio /home/cm/f2fs_perf/sw.fio --filename=/home/cm/cns/file_30G --size=30G
file1: (g=0): rw=write, bs=(R) 128KiB-128KiB, (W) 128KiB-128KiB, (T) 128KiB-128KiB, ioengine=io_uring, iodepth=128
fio-3.37-1-g4eef
Starting 1 process
file1: Laying out IO file (1 file / 30720MiB)
Jobs: 1 (f=1): [W(1)][100.0%][w=1310MiB/s][w=10.5k IOPS][eta 00m:00s]
file1: (groupid=0, jobs=1): err= 0: pid=8396: Fri Apr  5 04:12:43 2024
  write: IOPS=11.7k, BW=1462MiB/s (1533MB/s)(30.0GiB/21007msec); 0 zone resets
    slat (nsec): min=1517, max=46389k, avg=3500.86, stdev=93571.98
    clat (usec): min=466, max=74860, avg=10936.81, stdev=5334.46
     lat (usec): min=473, max=74863, avg=10940.31, stdev=5334.95
    clat percentiles (usec):
     |  1.00th=[ 7898],  5.00th=[ 7963], 10.00th=[ 8029], 20.00th=[ 8455],
     | 30.00th=[ 8717], 40.00th=[ 8848], 50.00th=[ 8979], 60.00th=[ 9110],
     | 70.00th=[ 9241], 80.00th=[10552], 90.00th=[19268], 95.00th=[19530],
     | 99.00th=[23200], 99.50th=[47449], 99.90th=[60031], 99.95th=[64750],
     | 99.99th=[72877]
   bw (  MiB/s): min=  993, max= 1999, per=100.00%, avg=1472.16, stdev=314.83, samples=41
   iops        : min= 7948, max=15992, avg=11777.32, stdev=2518.63, samples=41
  lat (usec)   : 500=0.01%, 750=0.05%, 1000=0.03%
  lat (msec)   : 2=0.09%, 4=0.19%, 10=77.13%, 20=20.12%, 50=1.93%
  lat (msec)   : 100=0.45%
  cpu          : usr=3.64%, sys=3.15%, ctx=243682, majf=0, minf=9
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwts: total=0,245760,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
  WRITE: bw=1462MiB/s (1533MB/s), 1462MiB/s-1462MiB/s (1533MB/s-1533MB/s), io=30.0GiB (32.2GB), run=21007-21007msec

Disk stats (read/write):
  nvme0n1: ios=0/92007, sectors=0/46783080, merge=0/0, ticks=0/910710, in_queue=910709, util=38.83%
```
```c
<READ>
cm@jyha-test:~/f2fs_perf$ sudo fio /home/cm/f2fs_perf/sr.fio
file1: (g=0): rw=read, bs=(R) 128KiB-128KiB, (W) 128KiB-128KiB, (T) 128KiB-128KiB, ioengine=io_uring, iodepth=32
fio-3.37-1-g4eef
Starting 1 process
file1: Laying out IO file (1 file / 30720MiB)
Jobs: 1 (f=1): [R(1)][100.0%][r=2289MiB/s][r=18.3k IOPS][eta 00m:00s]
file1: (groupid=0, jobs=1): err= 0: pid=8371: Fri Apr  5 04:10:23 2024
  read: IOPS=15.6k, BW=1948MiB/s (2043MB/s)(38.0GiB/20001msec)
    slat (nsec): min=1090, max=1481.4k, avg=55032.41, stdev=69677.22
    clat (nsec): min=609, max=2801.4M, avg=1988560.64, stdev=27915244.59
     lat (usec): min=173, max=2801.4k, avg=2043.59, stdev=27915.08
    clat percentiles (usec):
     |  1.00th=[ 1565],  5.00th=[ 1582], 10.00th=[ 1582], 20.00th=[ 1598],
     | 30.00th=[ 1713], 40.00th=[ 1729], 50.00th=[ 1745], 60.00th=[ 1745],
     | 70.00th=[ 1762], 80.00th=[ 1762], 90.00th=[ 1778], 95.00th=[ 1778],
     | 99.00th=[ 1811], 99.50th=[ 1811], 99.90th=[ 1827], 99.95th=[ 1876],
     | 99.99th=[ 7439]
   bw (  MiB/s): min=  265, max= 2293, per=100.00%, avg=2163.84, stdev=428.49, samples=35
   iops        : min= 2124, max=18350, avg=17310.74, stdev=3427.92, samples=35
  lat (nsec)   : 750=0.01%
  lat (usec)   : 250=0.01%, 500=0.01%, 750=0.01%, 1000=0.01%
  lat (msec)   : 2=99.98%, 4=0.01%, 10=0.01%, >=2000=0.01%
  cpu          : usr=0.29%, sys=99.50%, ctx=337, majf=0, minf=1035
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=100.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.1%, 64=0.0%, >=64=0.0%
     issued rwts: total=311699,0,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=32

Run status group 0 (all jobs):
   READ: bw=1948MiB/s (2043MB/s), 1948MiB/s-1948MiB/s (2043MB/s-2043MB/s), io=38.0GiB (40.9GB), run=20001-20001msec

Disk stats (read/write):
  nvme0n1: ios=154690/4, sectors=79201280/448, merge=0/0, ticks=17211/0, in_queue=17212, util=85.61%
```