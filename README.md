# Description
android kernel sm8250 Suitable for the Xiaomi Pad 5 Pro series

===================
## How to build
1. Clone repository
```shell
git clone --depth=1 -b 4.19.197 https://github.com/HuaJI66/Xiaomi_Kernel_OpenSource.git

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
apt-get update && apt install -y sudo git cpio curl llvm lld wget vim git ccache automake flex lzop bison gperf build-essential zip zlib1g-dev g++-multilib libxml2-utils bzip2 libbz2-dev libbz2-1.0 libghc-bzlib-dev squashfs-tools pngcrush schedtool dpkg-dev liblz4-tool make optipng maven libssl-dev pwgen libswitch-perl policycoreutils minicom libxml-sax-base-perl libxml-simple-perl bc libc6-dev-i386 x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev xsltproc unzip device-tree-compiler kmod python3 python3-pip
cd ${KERNEL_DIR:=$(pwd)}

# Export
export LINKER="ld.lld"
export ARCH=arm64
export SUBARCH=arm64
export PROCS=$(nproc --all)
export DISTRO=$(source /etc/os-release && echo "${NAME}")
export KBUILD_COMPILER_STRING=$("${KERNEL_DIR}"/gcc64/bin/aarch64-elf-gcc --version | head -n 1)
PATH="${KERNEL_DIR}"/gcc32/bin:"${KERNEL_DIR}"/gcc64/bin:/usr/bin/:${PATH}
VERBOSE=0
DEFCONFIG="vendor/kona-perf_defconfig vendor/xiaomi/sm8250-common.config vendor/xiaomi/${DEVICE}.config"
OUT_DIR=${OUT_DIR:=$(pwd)}

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

   Run build.sh in docker
```shell
docker run --name kb \
-v /root/.cache/ccache/:/root/.cache/ccache/ \
-v $(pwd):/root/source \
-e "DEVICE=enuma" \
-e "KERNEL_DIR=/root/source" \
-e "TOOLCHAIN=eva-gcc" \
-e "KBUILD_BUILD_HOST=docker_builder" \
-e "KBUILD_BUILD_USER=$(whoami)" \
ubuntu bash -c  "/root/source/build.sh"
```

3. Check output
   
   You can get the output file in directory arch/arm64/boot after compiled.
