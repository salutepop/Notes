# xNVME
https://www.youtube.com/watch?v=Y7A3dPpdjNs

## Build
1. repo clone 및 pkg 설치
```shell
git clone https://github.com/OpenMPDK/xNVMe.git xnvme
cd xnvme
chmod 775 ./toolbox/pkgs/ubuntu-jammy.sh
sudo ./toolbox/pkgs/ubuntu-jammy.sh
```
2. install
```shell
# configure xNVMe and build dependencies (fio, libvfn, and SPDK/NVMe)
meson setup builddir
cd builddir

# build xNVMe
meson compile

# install xNVMe
sudo meson install

# uninstall xNVMe
# meson --internal uninstall
```