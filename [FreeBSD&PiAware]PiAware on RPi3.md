# Running PiAware on FreeBSD on a Raspberry Pi

## dump1090

There's no real kernel code required for it - all of the rtl-sdr code just uses the generic USB userland API which is shared between many operating systems.

```
pkg install rtl-sdr
```

Then, using it to test ADSB is pretty easy:

```
rtl_adsb -V -S
```

You will see tons of short packets received from nearby aircrafts.

With dump1090, these packets can be decoded and displayed nicely on a graphical interface:
```
pkg install dump1090
chmod a+x /usr/local/bin/dump1090
dump1090 --net --aggressive
```

Point a webserver at http://localhost:8080/ and watch!

## PiAware

Some dependencies:
```
pkg install git autoconf cmake gmake pkgconf python3 openssl
```

Install tcllauncher and tcl libraries:
```
pkg install tcllauncher tclsh86 itcl tcllib tcltls
# chmod a+x /usr/local/bin/tcllauncher
# chmod a+x /usr/local/bin/tclsh8.6
```

Install piaware:
```
git clone https://github.com/flightaware/piaware
cd piaware
gmake install
```

Build dump1090 dependency: [bladeRF](https://github.com/Nuand/bladeRF)  
You can also see the [Detailed Guide from Official Wiki](https://github.com/Nuand/bladeRF/wiki/Getting-Started%3A-Linux#Building_bladeRF_libraries_and_tools_from_source)
```
pw add group bladerf
pw groupmod bladerf -m root
```
Check if root is in the group bladerf now:
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

Install dump1090 dependency: rtl-sdr
```
pkg install rtl-sdr
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

## Further reading

[RTL-SDR on FreeBSD, or "hey, cool, I live near an airport, I wonder if ADSB works.."](https://forums.freebsd.org/threads/52157/)
