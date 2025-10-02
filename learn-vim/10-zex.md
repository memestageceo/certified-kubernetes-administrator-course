# Network Namespaces Step-by-Step Tutorial

## What Are Network Namespaces?

Think of network namespaces as **separate network worlds** on the same computer. Each namespace has its own:

- Network interfaces (like separate ethernet cards)
- IP addresses
- Routing tables
- Firewall rules

It's like having multiple computers, but they're all virtual and running on one physical machine.

## Setup Requirements

- WSL Ubuntu (which you have) ✅
- Root/sudo access
- Basic Linux commands knowledge

---

## Exercise 1: Understanding the Problem (5 minutes)

### Goal: See what the host can see vs what containers see

**Step 1a: Check your host network**

```bash
# See all network interfaces on your WSL
ip link show

# See routing table
ip route show

# See ARP table (who we've talked to recently)
arp -a
```

**Step 1b: Understand the concept**

- Right now, everything runs in the same "network world"
- All processes see the same network interfaces
- All processes share the same routing table

**What you'll see:**

- Probably `eth0` (your main interface)
- Maybe `lo` (loopback)
- Routes to your Windows host and internet

---

## Exercise 2: Creating Your First Network Namespace (10 minutes)

### Goal: Create an isolated network environment

**Step 2a: Create a namespace**

```bash
# Create a new network namespace called 'red'
sudo ip netns add red

# List all namespaces
sudo ip netns list
```

**Step 2b: Compare host vs namespace**

```bash
# What can the HOST see?
ip link show

# What can the NAMESPACE see?
sudo ip netns exec red ip link show
```

**What you'll notice:**

- Host: sees all your normal interfaces
- Namespace: only sees `lo` (loopback) and it's DOWN!

**Step 2c: Try to ping from namespace**

```bash
# This will fail!
sudo ip netns exec red ping 8.8.8.8
```

**Why it fails:** The namespace is completely isolated - no internet access!

---

## Exercise 3: Connecting Two Isolated Worlds (15 minutes)

### Goal: Create two namespaces and connect them

**Step 3a: Create second namespace**

```bash
# Create blue namespace
sudo ip netns add blue

# Verify both exist
sudo ip netns list
```

**Step 3b: Create a virtual cable**

```bash
# Think of this as a virtual ethernet cable with two ends
sudo ip link add veth-red type veth peer name veth-blue

# See the cable on the host
ip link show | grep veth
```

**Step 3c: Put each end in different namespaces**

```bash
# Put red end in red namespace
sudo ip link set veth-red netns red

# Put blue end in blue namespace
sudo ip link set veth-blue netns blue

# Check: cable ends are gone from host
ip link show | grep veth
```

**Step 3d: Give each end an IP address**

```bash
# Red namespace gets 192.168.1.1
sudo ip netns exec red ip addr add 192.168.1.1/24 dev veth-red

# Blue namespace gets 192.168.1.2
sudo ip netns exec blue ip addr add 192.168.1.2/24 dev veth-blue
```

**Step 3e: Turn on the interfaces**

```bash
# Turn on red end
sudo ip netns exec red ip link set veth-red up
sudo ip netns exec red ip link set lo up

# Turn on blue end
sudo ip netns exec blue ip link set veth-blue up
sudo ip netns exec blue ip link set lo up
```

**Step 3f: Test the connection**

```bash
# Ping from red to blue
sudo ip netns exec red ping -c 3 192.168.1.2

# Ping from blue to red
sudo ip netns exec blue ping -c 3 192.168.1.1
```

**Success!** You've created two isolated networks that can talk to each other!

---

## Exercise 4: Understanding What You Built (10 minutes)

### Goal: Examine the isolated networks

**Step 4a: Check routing tables**

```bash
# Red's routing table
sudo ip netns exec red ip route show

# Blue's routing table
sudo ip netns exec blue ip route show

# Compare to host routing table
ip route show
```

**Step 4b: Check ARP tables**

```bash
# After the pings, check who red has talked to
sudo ip netns exec red arp -a

# Check who blue has talked to
sudo ip netns exec blue arp -a
```

**What you learned:**

- Each namespace has its own routing table
- Each namespace learns MAC addresses independently
- They're truly separate network environments

---

## Exercise 5: The Bridge - Connecting Multiple Namespaces (20 minutes)

### Goal: Create a "network switch" to connect multiple namespaces

**Step 5a: Clean up and start fresh**

```bash
# Delete old namespaces
sudo ip netns delete red
sudo ip netns delete blue

# Create new ones
sudo ip netns add red
sudo ip netns add blue
sudo ip netns add green
```

**Step 5b: Create a virtual bridge (think: network switch)**

```bash
# Create bridge
sudo ip link add br0 type bridge

# Turn it on
sudo ip link set br0 up

# Give bridge an IP (it becomes the "router")
sudo ip addr add 192.168.100.1/24 dev br0
```

**Step 5c: Create virtual cables for each namespace**

