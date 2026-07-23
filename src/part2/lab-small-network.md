# Build a Small Linux Network

Throughout Part 2, we've explored the fundamental building blocks of Linux networking.

We've learned how to inspect network interfaces, understand routing decisions, observe packets, and inspect network connections.

Now it's time to put everything together.

In this chapter, we'll build a small Linux network using virtual machines (or physical hosts) and use the tools from previous chapters to observe every step of the communication process.

The goal is not simply to make the network work, but to understand **how Linux behaves** while it is working.

---

## The Network Topology

We'll build a simple network consisting of two Linux hosts connected through a router.

```text
                  10.0.1.0/24
+---------+      +---------+      +---------+
| Host A  |------| Router  |------| Host B  |
|10.0.1.10|      |         |      |10.0.2.10|
+---------+      |         |      +---------+
                 |         |
                 |         |
                 +---------+
                10.0.2.0/24
```

This topology is intentionally simple.

It allows us to observe nearly every networking concept introduced in Part 2.

---

## What You'll Need

You can complete this chapter using:

* A virtualization platform such as VMware, VirtualBox, KVM, or Hyper-V
* Three virtual machines

The Linux distribution is not important.

The examples use the `iproute2` tools available on virtually every modern Linux distribution.

---

## Configure the Network

Assign the following addresses.

| Host   | Interface | Address      |
| ------ | --------- | ------------ |
| Host A | eth0      | 10.0.1.10/24 |
| Router | eth0      | 10.0.1.1/24  |
| Router | eth1      | 10.0.2.1/24  |
| Host B | eth0      | 10.0.2.10/24 |

Configure the addresses:

```bash
sudo ip addr add 10.0.1.10/24 dev eth0
```

Adjust the commands for each system as needed.

Verify:

```bash
ip addr
```

---

## Configure the Default Gateway

Host A:

```bash
sudo ip route add default via 10.0.1.1
```

Host B:

```bash
sudo ip route add default via 10.0.2.1
```

Verify:

```bash
ip route
```

The output should include a default route pointing to the router.

---

## Enable Packet Forwarding

By default, Linux does not forward packets between interfaces.

Enable forwarding on the router:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Verify:

```bash
sysctl net.ipv4.ip_forward
```

Expected output:

```text
net.ipv4.ip_forward = 1
```

At this point, the router is capable of forwarding packets between the two networks.

---

## Verify Local Connectivity

Before testing routing, verify that each host can reach its local gateway.

From Host A:

```bash
ping 10.0.1.1
```

From Host B:

```bash
ping 10.0.2.1
```

If these tests fail, troubleshoot the local network before proceeding.

---

## Verify End-to-End Connectivity

Now test communication across both networks.

From Host A:

```bash
ping 10.0.2.10
```

If everything is configured correctly, the ping should succeed.

At this point, Linux has performed every operation we've discussed so far:

* Selected a route.
* Determined the outgoing interface.
* Resolved the router's MAC address.
* Generated ICMP packets.
* Forwarded packets through the router.
* Delivered the reply back to the source.

---

## Observe the Routing Decision

Before sending traffic, ask Linux how it plans to reach Host B.

On Host A:

```bash
ip route get 10.0.2.10
```

Example:

```text
10.0.2.10 via 10.0.1.1 dev eth0 src 10.0.1.10
```

Notice that Linux does **not** attempt to reach Host B directly.

Instead, it sends packets to the router.

---

## Observe the Neighbor Table

Before the first ping:

```bash
ip neigh
```

You may see no entry for the router.

Now execute:

```bash
ping 10.0.2.10
```

Check again:

```bash
ip neigh
```

You'll now find an entry similar to:

```text
10.0.1.1 dev eth0 lladdr 52:54:00:aa:bb:cc REACHABLE
```

Notice that the neighbor table contains the MAC address of the **router**, not Host B.

This reinforces an important concept from Part 1:

Ethernet frames are always addressed to the **next hop**, not necessarily the final destination.

---

## Observe the Packets

Start a packet capture on Host A.

```bash
sudo tcpdump -n -i eth0
```

Run another ping.

You'll observe:

* ARP (if necessary)
* ICMP Echo Request
* ICMP Echo Reply

Repeat the capture on the router.

Notice that the router receives the packet on one interface and forwards it through another.

Finally, capture traffic on Host B.

Watching the same packet traverse multiple systems is one of the best ways to understand packet forwarding.

---

## Observe Interface Statistics

Before generating traffic:

```bash
ip -s link
```

Generate several pings.

Run the command again.

Notice how:

* RX counters increase.
* TX counters increase.

On the router, both interfaces should show activity because packets arrive on one interface and leave through the other.

---

## Observe Connections

Although `ping` uses ICMP instead of TCP, we can still generate TCP traffic.

Start a simple web server on Host B.

```bash
python3 -m http.server 8080
```

From Host A:

```bash
curl http://10.0.2.10:8080
```

Inspect the socket:

```bash
ss -tn
```

Observe:

* Listening socket
* Established connection
* Local and remote addresses

Everything we've learned about sockets now becomes visible.

---

## Trace the Complete Packet Journey

Let's summarize what happened when Host A connected to Host B.

```text
Application
      │
      ▼
Socket
      │
      ▼
Routing Table Lookup
      │
      ▼
Neighbor Table Lookup
      │
      ▼
Ethernet Frame Created
      │
      ▼
Outgoing Interface
      │
      ▼
Router
      │
      ▼
Forwarding Decision
      │
      ▼
Host B
      │
      ▼
Application
```

This is the complete path that every packet follows through the Linux networking stack at a high level.

The remaining chapters of this book will examine each stage in greater detail.

---

## Troubleshooting Checklist

If connectivity fails, work through the following checklist.

1. Is the interface up?

```bash
ip link
```

2. Does the interface have the correct IP address?

```bash
ip addr
```

3. Is the routing table correct?

```bash
ip route
```

4. Does the neighbor table contain the expected MAC address?

```bash
ip neigh
```

5. Are packets reaching the interface?

```bash
tcpdump -n -i eth0
```

6. Are interface counters increasing?

```bash
ip -s link
```

7. Is packet forwarding enabled on the router?

```bash
sysctl net.ipv4.ip_forward
```

This workflow should become second nature. It follows the same methodology introduced throughout Part 2 and applies equally well to simple laboratory environments and production systems.

---

## Summary

In this chapter, we combined the concepts from Part 2 into a working Linux network.

You observed how Linux:

* Configures network interfaces.
* Chooses routes.
* Resolves neighbors using ARP.
* Generates and captures packets.
* Forwards traffic through a router.
* Tracks interface statistics.
* Creates and manages network connections.

At this point, you have a solid understanding of Linux networking on a single network and across multiple subnets.

In Part 3, we'll begin replacing physical cables and switches with virtual networking components. You'll learn how Linux builds entirely virtual networks using namespaces, virtual Ethernet devices, bridges, and other kernel networking primitives—the same technologies that underpin containers, virtualization platforms, and many cloud networking solutions.
