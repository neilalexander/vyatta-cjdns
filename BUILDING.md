# Building vyatta-cjdns

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