```bash
# Create cable for red namespace
sudo ip link add veth-red type veth peer name veth-red-br

# Create cable for blue namespace
sudo ip link add veth-blue type veth peer name veth-blue-br

# Create cable for green namespace
sudo ip link add veth-green type veth peer name veth-green-br
```

**Step 5d: Connect everything**

```bash
# Put one end in each namespace
sudo ip link set veth-red netns red
sudo ip link set veth-blue netns blue
sudo ip link set veth-green netns green

# Connect other ends to bridge
sudo ip link set veth-red-br master br0
sudo ip link set veth-blue-br master br0
sudo ip link set veth-green-br master br0

# Turn on bridge connections
sudo ip link set veth-red-br up
sudo ip link set veth-blue-br up
sudo ip link set veth-green-br up
```

**Step 5e: Configure namespace interfaces**

```bash
# Configure red
sudo ip netns exec red ip addr add 192.168.100.10/24 dev veth-red
sudo ip netns exec red ip link set veth-red up
sudo ip netns exec red ip link set lo up

# Configure blue
sudo ip netns exec blue ip addr add 192.168.100.20/24 dev veth-blue
sudo ip netns exec blue ip link set veth-blue up
sudo ip netns exec blue ip link set lo up

# Configure green
sudo ip netns exec green ip addr add 192.168.100.30/24 dev veth-green
sudo ip netns exec green ip link set veth-green up
sudo ip netns exec green ip link set lo up
```

**Step 5f: Test the network**

```bash
# Red can ping blue
sudo ip netns exec red ping -c 2 192.168.100.20

# Blue can ping green
sudo ip netns exec blue ping -c 2 192.168.100.30

# All can ping the bridge (host)
sudo ip netns exec red ping -c 2 192.168.100.1
```

**Congratulations!** You've built a virtual network switch!

---

## Exercise 6: Internet Access (Advanced - 15 minutes)

### Goal: Give namespaces internet access

**Step 6a: Add default routes**

```bash
# Tell namespaces that bridge is their gateway to the world
sudo ip netns exec red ip route add default via 192.168.100.1
sudo ip netns exec blue ip route add default via 192.168.100.1
sudo ip netns exec green ip route add default via 192.168.100.1
```

**Step 6b: Enable IP forwarding on host**

```bash
# Allow the host to forward packets
sudo sysctl net.ipv4.ip_forward=1
```

**Step 6c: Add NAT rule**

```bash
# This translates private IPs to your public IP
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -j MASQUERADE
```

**Step 6d: Test internet access**

```bash
# Try to ping Google DNS from red namespace
sudo ip netns exec red ping -c 3 8.8.8.8

# Try to ping from blue
sudo ip netns exec blue ping -c 3 8.8.8.8
```

---

## What You've Learned

1. **Isolation**: Each namespace is completely separate
2. **Virtual Cables**: veth pairs connect namespaces
3. **Bridges**: Act like network switches
4. **Routing**: Each namespace has its own routing table
5. **NAT**: Allows private networks to access internet

## Real-World Applications

This is exactly how **Docker containers** get their network isolation:

- Each container = one network namespace
- Docker bridge = your br0 bridge
- Container networking = what you just built!

## Next Steps

Try these challenges:

1. Create a namespace that can't access the internet
2. Set up firewall rules between namespaces
3. Create multiple bridges for network segmentation
4. Monitor traffic between namespaces

## Cleanup Commands

```bash
# Delete all namespaces
sudo ip netns delete red
sudo ip netns delete blue
sudo ip netns delete green

# Delete bridge
sudo ip link delete br0

# Clear iptables rule
sudo iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -j MASQUERADE
```

---

## Understanding the NAT Rule in Detail

## The Problem We're Solving

When your namespace tries to reach the internet, here's what happens **without** NAT:

```
[Red Namespace]     [Host/WSL]        [Internet]
192.168.100.10  →   eth0: 172.x.x.x  →  8.8.8.8
                    "Hey Google, this
                     is from 192.168.100.10"

[Google's Response]
8.8.8.8  →  "Where is 192.168.100.10???"
             ❌ DROPPED - Unknown network
```

**The issue**: `192.168.100.10` is a **private IP** that only exists inside your computer. Google has no idea how to send packets back to it!

---

## What NAT Does (Network Address Translation)

NAT **transforms** the packets as they leave your computer:

```
[Red Namespace]     [NAT Magic]       [Internet]
192.168.100.10  →   172.x.x.x        →  8.8.8.8
"Original packet"   "Translated!"       "Sees host IP"

[Google's Response - Works!]
8.8.8.8  →  172.x.x.x  →  [NAT Magic]  →  192.168.100.10
                           "Translates back!"
```

---

## Breaking Down the Command

```bash
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -j MASQUERADE
```

Let's dissect each part:

### `sudo iptables`

- **iptables**: Linux firewall/packet filtering system
- **sudo**: Need admin rights to modify network rules

### `-t nat`

- **-t**: Specifies which "table" to use
- **nat**: The Network Address Translation table
- Other tables: `filter` (firewall), `mangle` (packet modification)

