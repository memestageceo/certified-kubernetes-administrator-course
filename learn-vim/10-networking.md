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

```
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

| Record Type | Example Hostname | Description |
|-------------|------------------|-------------|
| A | web-server | Maps hostname to an IPv4 address (e.g., 192.168.1.1) |
| AAAA | web-server | Maps hostname to an IPv6 address (e.g., 2001:0db8:85a3:0000:0000:8a2e:0370:7334) |
| CNAME | food.web-server | Aliases one hostname to another (e.g., aliasing to eat.web-server or hungry.web-server) |

**Note:** A records handle IPv4 addresses, AAAA records are for IPv6, and CNAME records allow hostname aliasing.
