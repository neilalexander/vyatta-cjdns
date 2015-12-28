# cjdns for Ubiquiti EdgeOS / Vyatta

### Introduction

This is a cjdns distributable package for Ubiquiti EdgeOS or Vyatta-based platforms such as VyOS. It supports configuration through the standard Vyatta configuration editor and should therefore be very straight-forward to deploy.

At this time this package is in very early stages of development, but the ultimate aim is to provide binary builds for some commonly-used platforms with pre-built cjdns binaries.

### Building

On Debian Jessie, start by installing the toolchain:
```
echo "deb http://emdebian.org/tools/debian/ jessie main" >> /etc/apt/sources.list

wget http://emdebian.org/tools/debian/emdebian-toolchain-archive.key
apt-key add emdebian-toolchain-archive.key

dpkg --add-architecture mipsel
apt-get update
apt-get install crossbuild-essential-mipsel
```
Compile the package then by cloning the repository and running 'make':
```
make
```
