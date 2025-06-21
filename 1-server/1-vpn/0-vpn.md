---
layout: default
title: VPN
parent: Server
nav_order: 2
---

## Configure VPN client

### Add the configuration

Install OpenVPN client on the server

```bash
sudo apt update
sudo apt install openvpn -y
```

Create OpenVPN client config directory

```bash
sudo mkdir -p /etc/openvpn/client/
```

Copy your client config file to that directory. Assuming you already copied `vpn-client.ovpn` to your home directory:

```bash
sudo mv ~/vpn-client.ovpn /etc/openvpn/client/client.conf
```

👉 The file must be renamed to `client.conf` for `systemd` to pick it up automatically.

### Configure the service

Enable and start the OpenVPN client service

```bash
sudo systemctl enable openvpn-client@client
sudo systemctl start openvpn-client@client
```

Verify that OpenVPN client is running correctly

```bash
sudo systemctl status openvpn-client@client
```

You should see `Active: active (running)`.

(Optional) Check VPN interface and IP

```bash
ip addr show tun0
```

You should see an IP like `10.8.0.x` assigned.
