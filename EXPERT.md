# Expert Notes

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
