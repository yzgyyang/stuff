# Use Docker with Virtualbox on FreeBSD

# Introduction

# Steps

## Ensure hardware virtualization is supported and enabled

Since boot2docker is a 64-bit Linux image, the VT technology must be enabled.

## Install VirtualBox and docker-machine

Install from pkg:
```
pkg install virtualbox-ose docker-machine
```

Or, install from ports (only option if you are using 12-CURRENT):
```
cd /usr/ports/sysutils/docker-machine && make install clean
cd /usr/ports/emulators/virtualbox-ose && make install clean
```
Ensure that you have the exact same source of your system under `/usr/src`, as virtualbox-ose needs to build kernel drivers based on that.

## Create a machine

```
docker-machine create --driver virtualbox linux
```
```
docker-machine env Linux
```
```
docker-machine ls
```

Try it out:
```
docker run busybox echo hello world
```

## Further usages

Please see: [Docker Docs - Get started with Docker Machine and a local VM](https://docs.docker.com/machine/get-started/#run-containers-and-experiment-with-machine-commands)
