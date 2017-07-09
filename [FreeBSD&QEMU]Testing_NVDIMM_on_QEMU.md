# Test NVDIMM functionality on FreeBSD with QEMU

# Introduction

We are actively iterating on adding NVDIMM support to FreeBSD. To test the program without actual NVDIMM hardware, we use the newest version of QEMU which has support of virtual NVDIMM and is not yet available in ports.

# Steps

## 1. Install build dependencies

```
pkg install git python pkgconf bison gmake
```

## 2. Clone source code and update dtc module

```
git clone https://github.com/qemu/qemu
cd qemu
git submodule update --init dtc
```

## 3. Build and install

```
mkdir build
cd build
../configure
gmake
gmake check
gmake install
```

The source code (including build) folder can then be deleted after the installation.

## 4. Prepare an image

## 5. Start QEMU and test the functionality

```
qemu-system-x86_64 -hda freebsd.img -boot d -machine pc,nvdimm -m 32G,maxmem=100G,slots=10 -object memory-backend-ram,id=mem2,size=10G -device nvdimm,memdev=mem2,id=nv2,label-size=128k
```
