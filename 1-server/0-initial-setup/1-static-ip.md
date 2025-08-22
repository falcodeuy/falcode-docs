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

By default, cloud-init will overwrite `/etc/netplan/01-netcfg.yaml` on each boot.  
To prevent this, make sure the following file exists:

```bash
sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg <<EOF
network: {config: disabled}
EOF
````

---

### 2. Edit your existing Netplan configuration

Instead of creating a new file, edit the existing one `/etc/netplan/01-netcfg.yaml`.
Adjust the interface name (`enp0s8` in most VirtualBox setups), IP address, prefix, gateway and DNS as needed:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.1.127/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

> **Note:** Always confirm your interface name with `ip a`.
> It may appear as `enp0s3`, `enp0s8` or something else depending on your VM configuration.
> Use exactly the name shown in the output.

---

### 3. Secure the Netplan file

Netplan requires its files to be unreadable by group/other. Set permissions to `600`:

```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
```

---

### 4. Generate and apply the new config

Generate the backend files, then apply changes:

```bash
sudo netplan generate
sudo netplan apply
```

---

### 5. Validate and reboot

1. Check that only your static IP is present:

   ```bash
   ip a show dev enp0s8
   ```

2. Reboot to confirm persistence:

   ```bash
   sudo reboot
   ```

3. After reboot, run `ip a` again — you should see **only** `192.168.1.127/24` on `enp0s8`.

---

> **Tip:** If you ever need to revert, rename or delete `/etc/netplan/01-netcfg.yaml` and also delete `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg`, then reboot.
