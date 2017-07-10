# Use Docker with Virtualbox on FreeBSD

# Introduction

Docker Toolbox (The legacy Implementation of Docker on Windows and Docker on macOS) is still widely used among Windows and macOS users. Thus, this method is independent from the host operating system as long as virtual machines can be run on the host.

# Steps

## Enable hardware virtualization

Since boot2docker is a 64-bit Linux image, we need to ensure that hardware virtualization is supported and enabled.

## Install docker-machine

Install from pkg:
```
pkg install docker-machine
```

## Install VirtualBox

Install from pkg:
```
pkg install virtualbox-ose
```

Or, make install from ports (the only option if you are using 12-CURRENT):
```
cd /usr/ports/sysutils/docker-machine && make install clean
cd /usr/ports/emulators/virtualbox-ose && make install clean
```
Ensure that you have the exact same source of your system under `/usr/src`, as virtualbox-ose needs to build kernel drivers based on that.

Check if VirtualBox drivers are properly loaded:
```
kldstat
```
Ensure the output contains:
```
vboxdrv.ko
```

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

## More usages

Please see: [Docker Docs - Get started with Docker Machine and a local VM](https://docs.docker.com/machine/get-started/#run-containers-and-experiment-with-machine-commands)
