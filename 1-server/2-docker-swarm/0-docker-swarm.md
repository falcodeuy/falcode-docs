---
layout: default
title: Docker swarm
parent: Server
nav_order: 3
---

## 🐳 Docker Swarm Setup over VPN

To add a new node the if the manager is a dynamic DNS machine, you will need to configure a VPN before start. See [Configure VPN](http://127.0.0.1:4000/1-server/docker-swarm/0-docker-swarm.html#-docker-swarm-setup-over-vpn)

### Initialize Swarm on the manager node

Run the following on your **manager** node:

```bash
docker swarm init --advertise-addr 10.8.0.1
```

Replace `10.8.0.1` with the manager’s VPN IP address.

This will output a `docker swarm join` command with a token like:

```bash
docker swarm join --token SWMTKN-... 10.8.0.1:2377
```

Copy this command — you'll need it for the worker nodes.

---

### Join a worker node

On each **worker node** (also connected via VPN), run:

```bash
docker swarm join --token SWMTKN-... 10.8.0.1:2377
```

---

### Verify the cluster from the manager

On the manager node, run:

```bash
docker node ls
```

You should see something like:

```
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
e5sd...fgh8                   manager-node        Ready               Active              Leader
asdf...erh3                   worker-node-1       Ready               Active
```

---

### Label nodes for targeted deployment

From the manager node, assign labels:

```bash
docker node update --label-add role=infra manager-node
docker node update --label-add role=frontend worker-node-1
```

You can also assign custom tags:

```bash
docker node update --label-add zone=home worker-node-1
docker node update --label-add storage=fast manager-node
```

---

### Use labels in your `docker-compose.yml`

Example service with placement constraint:

```yaml
services:
  frontend:
    image: myorg/frontend:latest
    deploy:
      placement:
        constraints:
          - node.labels.role == frontend
```

This ensures the service will only run on nodes labeled `role=frontend`.

---

### Optional: Leave or reset the cluster

To leave the swarm:

* On workers:

```bash
docker swarm leave
```

* On manager:

```bash
docker swarm leave --force
```

These help organize service placement in a distributed architecture.
