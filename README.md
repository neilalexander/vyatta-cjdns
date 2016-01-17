# cjdns for Ubiquiti EdgeOS 

### Introduction

This is a cjdns distributable package for Ubiquiti EdgeOS. It supports configuration through the standard configuration editor and should therefore be very straight-forward to deploy.

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
The package `vyatta-cjdns.deb` will be created in the parent directory. 

### Configuration

To establish a peering is straight-forward; replace `bind-address` with the address you want cjdroute to listen on:
```
configure
set interfaces cjdns tun0 udp-interface 0 bind-address 'a.b.c.d:e'
set interfaces cjdns tun0 udp-interface 0 peers 'a.b.c.d:e' password xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
set interfaces cjdns tun0 udp-interface 0 peers 'a.b.c.d:e' publickey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.k
commit
```
To configure beacons to automatically peer with other devices on your network (assuming `switch0` is your internal interface):
```
configure
set interfaces cjdns tun0 ethernet-interface 0 bind-interface switch0
set interfaces cjdns tun0 ethernet-interface 0 beacon 2
commit
```
To manually configure your IPv6 address and keypair with a pre-existing one (i.e. to port in an old identity):
```
configure
set interfaces cjdns tun0 publickey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.k
set interfaces cjdns tun0 privatekey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
set interfaces cjdns tun0 ipv6 xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
commit
```
To see information about peerings, in operational view:
```
show interfaces cjdns peers
```
To see your IPv6 address and public/private keys, in operational view:
```
show interfaces cjdns tun0 identity
```
To restart a cjdns tunnel, in operational view:
```
restart cjdns tun0
```
