# Networking Commands and Configuration

## Basic Network Commands

### Network Interface Management

```bash
# View network interfaces
ip link

# Add IP address to interface
ip addr add 192.168.1.10/24 dev eth0

# Test connectivity
ping 192.168.1.11
```

### Routing

```bash
# View routing table
route

# Add static routes
ip route add 192.168.2.0/24 via 192.168.1.1
ip route add 192.168.2.0/24 via 192.168.1.6
```

## IP Forwarding Configuration

### Check IP Forwarding Status

```bash
cat /proc/sys/net/ipv4/ip_forward
```

### Enable IP Forwarding Permanently

To ensure this setting persists across reboots, modify `/etc/sysctl.conf` and add or update the following line:

```ini
net.ipv4.ip_forward = 1
```

## DNS and Name Resolution

### Hosts File Configuration

To make "db" recognizable, add an entry in the `/etc/hosts` file on Computer A. This informs the system that Computer B (192.168.1.11) is known as "db".

### DNS Server Configuration

Suppose your centralized DNS server is at IP address 192.168.1.100. You configure each host to use this server by editing the `/etc/resolv.conf` file:

```bash
cat /etc/resolv.conf
nameserver 192.168.1.100
```

### Name Resolution Order

The resolution order is defined in `/etc/nsswitch.conf`.

### Search Domains

Within many organizations, it is often convenient to use short hostnames. To resolve a short name (for example, "web") to its fully qualified domain name (FQDN, such as web.mycompany.com), add a search domain to your `/etc/resolv.conf` file:

```bash
cat >> /etc/resolv.conf
nameserver 192.168.1.100
search mycompany.com prod.mycompany.com
```

## DNS Record Types

| Record Type | Example Hostname | Description                                                                             |
| ----------- | ---------------- | --------------------------------------------------------------------------------------- |
| A           | web-server       | Maps hostname to an IPv4 address (e.g., 192.168.1.1)                                    |
| AAAA        | web-server       | Maps hostname to an IPv6 address (e.g., 2001:0db8:85a3:0000:0000:8a2e:0370:7334)        |
| CNAME       | food.web-server  | Aliases one hostname to another (e.g., aliasing to eat.web-server or hungry.web-server) |

**Note:** A records handle IPv4 addresses, AAAA records are for IPv6, and CNAME records allow hostname aliasing.

---

## Docker Networking

```bash
# `--rm` when stopped, automatically delete container
docker run -itd --rm thor busybox

bridge link
```

### 🐳 Docker Networking Commands & Concepts

#### 1. **Disable Networking**

```bash
docker run --network none nginx
```

- This runs the container with **no network access**.

---

#### 2. **Host Networking**

```bash
docker run --network host nginx
```

- If a web application inside the container listens on port `80`, it becomes **directly accessible** on port `80` of the host.

---

#### 3. **Bridge Network (Default)**

- When Docker is installed, it creates a default **bridge network** (`docker0`) with a subnet like `172.17.0.0/16`.
- Each container connected to this network gets a **unique IP address** from this subnet.

---

#### 4. **Inspect Container Network Settings**

```bash
docker inspect <container_id>
```

Example output:

```json
"NetworkSettings": {
  "Bridge": "",
  "SandboxID": "b3165c10a92b50edc4c8aa5f37273e180907ded31",
  "SandboxKey": "/var/run/docker/netns/b3165c10a92b"
}
```

---

#### 5. **Container Creation Steps**

Each time a new container is created, Docker performs:

1. Creates a **new network namespace**.
2. Establishes a **pair of virtual interfaces**.
3. Attaches one end to the container’s namespace, the other to the `docker0` bridge.
4. Assigns an **IP address** to the container's interface.

---

#### 6. **Virtual Interface Pairing**

- Interface pairs are numbered consistently:
  - Example: `veth7` and `veth8`, `veth9` and `veth10`.

---

#### 7. **Port Forwarding with iptables**

Docker uses `iptables` to forward ports from host to container.

```bash
iptables -t nat -A PREROUTING -j DNAT --dport 8080 --to-destination 80
```

To view NAT rules:

```bash
iptables -nvL -t nat
```

## Container Network Interface

bridge add 2e34dcf34 /var/run/netns/2e34dcf34

When container platforms like Rocket or Kubernetes spin up a new container, they invoke this bridge program—passing the container ID and namespace—to automatically set up the network.
the following example that demonstrates how Kubernetes handles networking with Docker: docker run --network=none nginx
bridge add 2e34dcf34 /var/run/netns/2e34dcf34

ip link
ip addr
ip addr add 192.168.1.10/24 dev eth0
ip route
ip route add 192.168.1.0/24 via 192.168.2.1
cat /proc/sys/net/ipv4/ip_forward
arp
netstat -plnt
