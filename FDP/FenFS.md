# FenFS : ZenFS 기반으로 FDP enable

## mkfs tool

## file system
1. Open()
    zbd open()할 때 모든 존의 데이터와 갯수를 받아와서 zone 단위로 ONLINE Zone에 대한 정보(start offset, zone max. capa, write pointer)읽어와서
    vector<Zone *> io_zones에 모두 추가
    File metadata는 ZoneFile 구조체를 사용하여 device 정보(포인터)와 file_id, size, vector<extent>로 관리함

2. Mount()
    ZenFS 는 Filesystem metadata 관리를 위해 2개의 Metazone(header : crc 4B, size 4B)을 갖고있고, 
    Superblock에 최소한의 정보만 저장함. vector<Zone *> meta_zones에 추가
    이후 가장 seq_num이 높은 meta zone 1개만(실패 시 다음 meta zone 시도) Record 단위로 파일 정보를 읽어옴. (Record = zMetaHeader(crc 4B, size 4B) + Data(size만큼))
    meta_zone의 가장 처음 Record는 Superblock이고, 그 뒤로 각 log data가 담겨있음. 
    log data는 zMetaHeader(8B) + tag(4B) + length(가변길이, 최대 4B) + recovery data(length 길이 만큼, files) 로 구성되어있음.
    tag에 따라 
    1. kCompleteFilesSnapshot : 
        data는 file들이고, 또 [length(가변길이, 최대 4B) + tag(4B, file id) + file id(8B) + (file meta)반복 ] 반복으로 구성되어있고
        file meta또한 tag(4B) + kFileSize/kWriteLifeTimeHint/kExtent/kModificationTime/kLinkedFilename 등의 값의 쌍으로 저장되어있음(순서무관)
        그리고 kExtent는 data extent의 start lba와 length를 갖고있어, slba가 위치한 zone을 저장하고, 해당 zone의 used를 length만큼 감소시킴. 그리고 vector<extent *> extents_에 추가
        file data마다 zonefile을 생성하여 <string filename, zoneFile file>files_에 저장함
        `파일 수가 많으면 매우오래걸리려나, 아니면 rocksdb는 파일이 그렇게 많이 생기지 않으려나..mount는 자주일어나는건 아니니까?`
    2. kFileUpdate, kFileReplace, kFileDeletion : files_를 탐색하며 적절하게 처리
    이 과정을 log data Record의 tag가 kEndRecord (=4)일 때 까지 반복

그리고 File mapping 정보는 
## superblock
class Superblock {                              
    uint32_t magic_ = 0;
    char uuid_[37] = {0};               

    uint32_t sequence_ = 0;   
    uint32_t superblock_version_ = 0;        
    uint32_t flags_ = 0;                 
    uint32_t block_size_ = 0; /* in bytes */   
    uint32_t zone_size_ = 0;  /* in blocks */   
    uint32_t nr_zones_ = 0;                  
    char aux_fs_path_[256] = {0};         
    uint32_t finish_treshold_ = 0;    
    char zenfs_version_[64]{0};                   
    char reserved_[123] = {0};                                                                   
public:                                        
    const uint32_t MAGIC = 0x5a454e46; /* ZENF */ 
    const uint32_t ENCODED_SIZE = 512;            
    const uint32_t CURRENT_SUPERBLOCK_VERSION = 2;
    const uint32_t DEFAULT_FLAGS = 0;             
    const uint32_t FLAGS_ENABLE_GC = 1 << 0; 
}     
1. File Meta (1 sector = 4096 bytes)
   1. (20b) File ID : Max # of files = 1M (0xFFFFF = 1,048,575)
   2. (30b) Start LBA : 4TiB (1,073,741,824 sectors = 0x3FFF FFFF)
   3. (28b) Length : 1TiB (268,435,455 sectors = 0xFFF FFFF)
   4. (3b) Handle number (8ea)
   5. 
## format check
```shell
clang-format-11 -n -Werror --style=file fs/* util/zenfs.cc # Check for style issues
clang-format-11 -i --style=file fs/* util/zenfs.cc         # Auto-fix the style issues
```

## entry point gdb
- Factory 함수 호출 경로
    ./include/rocksdb/utilities/object_registry.h
    NewSharedObject() -> NewObject() -> FindFactory(target)

