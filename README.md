# cjdns for Ubiquiti EdgeOS / VyOS

### Introduction

This is a cjdns distributable package for Ubiquiti EdgeOS, VyOS and potentially other Vyatta-based systems. It supports configuration through the standard configuration editor and should therefore be very straight-forward to deploy.

At this time this package is in very early stages of development, but the ultimate aim is to provide binary builds for some commonly-used platforms with pre-built cjdns binaries.

### Compatibility

|                       | Architecture | Compatible |                      Notes                                    |
|-----------------------|:------------:|:----------:|:-------------------------------------------------------------:|
|    EdgeRouter X (ERX) |    mipsel    |     Yes    | Builds with crossbuild-essential, see below                   |
| EdgeRouter Lite (ERL) |    mips64    |     Yes    | Builds with Codescape SDK as mips32, see below                |
|       VyOS 1.1.x      | i386, amd64  |     Yes    | Builds with crossbuild-essential on Squeeze only, see below   |

### Building for Supported Platforms

Use the provided shell scripts, which will do most of the hard work for you:
```
./build-edgerouter-x
./build-edgerouter-lite
./build-vyos
```

### Manually Building for EdgeRouter X

On 64-bit Debian Jessie, start by installing the toolchain:
```
echo "deb http://emdebian.org/tools/debian/ jessie main" >> /etc/apt/sources.list

wget http://emdebian.org/tools/debian/emdebian-toolchain-archive.key
apt-key add emdebian-toolchain-archive.key

dpkg --add-architecture mipsel
apt-get update
apt-get install -y crossbuild-essential-mipsel
```
Compile the package from the `master` branch by cloning the repository and running 'make':
```
cd vyatta-cjdns
PREFIX='mipsel-linux-gnu-' PKGARCH='mipsel' make -e
```
Alternatively, clone and compile the `crashey` branch:
```
cd vyatta-cjdns
PREFIX='mipsel-linux-gnu-' PKGARCH='mipsel' BRANCH=crashey make -e
```
The package `vyatta-cjdns.deb` will be created in the parent directory. Copy it to the EdgeRouter and install it:
```
sudo dpkg -i vyatta-cjdns.deb
```

### Manually Building for EdgeRouter Lite

On 64-bit Debian Jessie, start by installing the build-essential package:
```
apt-get update
apt-get install -y build-essential
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
Clone the repository and then initiate the build by running:
```
PREFIX='mips-mti-linux-gnu-' PKGARCH='mips' make -e
```
Alternatively, clone and compile the `crashey` branch:
```
cd vyatta-cjdns
PREFIX='mips-mti-linux-gnu-' PKGARCH='mips' BRANCH=crashey make -e
```
The package `vyatta-cjdns.deb` will be created in the parent directory. Copy it to the EdgeRouter and install it:
```
sudo dpkg -i vyatta-cjdns.deb
```

### Manually Building for VyOS 1.1.7

At present VyOS 1.1.x are based on Debian Squeeze. To match the glibc version it is best to also use Debian Squeeze to target it. Future versions of VyOS (1.2.x) will be based on Debian Jessie.

Generally it is easiest to target the VyOS architecture you are using (`i386`, `amd64`) by using the same architecture in your build environment. However, you may be able to target `i386` from an `amd64` system by specifying some `CFLAGS` and `PKGARCH="i386"` to `make native`, but that process is not documented here.

On Debian Squeeze, start by installing the toolchain and dependencies:
```
echo "deb http://archive.debian.org/debian/ squeeze main non-free" >> /etc/apt/sources.list
apt-get update
apt-get install -y build-essential git
```
Build and install `python2.7`, as the `python2.6` bundled with Squeeze is not adequate:
```
wget https://www.python.org/ftp/python/2.7/Python-2.7.tgz
tar zxf Python-2.7.tgz
cd Python-2.7/
./configure
make
make install
```
Compile the package from the `master` branch by cloning the repository and running 'make native':
```
cd vyatta-cjdns
make native
```
Alternatively, clone and compile the `crashey` branch:
```
cd vyatta-cjdns
make native BRANCH=crashey
```
The package `vyatta-cjdns.deb` will be created in the parent directory. Copy it to the VyOS router and install it:
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
set interfaces cjdns tun0 description CJDNS
commit
```
This automatically generates a new private key and then populates the IPv6 address, public key, private key and admin socket details into the config, as shown with `show interfaces cjdns tun0` in the configure view.

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

An IPv6 address and a keypair are automatically generated when you create a new cjdns interface. The `publickey`, `privatekey` and `ipv6` fields will be automatically populated with these.

To override the automatically generated keypair and manually configure your own IPv6 address and keypair (i.e. to bring in an existing keypair from another machine):
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

### Manual Configuration

If you prefer not to make use of the EdgeRouter CLI to manage `cjdroute.conf`, or you prefer to use another utility to manage `cjdroute.conf`, it is still possible to use the vyatta-cjdns package, with the following caveats:

- The interface will not be recognised by the EdgeRouter GUI, or by operational commands in the CLI such as `show interfaces`
- It may not be possible to configure firewall rules or other services for the cjdns interface using the EdgeRouter GUI or CLI as a result
- If you are not careful, the EdgeRouter GUI or CLI may allow you to incorrectly define a different tunnel interface using the same `tunX` name as the cjdns tunnel
- If a cjdns tunnel is defined later in the EdgeRouter GUI or CLI with the same `tunX` name, the configuration may be overwritten partially or entirely

To generate the configuration, choose a `tunX` interface which is not in use by anything else on the system, and then generate the config into `/config/cjdroute.tunX.conf`, uncommenting and amending the `tunDevice` directive in the config file to always use the chosen `tunX` interface. For example, to use `tun1`:
```
/opt/vyatta/sbin/cjdroute --genconf > /config/cjdroute.tun1.conf
sed -i 's/\/\/"tunDevice": "tun0"/"tunDevice": "tun1"/' /config/cjdroute.tun1.conf
restart cjdns tun1
```
Use the Crash Detection scheduled task above to automatically check and start cjdns on a given interval, as it will not be automatically started by the EdgeRouter configuration system when using this method.

### Footnotes

There is very little input validation right now on the configuration, so if you enter badly-formed config then `cjdroute` will simply fail to start.

If cjdns fails to start, you can find logging output in `/tmp/cjdroute.tunX.log`, where `tunX` is the specified interface. 

You may also need to manually adjust your firewall to allow traffic on the `bind-address` that you specified.
