pkg install git python pkgconf bison gmake
git clone https://github.com/qemu/qemu
cd qemu
git submodule update --init dtc
mkdir build
cd build
../configure
gmake
gmake check
gmake install