### `-A POSTROUTING`

- **-A**: **A**ppend a new rule
- **POSTROUTING**: Apply rule **after** routing decision is made
- This means: "Just before the packet leaves the computer"

### `-s 192.168.100.0/24`

- **-s**: **S**ource address match
- **192.168.100.0/24**: Any IP from 192.168.100.1 to 192.168.100.254
- This matches packets coming FROM your namespaces

### `-j MASQUERADE`

- **-j**: **J**ump to target (what action to take)
- **MASQUERADE**: Replace source IP with the outgoing interface's IP
- It's "smart" - automatically uses whatever IP your host has

---

## MASQUERADE vs SNAT

There are two types of NAT:

### MASQUERADE (what we're using)

```bash
-j MASQUERADE
```

- **Dynamic**: Automatically uses whatever IP the outgoing interface has
- **Perfect for**: DHCP connections, changing IPs
- **Your case**: WSL IP might change, so this adapts automatically

### SNAT (Static NAT)

```bash
-j SNAT --to-source 172.20.10.5
```

- **Static**: Always use this specific IP
- **Perfect for**: Servers with fixed IPs
- **Problem**: If your WSL IP changes, this breaks

---

## Step-by-Step Packet Journey

Let's trace a packet from your red namespace to Google:

### Step 1: Packet Created

```
Source: 192.168.100.10:54321
Dest:   8.8.8.8:53
```

### Step 2: Routing Decision

```
Host kernel: "8.8.8.8 is external, send via eth0"
```

### Step 3: POSTROUTING (Our NAT Rule Fires!)

```
BEFORE: 192.168.100.10:54321 → 8.8.8.8:53
AFTER:  172.20.10.5:54321    → 8.8.8.8:53
        ↑ Your WSL IP
```

### Step 4: Packet Leaves

```
Internet sees: "Request from 172.20.10.5"
```

### Step 5: Response Comes Back

```
8.8.8.8:53 → 172.20.10.5:54321
```

### Step 6: NAT Translation Back (Automatic!)

```
Kernel remembers: "Port 54321 belongs to 192.168.100.10"
BEFORE: 8.8.8.8:53 → 172.20.10.5:54321
AFTER:  8.8.8.8:53 → 192.168.100.10:54321
```

---

## Why This Works: The Connection Table

The kernel maintains a **connection tracking table**:

```
Internal IP:Port    ↔    External IP:Port    ↔    Remote
192.168.100.10:54321     172.20.10.5:54321       8.8.8.8:53
192.168.100.20:45678     172.20.10.5:45678       1.1.1.1:53
```

This table ensures responses get back to the right namespace!

---

## Visual Analogy: The Post Office

Think of NAT like a **mail forwarding service**:

1. **You write a letter** (namespace creates packet)
   - From: "Apartment 10, Building 100" (192.168.100.10)
   - To: "Google HQ" (8.8.8.8)

2. **Post office rewrites the return address** (NAT translation)
   - From: "123 Main St" (your public IP)
   - To: "Google HQ"
   - **Keeps a note**: "If anything comes back to 123 Main St, forward to Apartment 10"

3. **Google responds** to 123 Main St

4. **Post office forwards** to Apartment 10 using their notes

---

## Testing Your Understanding

After applying the NAT rule, try this experiment:

```bash
# Before NAT rule
sudo ip netns exec red ping 8.8.8.8
# ❌ Fails - no route back

# After NAT rule
sudo ip netns exec red ping 8.8.8.8
# ✅ Works - packets translated!

# Check the connection tracking
sudo cat /proc/net/nf_conntrack | grep 192.168.100
# See the active translations!
```

---

## Common Mistakes

### 1. Wrong table

```bash
# ❌ Wrong - this is firewall table
sudo iptables -A FORWARD -s 192.168.100.0/24 -j ACCEPT

# ✅ Correct - this is NAT table
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -j MASQUERADE
```

### 2. Wrong chain

```bash
# ❌ Wrong - too early in packet processing
sudo iptables -t nat -A PREROUTING -s 192.168.100.0/24 -j MASQUERADE

# ✅ Correct - just before packet leaves
sudo iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -j MASQUERADE
```

### 3. Forgetting IP forwarding

```bash
# NAT rule alone isn't enough!
# You also need:
sudo sysctl net.ipv4.ip_forward=1
```

---

## Real-World Applications

This exact technique is used by:

- **Your home router**: Translates your devices' private IPs (192.168.1.x) to your public IP
- **Docker**: Each container gets internet access this way
- **Kubernetes**: Pod networking uses similar NAT
- **VPNs**: Corporate networks use NAT for internal resources

---

## Quick Verification Commands

```bash
# See your NAT rules
sudo iptables -t nat -L POSTROUTING -v

# See active connections being translated
sudo cat /proc/net/nf_conntrack | head -5

# Test from namespace
sudo ip netns exec red curl -I http://google.com
```

The beauty of MASQUERADE is that it handles all the complexity automatically - you just tell it "translate packets from this network" and it figures out the rest!
