# Configure a devbox VM in Virtualbox

Install GuestAdditions (try to run and install dependencies given in errors)

Clipboard sharing:
- Enable in virutalbox settings
Shared folder:
- Enable in Virtualbox settings
- Add user to `vboxsf` group:        `usermod -aG vboxsf <username>`
- Check if added to group correctly: `groups <username>`


# Ubuntu server: Initial installation options

## disk encryption

- passphrase: see secrets/
- recovery key: /var/log/installer/ or ~/recovery-key.txt

## Extra packages during install

- docker only

## File system summary

|Mount Point              | Use Case    | Size      |
|-------------------------|-------------|-----------|
|/                        | root        |  40.000G  |
|/var/lib/libvirt/images  | vm-storage  | 422.691G  |


# Setup after Ubuntu installation

## Additional packages:

- vim (+ settings via vimrc template `/config-templates/.vimrc`)
- net-tools, for ifconfig: `sudo apt install net-tools`
- network-manager, for nmcli: `supo apt install network-manager`

## Networking

### Set up static ip

1. Disable cloud-init dynamic network setup:

Create the following file:
```yaml /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
network: {config: disabled}
```

As described in the cloud init config below:
```yaml /etc/netplan/50-cloud-init.yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets: {}
    version: 2
    wifis:
        wlo1:
            access-points:
                Albatraoz 5GHZ:
                    password: littlemouse
            dhcp4: true
```

2. Manually add a new network config file:

Add config, the following is for the wifi network interface

```yaml  /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  wifis:
    wlo1:  # Wi-Fi interface name (ip addr show)
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.100/24   # Static IP address and subnet mask for Wi-Fi
      routes:
        - to: default
          via: 192.168.0.1  # Router's IP address
      nameservers:
        addresses:
          - 8.8.8.8  # Google DNS (you can add more, like 8.8.4.4 or 1.1.1.1)
          - 1.1.1.1
      access-points:
        "Albatraoz 5GHZ":          # Wi-Fi network name
          password: "littlemouse"  # Wi-Fi password
  ethernets:
    enp2s0: # Wired ethernet interface 1 (ip addr show)
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.101/24 # Static IP address and subnet mask for ethernet port 1
      routes:
        - to: default
          via: 192.168.0.1  # Router's IP address
      nameservers:
        addresses:
          - 8.8.8.8  # Google DNS (you can add more, like 8.8.4.4 or 1.1.1.1)
          - 1.1.1.1
    enp3s0: # Wired ethernet interface 2 (ip addr show)
      dhcp4: false
      dhcp6: false
      addresses:
        - 192.168.0.102/24 # Static IP address and subnet mask for ethernet port 2
      routes:
        - to: default
          via: 192.168.0.1  # Router's IP address
      nameservers:
        addresses:
          - 8.8.8.8  # Google DNS (you can add more, like 8.8.4.4 or 1.1.1.1)
          - 1.1.1.1
``` 

3. Apply the new config: `sudo netplan apply`

*note* `/etc/netplan/50-cloud-init.yaml`  should disappear by itself upon reboot

## TODO: store these yaml files in the nimbus repo (/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg and /etc/netplan/01-netcfg.yaml) ##

