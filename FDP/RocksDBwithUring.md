# RocksDB with Uring(for FDP)
 - Goal : RocksDB에서 raw device NVMe(/w FDP) 사용
 - How To
   - cachelib에서 FDP 사용하는 방법 파악 (아마 liburing.h 기반)
   - RocksDB에서 ZNS를 사용하기 위한 파일시스템(ZenFS)에서 raw device 접근/제어 방법 파악
   - RocksDB에서 FDP를 사용하고, 이를 관리하는 파일시스템 구현


## RocksDB Filesystem Background
1. RocksDB는 데이터를 파일로 관리하며, 이를 위해 파일시스템이 존재함
2. raw device는 mount되어 mount point(directory)로 접근됨
3. file_system.h에 filesystem wrapper class가 있고, 이를 상속받아 별도의 file system을 구현할 수 있음 (ZenFS 참고)

## Cachelib에서 FDP 적용한 과정
Study 예정

## RocksDB의 ZenFS에서 raw device 접근/제어 과정
1. RocksDB Filesystem Wrapper
   1. /home/cm/repo/rocksdb/include/rocksdb/file_system.h
2. ZenFS
   1. (raw) zbdlib_zenfs/zonefs_zenfs -> zbd_zenfs -> fs_zenfs (filesystem) 순으로 추상화됨
   2. zbd_zenfs에서 ZonedBlockDevice() 초기화 시, ZBD_Backend를 선택함 (zbdlib or zonefs)
   3. zbdlib에서 filename으로 Open()하고, Read(), Write()로 data access함. Read/Write 내부에서 zbdlib의 pread, pwrite를 호출함
      ```cpp
      pread(direct ? read_direct_f_ : read_f_, buf, size, pos)
      pwrite(write_f_, data, size, pos)
      ```
---
- zbd_zenfs.cc : ZonedBlockDevice::ZonedBlockDevice() 과정에서 backend type에 따라 zbd_be_ 결정.
   ZbdBackendType::kBlockDev or ZbdBackendType::kZoneFS
- zonefs_zenfs.h|59| <<global>> class ZoneFsBackend : public ZonedBlockDeviceBackend
- zbdlib_zenfs.h|20| <<global>> class ZbdlibBackend : public ZonedBlockDeviceBackend

```cpp
// zbd_zenfs.h
class ZonedBlockDeviceBackend {
 public:
  uint32_t block_sz_ = 0;
  uint64_t zone_sz_ = 0;
  uint32_t nr_zones_ = 0;
 
 public:
  virtual IOStatus Open(bool readonly, bool exclusive,
                        unsigned int *max_active_zones,
                        unsigned int *max_open_zones) = 0;
  
  virtual std::unique_ptr<ZoneList> ListZones() = 0;
  virtual IOStatus Reset(uint64_t start, bool *offline,
                         uint64_t *max_capacity) = 0;
  virtual IOStatus Finish(uint64_t start) = 0;
  virtual IOStatus Close(uint64_t start) = 0;
  virtual int Read(char *buf, int size, uint64_t pos, bool direct) = 0;
  virtual int Write(char *data, uint32_t size, uint64_t pos) = 0;
  virtual int InvalidateCache(uint64_t pos, uint64_t size) = 0;
  virtual bool ZoneIsSwr(std::unique_ptr<ZoneList> &zones,
                         unsigned int idx) = 0;
  virtual bool ZoneIsOffline(std::unique_ptr<ZoneList> &zones,
                             unsigned int idx) = 0;
  virtual bool ZoneIsWritable(std::unique_ptr<ZoneList> &zones,
                              unsigned int idx) = 0;
  virtual bool ZoneIsActive(std::unique_ptr<ZoneList> &zones,
                            unsigned int idx) = 0;
  virtual bool ZoneIsOpen(std::unique_ptr<ZoneList> &zones,
                          unsigned int idx) = 0;
  virtual uint64_t ZoneStart(std::unique_ptr<ZoneList> &zones,
                             unsigned int idx) = 0;
  virtual uint64_t ZoneMaxCapacity(std::unique_ptr<ZoneList> &zones,
                                   unsigned int idx) = 0;
  virtual uint64_t ZoneWp(std::unique_ptr<ZoneList> &zones,
                          unsigned int idx) = 0;
  virtual std::string GetFilename() = 0;
  uint32_t GetBlockSize() { return block_sz_; };
  uint64_t GetZoneSize() { return zone_sz_; };
  uint32_t GetNrZones() { return nr_zones_; };
  virtual ~ZonedBlockDeviceBackend(){};
};

```

ZenFS Open command (zbdlib 사용, 향후 liburing 변경)
```cpp
// zbdlib_zenfs.cc
IOStatus ZbdlibBackend::Open(bool readonly, bool exclusive,
                             unsigned int *max_active_zones,
                             unsigned int *max_open_zones) {
  zbd_info info;
 
  /* The non-direct file descriptor acts as an exclusive-use semaphore */
  if (exclusive) {
    read_f_ = zbd_open(filename_.c_str(), O_RDONLY | O_EXCL, &info);
  } else {
    read_f_ = zbd_open(filename_.c_str(), O_RDONLY, &info);
  }
 
  if (read_f_ < 0) {
    return IOStatus::InvalidArgument(
        "Failed to open zoned block device for read: " + ErrorToString(errno));
  }
 
  read_direct_f_ = zbd_open(filename_.c_str(), O_RDONLY | O_DIRECT, &info);
  if (read_direct_f_ < 0) {
    return IOStatus::InvalidArgument(
        "Failed to open zoned block device for direct read: " +
        ErrorToString(errno));
  }
 
```

ZenFS Read/Write command (zbdlib 사용, 향후 liburing 변경)
```cpp
// zbdlib_zenfs.cc

int ZbdlibBackend::Read(char *buf, int size, uint64_t pos, bool direct) {
  return pread(direct ? read_direct_f_ : read_f_, buf, size, pos);
}

int ZbdlibBackend::Write(char *data, uint32_t size, uint64_t pos) {
  return pwrite(write_f_, data, size, pos);
}
```

## liburing 사용방법
1. Read : 아래 링크 참고하여 sq 얻고, read command 전송, read buffer 취득하는 과정 확인 및 cachelib에서 read하는 것과 비교
   1. https://unixism.net/loti/tutorial/cat_liburing.html
2. Write : 아래 링크 참고하여 read -> write 하는 과정 이해하고, cachelib에서 write하는 것과 비교
   1. https://unixism.net/loti/tutorial/cp_liburing.html