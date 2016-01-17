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

To establish a peering is straight-forward; replace `bind-address a.b.c.d:e` with the address you want cjdroute to listen on in `ip:port` format and replace `peers a.b.c.d:e` with the `ip:port` address of your peer:
```
configure
set interfaces cjdns tun0 udp-interface 0 bind-address a.b.c.d:e
set interfaces cjdns tun0 udp-interface 0 peers a.b.c.d:e password xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
set interfaces cjdns tun0 udp-interface 0 peers a.b.c.d:e publickey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.k
commit
```
To configure beacons to automatically peer with other devices on your network using ethernet (assuming `switch0` is your internal interface):
```
configure
set interfaces cjdns tun0 ethernet-interface 0 bind-interface switch0
set interfaces cjdns tun0 ethernet-interface 0 beacon 2
commit
```
To manually configure your IPv6 address and keypair with a pre-existing one (i.e. to bring in an existing identity):
```
configure
set interfaces cjdns tun0 publickey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.k
set interfaces cjdns tun0 privatekey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
set interfaces cjdns tun0 ipv6 xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
commit
```
An example stateful firewall configuration that will block unexpected incoming traffic on the cjdns interface, i.e. to prevent other people from reaching your ssh or Web UI ports:
```
set firewall ipv6-name CJD_LOCAL default-action drop
set firewall ipv6-name CJD_LOCAL rule 10 action accept
set firewall ipv6-name CJD_LOCAL rule 10 state established enable
set firewall ipv6-name CJD_LOCAL rule 10 state related enable
set firewall ipv6-name CJD_LOCAL rule 20 action drop
set firewall ipv6-name CJD_LOCAL rule 20 state invalid enable
set interfaces cjdns tun0 firewall local ipv6-name CJD_LOCAL
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

### Footnotes

There is very little input validation right now on the configuration, so if you enter badly-formed config then `cjdroute` will simply fail to start. 

You may also need to manually adjust your firewall to allow traffic on the `bind-address` that you specified.

The EdgeRouter X claims architecture `mipsel`, but the EdgeRouter Lite claims architecture `mips`. If you have problems installing the `.deb` file onto the EdgeRouter Lite, then change the `Architecture: mipsel` line in `debian/control` to `Architecture: mips` and then run `make package` to rebuild the package.
