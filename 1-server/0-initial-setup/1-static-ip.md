---
layout: default
title: Static IP
parent: Initial setup
grand_parent: Server
nav_order: 1
---

## Static IP Configuration with Netplan on Ubuntu

Follow these steps to disable cloud-init network management, define a static IP for your VM, and apply the configuration.

---

### 1. Disable cloud-init network configuration

By default, cloud-init will overwrite `/etc/netplan/50-cloud-init.yaml` on each boot. Create a file to disable its network handling:

```bash
sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg <<EOF
network: {config: disabled}
EOF
````

---

### 2. Create or edit your Netplan configuration

Edit the file `/etc/netplan/50-cloud-init.yaml`. Adjust the interface name (`enp0s3`), IP address, prefix, gateway and DNS as needed:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.68/24
      nameservers:
        addresses: [8.8.8.8]
      routes:
        - to: default
          via: 192.168.1.1
```

---

### 3. Secure the Netplan file

Netplan requires its files to be unreadable by group/other. Set permissions to `600`:

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
```

---

### 4. Generate and apply the new config

First, generate the backend files from your YAML without touching the live network and then apply changes:

```bash
sudo netplan generate
sudo netplan apply
```

If you see:

```bash
WARNING:root:Cannot call Open vSwitch: ovsdb-server.service is not running.
```

you can safely ignore it unless you want to use OVS.

### 6. Validate and reboot

1. Check that only your static IP is present:

    ```bash
    ip a show dev enp0s3
    ```

2. Reboot to confirm persistence:

    ```bash
    sudo reboot
    ```

3. After reboot, run `ip a` again — you should see **only** `192.168.1.68/24` on `enp0s3`.

---

> **Tip:** If you ever need to revert, remove or rename `/etc/netplan/01-netcfg.yaml` and delete `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg`, then reboot.