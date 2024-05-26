


## my microbench
### Command Subimit / Completed 통합
<256KB read>
uring - QD 1
[LOG] uring_test.cpp(60) : Final offset= 26214400000
[LOG] uring_test.cpp(61) : 100K times(sec)= 7.21978
[LOG] uring_test.cpp(62) : IOPS= 13850.8

uring - QD 16
[LOG] uring_test.cpp(60) : Final offset= 26214400000
[LOG] uring_test.cpp(61) : 100K times(sec)= 7.23393
[LOG] uring_test.cpp(62) : IOPS= 13823.7

uring_cmd - QD 1
[LOG] uring_test.cpp(60) : Final offset= 26214400000
[LOG] uring_test.cpp(61) : 100K times(sec)= 10.0723
[LOG] uring_test.cpp(62) : IOPS= 9928.17

uring_cmd - QD 16
[LOG] uring_test.cpp(60) : Final offset= 26214400000
[LOG] uring_test.cpp(61) : 100K times(sec)= 10.0075
[LOG] uring_test.cpp(62) : IOPS= 9992.48

### Command Submit / Completed 분리

[LOG] uring_test.cpp(90) : QD= 16
[LOG] uring_test.cpp(91) : BS= 4096
[LOG] uring_test.cpp(92) : isPassthru= 1
[LOG] uring_test.cpp(93) : isRead= 1
[LOG] uring_test.cpp(94) : PlacementID= 1
[LOG] uring_test.cpp(95) : Final offset= 4096000000
[LOG] uring_test.cpp(96) : 1M times(sec)= 1.58205
[LOG] uring_test.cpp(97) : IOPS= 632093

[LOG] uring_test.cpp(90) : QD= 16
[LOG] uring_test.cpp(91) : BS= 4096
[LOG] uring_test.cpp(92) : isPassthru= 1
[LOG] uring_test.cpp(93) : isRead= 0
[LOG] uring_test.cpp(94) : PlacementID= 1
[LOG] uring_test.cpp(95) : Final offset= 4096000000
[LOG] uring_test.cpp(96) : 1M times(sec)= 1.58562
[LOG] uring_test.cpp(97) : IOPS= 630669

[LOG] uring_test.cpp(90) : QD= 1
[LOG] uring_test.cpp(91) : BS= 4096
[LOG] uring_test.cpp(92) : isPassthru= 1
[LOG] uring_test.cpp(93) : isRead= 1
[LOG] uring_test.cpp(94) : PlacementID= 1
[LOG] uring_test.cpp(95) : Final offset= 4096000000
[LOG] uring_test.cpp(96) : 1M times(sec)= 1.59275
[LOG] uring_test.cpp(97) : IOPS= 627846

[LOG] uring_test.cpp(90) : QD= 1
[LOG] uring_test.cpp(91) : BS= 4096
[LOG] uring_test.cpp(92) : isPassthru= 1
[LOG] uring_test.cpp(93) : isRead= 0
[LOG] uring_test.cpp(94) : PlacementID= 1
[LOG] uring_test.cpp(95) : Final offset= 4096000000
[LOG] uring_test.cpp(96) : 1M times(sec)= 1.58901
[LOG] uring_test.cpp(97) : IOPS= 629323

### nvme fdp status /dev/nvme0n1
- PID 1로 위 테스트 진행 중 확인
Placement Identifier 0; Reclaim Unit Handle Identifier 1
  Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
  Reclaim Unit Available Media Writes (RUAMW): 3193344

Placement Identifier 1; Reclaim Unit Handle Identifier 2
  Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
  Reclaim Unit Available Media Writes (RUAMW): 1197504      <<

Placement Identifier 2; Reclaim Unit Handle Identifier 3
  Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
  Reclaim Unit Available Media Writes (RUAMW): 3193344