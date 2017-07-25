Some dependencies:
```
pkg install git autoconf cmake gmake pkgconf python3
```

Install tcllauncher:
```
pkg install tcllauncher
```

Build dump1090 dependency: [bladeRF](https://github.com/Nuand/bladeRF)
You can also see the [Detailed Guide from Official Wiki](https://github.com/Nuand/bladeRF/wiki/Getting-Started%3A-Linux#Building_bladeRF_libraries_and_tools_from_source)
```
pw add group bladerf
pw groupmod bladerf -m root
```
```
groups
```
```
git clone https://github.com/Nuand/bladeRF
cd bladeRF
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -DINSTALL_UDEV_RULES=ON ../ -DBLADERF_GROUP=bladerf
make
make install
ldconfig
```

Install dump1090 dependency: rtlsdr
```
pkg install rtlsdr
```

Build faup1090:
```
git clone https://github.com/yzgyyang/dump1090
cd dump1090
gmake faup1090
cp faup1090 /usr/lib/piaware/helpers/faup1090
```

Build mlat-client:
```
git clone https://github.com/mutability/mlat-client
cd mlat-client
python3 setup.py install
cp /usr/local/bin/fa-mlat-client /usr/lib/piaware/helpers/fa-mlat-client
```
