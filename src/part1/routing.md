# Routing

Every time an application sends data across a network, Linux must answer one simple question:

> **Where should this packet go?**

The answer is determined by **routing**.

Routing is the process of deciding where a packet should be sent next. Every packet that leaves your computer goes through this process, whether it's destined for another machine on your local network or a server on the other side of the Internet.

Before we can understand how routing works, we first need to understand what a **network** is and how a computer determines whether another IP address is local or remote.

---

## IP Addresses, Networks, and Hosts

An IPv4 address consists of **32 bits**.

For example:

```text
192.168.1.10
```

By itself, this address is incomplete.

A computer also needs to know **which part of the address identifies the network** and **which part identifies the host**.

For example:

```text
192.168.1.10/24
```

The `/24` tells us that:

* The first **24 bits** identify the **network**
* The remaining **8 bits** identify the **host**

Visually:

```text
192.168.1.10/24

11000000.10101000.00000001.00001010

|-------- Network --------|-- Host --|
       24 bits            8 bits
```

The network portion identifies the subnet.

The host portion identifies an individual device inside that subnet.

---

### Netmask (Subnet Mask)

Before CIDR notation became common, networks were represented using a **subnet mask** (or **netmask**).

For example:

```text
IP Address
192.168.1.10
11000000.10101000.00000001.00001010

Subnet Mask
255.255.255.0
11111111.11111111.11111111.00000000


|-------- Network --------|-- Host --|
       num of 1 bits     
```

Today, the same network is usually written as:

```text
192.168.1.10/24
```

These two notations mean exactly the same thing.

| CIDR | Netmask         |
| ---- | --------------- |
| /8   | 255.0.0.0       |
| /16  | 255.255.0.0     |
| /24  | 255.255.255.0   |
| /25  | 255.255.255.128 |
| /26  | 255.255.255.192 |
| /30  | 255.255.255.252 |

Linux accepts both formats, although CIDR notation is used almost everywhere today.

---

### How Does a Netmask Work?

A netmask is simply another 32-bit value.

Every `1` bit belongs to the network.

Every `0` bit belongs to the host.

For example:

```text
IP Address

192.168.1.10

11000000.10101000.00000001.00001010


Subnet Mask

255.255.255.0

11111111.11111111.11111111.00000000
```

The mask separates the address into two parts:

```text
Network
192.168.1

Host
10
```

The computer performs a **bitwise AND** between the IP address and the subnet mask to determine the network address.

Fortunately, you almost never need to perform this calculation manually. Understanding what the mask represents is much more important than memorizing the binary arithmetic.

---

### Network Address

The network address identifies the subnet itself.

Suppose a computer has:

```text
192.168.1.42/24
```

The network address is:

```text
192.168.1.0
```

because the host bits become zero.

Likewise:

| IP Address       | Network        |
| ---------------- | -------------- |
| 192.168.1.10/24  | 192.168.1.0/24 |
| 192.168.1.200/24 | 192.168.1.0/24 |
| 10.20.30.15/16   | 10.20.0.0/16   |
| 172.16.5.100/20  | 172.16.0.0/20  |

Notice that many different hosts can belong to the same network.

---

### What Does "Same Network" Mean?

Consider a computer configured as:

```text
IP Address
192.168.1.10/24
```

Now suppose it wants to communicate with the following destinations.

| Destination   | Same Network? |
| ------------- | ------------- |
| 192.168.1.20  | ✅ Yes         |
| 192.168.1.200 | ✅ Yes         |
| 192.168.2.15  | ❌ No          |
| 10.0.0.5      | ❌ No          |

The first two destinations belong to the same subnet:

```text
192.168.1.0/24
```

Linux knows these devices are directly reachable.

The last two belong to different networks.

Linux knows it cannot reach them directly.

Instead, it must send the packets to a router.

---

### Directly Connected Networks

Every network interface defines a directly connected network.

For example:

```text
eth0

192.168.1.10/24
```

means:

> Everything inside **192.168.1.0/24** is directly reachable through **eth0**.

Linux does not need another router to reach these hosts.

It can communicate with them directly.

---

## What Is Routing?

Routing begins only after Linux determines that the destination is **not** on one of its directly connected networks.

Suppose our computer wants to reach:

```text
8.8.8.8
```

Since this address is not part of:

```text
192.168.1.0/24
```

Linux must ask:

> **Which router should receive this packet?**

The answer comes from the routing table.

---

## The Routing Table

Linux stores its routing information in a routing table.

You can display it with:

```bash
ip route
```

Example:

```text
default via 192.168.1.1 dev eth0

192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.10

10.10.0.0/16 via 192.168.1.254 dev eth0
```

Each entry tells Linux how to reach a particular network.

Notice something important:

The routing table contains **networks**, not individual hosts.

Routing almost always happens at the network level.

---

## Reading a Route

Let's examine one entry:

```text
10.10.0.0/16 via 192.168.1.254 dev eth0
```

This means:

* To reach the network `10.10.0.0/16`
* Send the packet to `192.168.1.254`
* Use the interface `eth0`

The address `192.168.1.254` is called the **next hop** or **gateway**.

It is the next device responsible for forwarding the packet.

---

## The Default Route

Eventually you'll encounter this route:

```text
default via 192.168.1.1
```

or equivalently:

```text
0.0.0.0/0
```

This is the **default route**.

It simply means:

> If no other route matches the destination, send the packet to this gateway.

Almost every computer connected to the Internet has a default route.

Without one, the computer could communicate only with devices on its directly connected networks.

---

## Longest Prefix Match

Sometimes multiple routes match the same destination.

For example:

```text
10.0.0.0/8

10.10.0.0/16

10.10.10.0/24
```

Now suppose the destination is:

```text
10.10.10.42
```

All three routes match.

Linux chooses the **most specific** route.

This rule is called the **Longest Prefix Match**.

Since `/24` is more specific than `/16`, and `/16` is more specific than `/8`, Linux selects:

```text
10.10.10.0/24
```

This rule is fundamental to IP routing.

---

## Putting It All Together

Whenever Linux sends a packet, it follows roughly this process:

```text
Application
        │
        ▼
Destination IP
        │
        ▼
Is the destination on one of my directly connected networks?
        │
   Yes ─┴─ No
    │        │
    ▼        ▼
Send      Search the routing table
Directly       │
               ▼
      Select the best matching route
               │
               ▼
        Determine the next hop
               │
               ▼
        Send packet to the gateway
```

At this point, Linux knows **where** the packet should go.

However, it still has another problem.

It knows the **IP address** of the next hop, but Ethernet doesn't send frames to IP addresses—it sends them to **MAC addresses**.

So how does Linux discover the MAC address of the next hop?

That's the job of the **Address Resolution Protocol (ARP)**, which we'll explore in the next chapter.
