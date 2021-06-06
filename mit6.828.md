## 坑
### 1. 安装 gmp-5.0.2.tar.bz2 到 make 的时候出现 make: *** No targets specified and no makefile found.  Stop
解决方案：
sudo apt-get install m4
然后重新 ./configure --prefix=$PFX


### 2. git clone --recursive https://github.com/riscv/riscv-gnu-toolchain 下载不下来

```
git clone  https://gitee.com/mirrors/riscv-gnu-toolchain
cd riscv-gnu-toolchain
#把之前的空文件夹删了
rm -rf riscv-*
#再分别拉取分支
git clone -b riscv-newlib-3.2.0 git@gitee.com:mirrors/riscv-newlib.git
git clone -b riscv-glibc-2.29 git@gitee.com:mirrors/riscv-glibc.git
git clone -b riscv-gcc-9.2.0 git@gitee.com:mirrors/riscv-gcc.git
git clone git@gitee.com:mirrors/riscv-dejagnu.git

#riscv-binutils与riscv-gdb为同一个仓库的不同分支
git clone -b riscv-binutils-2.34 git@gitee.com:mirrors/riscv-binutils-gdb.git  riscv-binutils
git clone -b fsf-gdb-9.1-with-sim git@gitee.com:mirrors/riscv-binutils-gdb.git riscv-gdb
```
