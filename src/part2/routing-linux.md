# How Linux Makes Routing Decisions

In Part 1, we learned the concepts behind IP routing: networks, routing tables, longest prefix match, and the default gateway.

In this chapter, we'll look at how the Linux kernel actually makes routing decisions. We'll inspect the routing table, understand how routes are selected, and learn the tools used to troubleshoot routing problems.

By the end of this chapter, you should be able to answer one of the most common networking questions:

> **Why did Linux send this packet through that interface?**

---

## The Linux Routing Table

The Linux kernel maintains one or more routing tables that describe how packets should be forwarded.

Each route represents a destination network and tells the kernel how to reach it.

Display the routing table:

```bash
ip route
```

Example output:

```text
default via 192.168.1.1 dev ens18 proto dhcp metric 100

192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.10

10.10.0.0/16 via 192.168.1.254 dev ens18
```

Each line is a route that the kernel may use when forwarding packets.

---

## Understanding a Route

Consider the following route:

```text
10.10.0.0/16 via 192.168.1.254 dev ens18
```

It can be read as:

| Field               | Meaning             |
| ------------------- | ------------------- |
| `10.10.0.0/16`      | Destination network |
| `via 192.168.1.254` | Next-hop gateway    |
| `dev ens18`         | Outgoing interface  |

In other words:

> To reach the `10.10.0.0/16` network, send packets through interface `ens18` to the router at `192.168.1.254`.

Not every route contains a gateway.

For example:

```text
192.168.1.0/24 dev ens18
```

This is a **directly connected route**.

Linux already knows that every host in this network is reachable through `ens18`, so no intermediate router is required.

---

## Where Do Routes Come From?

Routes can be installed in several ways.

### Automatically

When an interface receives an IP address, Linux automatically creates a route for the directly connected network.

For example:

```text
192.168.1.10/24
```

automatically creates:

```text
192.168.1.0/24 dev ens18
```

---

### By DHCP

DHCP servers commonly install a default gateway.

For example:

```text
default via 192.168.1.1
```

This allows the system to communicate with networks outside the local subnet.

---

### Statically

Administrators can manually add routes.

Example:

```bash
sudo ip route add 10.20.0.0/16 via 192.168.1.254
```

Delete it:

```bash
sudo ip route del 10.20.0.0/16
```

Unless persisted through your network management software, these changes are temporary and disappear after a reboot.

---

## Longest Prefix Match

One of the most important routing rules is **Longest Prefix Match**.

Suppose the routing table contains:

```text
10.0.0.0/8
10.10.0.0/16
10.10.10.0/24
```

Now the destination is:

```text
10.10.10.42
```

All three routes match.

Linux chooses the most specific route:

```text
10.10.10.0/24
```

The kernel always prefers the route with the longest matching network prefix.

This behavior is independent of the order in which the routes appear in the routing table.

---

## Route Metrics

Sometimes multiple routes have exactly the same destination prefix.

For example:

```text
default via 192.168.1.1 dev ens18 metric 100

default via 10.0.0.1 dev ens19 metric 200
```

Since both routes are equally specific, Linux compares their **metrics**.

A **lower metric** is preferred.

In this example:

```text
metric 100
```

is selected over:

```text
metric 200
```

Metrics are commonly assigned automatically by DHCP clients, NetworkManager, or systemd-networkd, but they can also be configured manually.

---

## Route Scope

Some routes include a scope.

Example:

```text
192.168.1.0/24 dev ens18 scope link
```

The scope describes where a route is considered valid.

The most common values are:

| Scope    | Meaning                              |
| -------- | ------------------------------------ |
| `host`   | Valid only for the local system      |
| `link`   | Reachable directly on the local link |
| `global` | Reachable through routing            |

Most administrators rarely configure scope manually, but understanding it helps when reading routing tables.

---

## Finding the Selected Route

One of the most useful commands in Linux networking is:

```bash
ip route get <destination>
```

For example:

```bash
ip route get 8.8.8.8
```

Example output:

```text
8.8.8.8 via 192.168.1.1 dev ens18 src 192.168.1.10
```

This command shows exactly how Linux would route traffic to the specified destination.

It answers several important questions:

* Which interface will be used?
* Which gateway will receive the packet?
* Which source IP address will be chosen?

When troubleshooting routing issues, this command is often more useful than simply viewing the routing table.

---

## Source Address Selection

A Linux system may have multiple IP addresses.

For example:

```text
ens18
    192.168.1.10/24
    192.168.1.20/24
```

or even multiple interfaces:

```text
ens18
    192.168.1.10

ens19
    10.10.0.10
```

When sending a packet, Linux must also choose the **source IP address**.

The selected source address usually belongs to the outgoing interface and may be influenced by the route itself.

You can see the selected source address with:

```bash
ip route get 8.8.8.8
```

The output includes:

```text
8.8.8.8 via 192.168.1.1 dev ens18 src 192.168.1.10
```

This is the address remote systems will see as the sender.

We'll revisit source address selection later when discussing policy routing and VRFs.

---

## The Local Routing Table

Linux maintains a special routing table for addresses that belong to the local system.

Display it with:

```bash
ip route show table local
```

Example:

```text
local 192.168.1.10 dev lo
broadcast 192.168.1.255 dev ens18
```

These routes are managed automatically by the kernel and ensure that packets destined for local addresses are delivered internally instead of being transmitted on the network.

---

## Monitoring Routing Changes

To observe routing updates in real time:

```bash
ip monitor route
```

Now make a routing change or disconnect a network interface.

You'll immediately see updates as the kernel modifies its routing tables.

This command is particularly useful when troubleshooting DHCP, VPNs, or dynamic routing changes.

---

## Common Troubleshooting Commands

The following commands should become part of your daily toolkit.

Display all routes:

```bash
ip route
```

Display detailed interface addresses:

```bash
ip addr
```

Determine how Linux would reach a destination:

```bash
ip route get 1.1.1.1
```

Show the local routing table:

```bash
ip route show table local
```

Monitor routing changes:

```bash
ip monitor route
```

These commands answer the majority of routing questions encountered on Linux systems.

---

## Looking Ahead

By the end of this chapter, Linux has determined:

* The destination network.
* The best matching route.
* The outgoing interface.
* The next-hop gateway.
* The source IP address.

However, one question still remains before the packet can leave the machine:

> **How does Linux actually transmit the packet through the selected interface?**

The answer lies in the **neighbor table**, where Linux resolves IP addresses to MAC addresses before constructing the Ethernet frame. In the next chapter, we'll begin observing packets on a running Linux system and watch this process happen in real time.