```c
Breakpoint 1, rocksdb::NewFlexFileSystem () at plugin/flexfs/fs/fs_flexfs.cc:51
51      NewFlexFileSystem() {
(gdb) n
53            new FlexFileSystem(FileSystem::Default()));
(gdb) bt
#0  rocksdb::NewFlexFileSystem () at plugin/flexfs/fs/fs_flexfs.cc:53
#1  0x00007ffff784a1e9 in rocksdb::flexfs_reg::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<std::unique_ptr> >*, std::allocator<char>*)#1}::operator()(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const, std::unique_ptr<rocksdb::FileSystem, std::default_delete<std::unique_ptr> >, std::unique_ptr<rocksdb::FileSystem, std::default_delete<std::unique_ptr> >*) const (
    __closure=0x7fffffffd300, f=0x7fffffffd3e0) at plugin/flexfs/fs/fs_flexfs.cc:24
#2  0x00007ffff784a7d1 in std::__invoke_impl<rocksdb::FileSystem*, rocksdb::flexfs_reg::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::allocator<char>*)#1}&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*>(std::__invoke_other, rocksdb::flexfs_reg::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::allocator<char>*)#1}&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >&&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*&&) (__f=...) at /usr/include/c++/11/bits/invoke.h:61
#3  0x00007ffff784a630 in std::__invoke_r<rocksdb::FileSystem*, rocksdb::flexfs_reg::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::allocator<char>*)#1}&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*>(rocksdb::FileSystem*&&, (rocksdb::flexfs_reg::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::allocator<char>*)#1}&)...) (__fn=...)
    at /usr/include/c++/11/bits/invoke.h:114
#4  0x00007ffff784a495 in std::_Function_handler<rocksdb::FileSystem* (std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*), rocksdb::flexfs_reg::{lambda(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*)#1}>::_M_invoke(std::_Any_data const&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*&&, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*&&) (__functor=..., __args#0="flexfs",
    __args#1=@0x7fffffffd258: 0x7fffffffd3e0, __args#2=@0x7fffffffd250: 0x7fffffffd320)
    at /usr/include/c++/11/bits/std_function.h:290
#5  0x00007ffff73d151e in std::function<rocksdb::FileSystem* (std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*)>::operator()(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::unique_ptr<rocksdb::FileSystem, std::default_delete<rocksdb::FileSystem> >*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >*) const (this=0x7fffffffd300,
    __args#0="flexfs", __args#1=0x7fffffffd3e0, __args#2=0x7fffffffd320)
    at /usr/include/c++/11/bits/std_function.h:590
#6  0x00007ffff73d0e59 in rocksdb::ObjectRegistry::NewObject<rocksdb::FileSystem> (
    this=0x7ffff4ec32b0, target="flexfs", object=0x7fffffffd3e8, guard=0x7fffffffd3e0)
    at ./include/rocksdb/utilities/object_registry.h:325
#7  0x00007ffff73d0a83 in rocksdb::ObjectRegistry::NewSharedObject<rocksdb::FileSystem> (
    this=0x7ffff4ec32b0, target="flexfs", result=0x7fffffffd6b0)
    at ./include/rocksdb/utilities/object_registry.h:372
#8  0x00007ffff73ccef3 in rocksdb::NewSharedObject<rocksdb::FileSystem> (config_options=...,
    id="flexfs", opt_map=std::unordered_map with 0 elements, result=0x7fffffffd6b0)
    at ./include/rocksdb/utilities/customizable_util.h:53
--Type <RET> for more, q to quit, c to continue without paging--
#9  0x00007ffff73cc55a in rocksdb::LoadSharedObject<rocksdb::FileSystem> (config_options=..., value="flexfs", result=0x7fffffffd6b0) at ./include/rocksdb/utilities/customizable_util.h:153
#10 0x00007ffff73cadf7 in rocksdb::FileSystem::CreateFromString (config_options=..., value="flexfs", result=0x7fffffffd6b0) at env/file_system.cc:94
#11 0x00007ffff739c584 in rocksdb::Env::CreateFromUri (config_options=..., env_uri="", fs_uri="flexfs", result=0x5555557241e0 <FLAGS_env>, guard=0x5555557241d0 <env_guard>) at env/env.cc:703
#12 0x000055555561ae90 in rocksdb::db_bench_tool (argc=1, argv=0x7fffffffe230) at tools/db_bench_tool.cc:8630
#13 0x0000555555619f7d in main (argc=4, argv=0x7fffffffe218) at tools/db_bench.cc:19
```