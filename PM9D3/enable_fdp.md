# Enabling FDP
## 설치 파일
1. nvme-cli
   ```shell
   git clone https://github.com/linux-nvme/nvme-cli.git
   cd nvme-cli
   meson setup --force-fallback-for=libnvme,json-c .build
   meson compile -C .build
   meson install -C .build
   ```
2. fio
   ```shell
   git clone https://github.com/axboe/fio.git
   ```


## CNS 설정
1. 기존 namespace 삭제
   ```shell
   $ sudo nvme delete-ns /dev/nvme0n1
   delete-ns: Success, deleted nsid:1
   ```
2. namespace 생성
   1. size of ns (NSZE) == capacity of ns (NCAP) = 0x36b9d996 (PM9D3 초기 값)
   ```shell
   $ sudo nvme create-ns /dev/nvme0 -s 0x36b9d996 -c 0x36b9d996 -f 0
   create-ns: Success, created nsid:1
   ```
3. namespace 연결
   ```shell
   $ sudo nvme attach-ns /dev/nvme0 -n 1 -c 7
   attach-ns: Success, nsid:1
   ```

## FDP 설정
1. FDPS(FDP Support) bit '1' 확인
   1. ID Controller[99:96]의 CTRATT [19]
   2. 1000 0000 0010 1001 0000  (19bit 1로 set 상태)
   ```shell
   $ sudo nvme id-ctrl /dev/nvme0 | grep -i ctratt
   ctratt    : 0x80290
   ```
2. FDP Enable
   1. Enable
   ```shell
   $ sudo nvme set-feaeture /dev/nvme0 -f 0x1d -c 1 -s
   set-feature:0x1d (Flexible Direct Placement), value:00000000, cdw12:0x00000001, save:0x1
   ```
   2. Disable
   ```shell
   $ sudo nvme set-feature /dev/nvme0 -f 0x1d -c 0 -s
   ```
   3. Check
   ```shell
   $ sudo nvme get-feature /dev/nvme0 -f 0x1d -H
   get-feature:0x1d (Flexible Direct Placement), Current value:0x00000001
   ```
3. 기존 namespace 삭제
   ```shell
   $ sudo nvme delete-ns /dev/nvme0 -n 1
   delete-ns: Success, deleted nsid:1
   $ sudo nvme delete-ns /dev/nvme0 -n 2
   delete-ns: Success, deleted nsid:2
   ```
4. namespace 생성
   ```shell
   $ sudo nvme create-ns /dev/nvme0 -s 1024000 -c 1024000 -f 0
   create-ns: Success, created nsid:1
   $ sudo nvme create-ns /dev/nvme0 -s 917100000 -c 917100000 -f 0 -e 1 -n 7 -p 1,2,3,4,5,6,7
   create-ns: Success, created nsid:2
   ```
5. namespace 연결
   ```shell
   $ sudo nvme attach-ns /dev/nvme0 -n 1 -c 7
   attach-ns: Success, nsid:1
   $ sudo nvme attach-ns /dev/nvme0 -n 2 -c 7
   attach-ns: Success, nsid:2
   ```

## 기타 정보 확인
1. nvme list 확인
   ```shell
   sudo nvme list
   Node                  Generic               SN                   Model                                    Namespace  Usage                      Format           FW Rev
--------------------- --------------------- -------------------- ---------------------------------------- ---------- -------------------------- ---------------- --------
/dev/nvme0n1          /dev/ng0n1            PHKS015300QX375AGN   INTEL SSDPED1K375GA                      0x1        375.08  GB / 375.08  GB      4 KiB +  0 B   E2010485
/dev/nvme1n1          /dev/ng1n1            S77UNG0TB00664       MZOL63T8HDLT-00AFB                       0x1          4.19  GB /   4.19  GB      4 KiB +  0 B   LDA64F2Q
/dev/nvme1n2          /dev/ng1n2            S77UNG0TB00664       MZOL63T8HDLT-00AFB                       0x2          3.76  TB /   3.76  TB      4 KiB +  0 B   LDA64F2Q
   ```
2. namespace id
   ```shell
   $ sudo nvme id-ns /dev/nvme0n1
   NVME Identify Namespace 1:
      nsze    : 0x36b9d996
      ncap    : 0x36b9d996
      nuse    : 0
      nsfeat  : 0x1a
      nlbaf   : 1
      flbas   : 0
      mc      : 0x2
      dpc     : 0x14
      dps     : 0
      nmic    : 0
      rescap  : 0
      fpi     : 0x80
      dlfeat  : 9
      nawun   : 63
      nawupf  : 0
      nacwu   : 0
      nabsn   : 63
      nabo    : 0
      nabspf  : 0
      noiob   : 0
      nvmcap  : 3760740458496
      npwg    : 31
      npwa    : 0
      npdg    : 31
      npda    : 0
      nows    : 31
      mssrl   : 0
      mcl     : 0
      msrc    : 0
      nulbaf  : 0
      anagrpid: 0
      nsattr	: 0
      nvmsetid: 0
      endgid  : 1
      nguid   : 3737553054b006640025384700000001
      eui64   : 0000000000000000
      lbaf  0 : ms:0   lbads:12 rp:0 (in use)
      lbaf  1 : ms:64  lbads:12 rp:0
   ```
3. FDP config
   ```shell
   $ sudo nvme fdp configs /dev/nvme0 -e 1
   FDP Attributes: 0x80
   Vendor Specific Size: 144
   Number of Reclaim Groups: 1
   Number of Reclaim Unit Handles: 8
   Number of Namespaces Supported: 2
   Reclaim Unit Nominal Size: 13079937024
   Estimated Reclaim Unit Time Limit: 0
   Reclaim Unit Handle List:
   [0]: Initially Isolated
   [1]: Initially Isolated
   [2]: Initially Isolated
   [3]: Initially Isolated
   [4]: Initially Isolated
   [5]: Initially Isolated
   [6]: Initially Isolated
   [7]: Initially Isolated
   ```
4. RU 남은 용량 확인
   ```shell
   $ sudo nvme fdp stats /dev/nvme1n2

   Placement Identifier 0; Reclaim Unit Handle Identifier 1
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344

   Placement Identifier 1; Reclaim Unit Handle Identifier 2
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344

   Placement Identifier 2; Reclaim Unit Handle Identifier 3
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344

   Placement Identifier 3; Reclaim Unit Handle Identifier 4
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344

   Placement Identifier 4; Reclaim Unit Handle Identifier 5
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344

   Placement Identifier 5; Reclaim Unit Handle Identifier 6
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344

   Placement Identifier 6; Reclaim Unit Handle Identifier 7
   Estimated Active Reclaim Unit Time Remaining (EARUTR): 0
   Reclaim Unit Available Media Writes (RUAMW): 3193344
   ```