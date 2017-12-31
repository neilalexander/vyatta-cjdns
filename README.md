# cjdns for Ubiquiti EdgeOS / VyOS

### Introduction

The `vyatta-cjdns` package provides cjdns support on supported Ubiquiti EdgeMAX, VyOS and potentially other Vyatta-based routers.  It is integrated with the command line interface (CLI) allowing cjdns to be configured through the standard configuration system.

### Compatibility

|                       | Architecture | Compatible |                      Notes                                    |
|-----------------------|:------------:|:----------:|:-------------------------------------------------------------:|
|    EdgeRouter X (ERX) |    mipsel    |     Yes    |                                                               |
| EdgeRouter Lite (ERL) |    mips64    |     Yes    |                                                               |
|       VyOS 1.1.x      | i386, amd64  |     Yes    | No support for IPv6 masquerade                                |

### Install / Upgrade

Either download or build a release and copy it to the router, then install/upgrade it:
```
sudo dpkg -i vyatta-cjdns-x.x.x-xxxxxx.deb
```
If you are upgrading from a previous release of `vyatta-cjdns`, then restart cjdns once the upgraded package is installed:
```
restart cjdns tun0
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
set interfaces cjdns tun0 ethernet-interface 0 beacon listen-send
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
configure
set firewall ipv6-name CJD_LOCAL default-action drop
set firewall ipv6-name CJD_LOCAL rule 10 action accept
set firewall ipv6-name CJD_LOCAL rule 10 state established enable
set firewall ipv6-name CJD_LOCAL rule 10 state related enable
set firewall ipv6-name CJD_LOCAL rule 20 action drop
set firewall ipv6-name CJD_LOCAL rule 20 state invalid enable
set interfaces cjdns tun0 firewall local ipv6-name CJD_LOCAL
configure
```

#### Masquerade

If you want to allow other IPv6 hosts on your network to communicate through cjdns, you can configure an IPv6 masquerade rule. All traffic sent from other hosts on the network through the cjdns interface will be NAT'd.

For example:
```
configure
set interfaces cjdns tun0 masquerade from xxxx:xxxx:xxxx::/48
commit
```
If you have multiple IPv6 subnets, then they can be configured individually by setting multiple `masquerade from` source ranges. Both private/ULA and public IPv6 subnets are acceptable.

IPv6 masquerade is not supported on VyOS 1.1.x due to missing support in the kernel.

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

### Upgrading EdgeOS ###

When EdgeOS is upgraded, `vyatta-cjdns` will be uninstalled and, as a result, the cjdns interface configuration will be missing from the *running* configuration after upgrade. In order to preserve your `vyatta-cjdns` configuration through a system upgrade, use the following steps.

Before upgrading EdgeOS, make sure to save your running configuration to storage:
```
configure
save
```
Download a backup of your configuration from the router and then upgrade EdgeOS.

Once the EdgeRouter has come back up, you must reinstall `vyatta-cjdns` and then reload the *saved* configuration from storage:
```
sudo dpkg -i vyatta-cjdns-x.x.x-xxxxxx.deb
configure
load
compare
```
If at this point your `interfaces cjdns` configuration is displayed, then `commit` to make it effective:
```
commit
```
If you do not see your `interfaces cjdns` configuration at this point, then restore from your previously backed up configuration.

**WARNING:** Do not issue the `save` command at any point after the system upgrade or make any other configuration changes at. all **UNTIL** you are happy that your cjdns configuration has been successfully committed, otherwise it may be lost.

Verify that `cjdroute` is running again, in operational view:
```
ps -ef | grep cjdroute
show interfaces cjdns tunX peers
```

### Footnotes

If cjdns fails to start, you can find logging output in `/tmp/cjdroute.tunX.log`, where `tunX` is the specified interface.

You may also need to manually adjust your firewall to allow traffic on the `bind-address` that you specified.
