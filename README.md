# Description

Android sm8250 kernel suitable for the Xiaomi Pad 5 Pro series.

## How to build
1. Clone repository
```shell
git clone --depth=1 -b lineage-21 https://github.com/HuaJI66/Xiaomi_Kernel_OpenSource.git source
cd source

# Clone GCC
git clone https://github.com/mvaisakh/gcc-arm64 --depth=1 gcc64
git clone https://github.com/mvaisakh/gcc-arm --depth=1 gcc32

```

2. Build

   Add build.sh
```shell
vi build.sh
```

```shell
cd ${KERNEL_DIR:=$(pwd)}

# Export Variable
export LINKER="ld.lld"
export ARCH=arm64
export SUBARCH=arm64
export PROCS=$(nproc --all)
export DISTRO=$(source /etc/os-release && echo "${NAME}")
export KBUILD_COMPILER_STRING=$("${KERNEL_DIR}"/gcc64/bin/aarch64-elf-gcc --version | head -n 1)
PATH="${KERNEL_DIR}"/gcc32/bin:"${KERNEL_DIR}"/gcc64/bin:/usr/bin/:${PATH}
VERBOSE=0
DEFCONFIG="vendor/kona-perf_defconfig vendor/xiaomi/sm8250-common.config vendor/xiaomi/${DEVICE}.config"
OUT_DIR=${OUT_DIR:=out}

make O=${OUT_DIR} ARCH=arm64 ${DEFCONFIG}
if [ "$METHOD" = "lto" ]; then
	scripts/config --file ${OUT_DIR}/.config \
	-e CONFIG_LTO_GCC
fi
make -kj$(nproc --all) O=${OUT_DIR} \
ARCH=arm64 \
CC="ccache aarch64-elf-gcc" \
LD="${KERNEL_DIR}/gcc64/bin/aarch64-elf-ld.lld" \
AR=llvm-ar \
NM=llvm-nm \
OBJCOPY=llvm-objcopy \
OBJDUMP=llvm-objdump \
OBJCOPY=llvm-objcopy \
OBJSIZE=llvm-size \
STRIP=llvm-strip \
CROSS_COMPILE=aarch64-elf- \
CROSS_COMPILE_COMPAT=arm-eabi- \
CC_COMPAT=arm-eabi-gcc \
V=$VERBOSE 2>&1 | tee error.log
```
   
   Prepare docker container
```shell
docker run --name kb \
-v /root/.cache/ccache/:/root/.cache/ccache/ \
-v $(pwd):/root/source \
-e "DEVICE=enuma" \
-e "KERNEL_DIR=/root/source" \
-e "TOOLCHAIN=eva-gcc" \
-e "KBUILD_BUILD_HOST=docker_builder" \
-e "KBUILD_BUILD_USER=$(whoami)" \
ubuntu bash -c  "tail -F /dev/null"
```

```shell
chmod +x build.sh
```
   Reopen a new terminal window

```shell
# Enter container
docker exec -it kb bash
```

```shell
# Install dependencies
apt-get update && apt install -y sudo git cpio curl llvm lld wget vim git ccache automake flex lzop bison gperf build-essential zip zlib1g-dev g++-multilib libxml2-utils bzip2 libbz2-dev libbz2-1.0 libghc-bzlib-dev squashfs-tools pngcrush schedtool dpkg-dev liblz4-tool make optipng maven libssl-dev pwgen libswitch-perl policycoreutils minicom libxml-sax-base-perl libxml-simple-perl bc libc6-dev-i386 x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev xsltproc unzip device-tree-compiler kmod python3 python3-pip
## python2
wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz
sudo tar xzf Python-2.7.18.tgz
cd Python-2.7.18
sudo ./configure --enable-optimizations
sudo make altinstall
sudo ln -sfn '/usr/local/bin/python2.7' '/usr/bin/python2'
```
```shell
# Run build.sh
 cd /root/source && bash build.sh
```

3. Check output
   
You can get the output file in directory /root/source/out/arch/arm64/boot after compiled.
