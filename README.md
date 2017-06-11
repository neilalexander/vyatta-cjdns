# cjdns for Ubiquiti EdgeOS

### Introduction

This is a cjdns distributable package for Ubiquiti EdgeOS. It supports configuration through the standard configuration editor and should therefore be very straight-forward to deploy.

At this time this package is in very early stages of development, but the ultimate aim is to provide binary builds for some commonly-used platforms with pre-built cjdns binaries.

### Compatibility

|                       | Architecture | Compatible |                      Notes                     |
|-----------------------|:------------:|:----------:|:----------------------------------------------:|
|    EdgeRouter X (ERX) |    mipsel    |     Yes    | Builds with crossbuild-essential, see below    |
| EdgeRouter Lite (ERL) |    mips64    |     Yes    | Builds with Codescape SDK as mips32, see below |

### Building for EdgeRouter X

On 64-bit Debian Jessie, start by installing the toolchain:
```
echo "deb http://emdebian.org/tools/debian/ jessie main" >> /etc/apt/sources.list

wget http://emdebian.org/tools/debian/emdebian-toolchain-archive.key
apt-key add emdebian-toolchain-archive.key

dpkg --add-architecture mipsel
apt-get update
apt-get install crossbuild-essential-mipsel
```
Compile the package from the `master` branch by cloning the repository and running 'make':
```
cd vyatta-cjdns
make
```
Alternatively, clone and compile the `crashey` branch:
```
cd vyatta-cjdns
make BRANCH=crashey
```
The package `vyatta-cjdns.deb` will be created in the parent directory. Copy it to the EdgeRouter and install it:
```
sudo dpkg -i vyatta-cjdns.deb
```

### Building for EdgeRouter Lite

On 64-bit Debian Jessie, start by installing the build-essential package:
```
sudo apt-get update
sudo apt-get install -y build-essential
```
So far the only proven working toolchain is the Codescape SDK 2015.01.7 (and 2015.06.05).

Download the toolchain, assuming your homedir:
```
wget http://codescape-mips-sdk.imgtec.com/components/toolchain/2015.06-05/Codescape.GNU.Tools.Package.2015.06-05.for.MIPS.MTI.Linux.CentOS-5.x86_64.tar.gz
tar xf Codescape.GNU.Tools.Package.2015.06-05.for.MIPS.MTI.Linux.CentOS-5.x86_64.tar.gz
```
Add the bin dir to your `PATH` variable:
```
PATH=$HOME/mips-mti-linux-gnu/2015.06-05/bin:$PATH
```
Clone the repository, edit the file `vyatta-cjdns/debian/control` and change `Architecture: mipsel` to `Architecture: mips`, and then initiate the build by running:
```
PREFIX='mips-mti-linux-gnu-' make -e
```
The package `vyatta-cjdns.deb` will be created in the parent directory. Copy it to the EdgeRouter and install it:
```
sudo dpkg -i vyatta-cjdns.deb
```

### Configuration

All configuration is entered through the CLI. `set` commands, as listed below, will add new configuration and the cjdroute configuration file will be updated automatically. To remove configuration, for instance to remove a peering, authorised password or IP tunnel setting, replace the `set` keyword with `delete`.

cjdroute is restarted automatically after a configuration change is made.

### Initial

Start by creating the default configuration on the interface:
```
configure
set interfaces cjdns tun0
commit
```
This automatically populates the IPv6 address, public key, private key and admin socket details, as shown with `show interfaces cjdns tun0` in the configure view.

#### Peerings

To establish a peering is straight-forward; replace `bind-address a.b.c.d:e` with the address you want cjdroute to listen on in `ip:port` format and replace `peers a.b.c.d:e` with the `ip:port` address of your peer. Use `login` to specify the login name (which is sometimes `default-login`), and `peername` to identify the peering friendly name (as seen in the peering stats):
```
configure
set interfaces cjdns tun0 udp-interface 0 bind-address a.b.c.d:e
set interfaces cjdns tun0 udp-interface 0 peers a.b.c.d:e password xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
set interfaces cjdns tun0 udp-interface 0 peers a.b.c.d:e publickey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.k
set interfaces cjdns tun0 udp-interface 0 peers a.b.c.d:e login xxxxxxxx
set interfaces cjdns tun0 udp-interface 0 peers a.b.c.d:e peername xxxxxxxx
commit
```
To configure beacons to automatically peer with other devices on your network using ethernet (assuming `switch0` is your internal interface):
```
configure
set interfaces cjdns tun0 ethernet-interface 0 bind-interface switch0
set interfaces cjdns tun0 ethernet-interface 0 beacon 2
commit
```
To configure new authorized passwords for incoming connections:
```
configure
set interfaces cjdns tun0 authorized-password user1 password password1
set interfaces cjdns tun0 authorized-password user2 password password2
commit
```

#### Identity

An IPv6 address and a keypair are automatically generated when you create a new cjdns interface. The `publickey`, `privatekey` and `ipv6` fields will be automatically populated with these. To manually configure your own IPv6 address and keypair (i.e. to bring in an existing keypair from another machine):
```
configure
set interfaces cjdns tun0 publickey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.k
set interfaces cjdns tun0 privatekey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
set interfaces cjdns tun0 ipv6 xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
commit
```

#### Firewall

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

#### IP Tunnel

To connect to and receive a tunnel prefix from a remote peer, where `xxx.k` is the remote public key:
```
configure
set interfaces cjdns tun0 ip-tunnel xxx.k connect
commit
```
To provide an IPv4 tunnel prefix to a remote peer, where `xxx.k` is the remote public key:
```
configure
set interfaces cjdns tun0 ip-tunnel xxx.k provide-ipv4-prefix x.x.x.x/x
commit
```
To provide an IPv6 tunnel prefix to a remote peer where, `xxx.k` is the remote public key:
```
configure
set interfaces cjdns tun0 ip-tunnel xxx.k provide-ipv6-prefix x::x/x
commit
```

#### Operational Commands

To see information about peerings, in operational view:
```
show interfaces cjdns tun0 peers
```
To see your IPv6 address and public/private keys, in operational view:
```
show interfaces cjdns tun0 identity
```
To restart a cjdns tunnel, in operational view:
```
restart cjdns tun0
```

#### Crash Detection

The cjdroute daemon is still in development and is prone to crashes sometimes. The easiest way to make sure that the process is restarted if it crashes is to schedule the `vyatta-check-cjdns` script to run at a regular interval:
```
configure
set system task-scheduler task check-cjdns executable path /opt/vyatta/sbin/vyatta-check-cjdns
set system task-scheduler task check-cjdns interval 1m
commit
```

### Footnotes

There is very little input validation right now on the configuration, so if you enter badly-formed config then `cjdroute` will simply fail to start.

You may also need to manually adjust your firewall to allow traffic on the `bind-address` that you specified.
