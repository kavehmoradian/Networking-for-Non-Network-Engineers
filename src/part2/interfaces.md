# Linux Network Interfaces

In Part 1, we learned how computers communicate using IP addresses, routing, ARP, TCP, and DNS.

Now it's time to see how Linux implements these concepts.

Everything begins with the **network interface**.

Whether you're configuring a physical Ethernet card, creating a bridge, attaching a container to a virtual network, or debugging Kubernetes networking, you're ultimately working with Linux network interfaces.

Understanding interfaces is one of the most important skills in Linux networking because almost every networking operation starts from one.

---

## What Is a Network Interface?

A **network interface** is Linux's representation of a network connection.

From the kernel's perspective, every packet enters and leaves the system through a network interface.

An interface might represent:

* A physical Ethernet card
* A Wi-Fi adapter
* A virtual Ethernet device
* A bridge
* A VLAN
* A tunnel
* A loopback device

Linux treats all of these as network interfaces.

This unified design is one of the reasons Linux networking is so powerful.

---

## Listing Network Interfaces

The modern way to list interfaces is with the `ip` command.

```bash
ip link
```

Example output:

```text
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT
```

A simpler view:

```bash
ip -br link
```

Example:

```text
lo               UNKNOWN        00:00:00:00:00:00
ens18            UP             52:54:00:12:34:56
```

The `-br` (brief) option is often easier to read, especially on systems with many interfaces.

---

## Interface Names

Older Linux systems used names like:

```text
eth0
eth1
wlan0
```

Modern distributions typically use **predictable interface names**, such as:

```text
ens18
ens160
enp3s0
eno1
```

These names are generated based on hardware location, making them more stable across reboots.

You'll still encounter `eth0` in:

* Containers
* Virtual machines
* Documentation
* Older systems

Throughout this book, we'll use whatever interface names exist on the system.

---

## Viewing Interface Details

To inspect an interface:

```bash
ip addr show ens18
```

Example:

```text
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.10/24
    link/ether 52:54:00:12:34:56
```

Notice that one interface contains several pieces of information.

* Interface name
* Administrative state
* MAC address
* IP address
* Prefix length

Unlike many networking devices, Linux stores all of these properties as part of the interface itself.

---

## Administrative State
Every interface has an administrative states. Whether the interface has been enabled.

View:

```bash
ip link
```

Bring an interface up:

```bash
sudo ip link set ens18 up
```

Bring it down:

```bash
sudo ip link set ens18 down
```

An interface that is administratively down cannot send or receive traffic.

---

## The Loopback Interface

Every Linux system has an interface called:

```text
lo
```

View it:

```bash
ip addr show lo
```

Example:

```text
1: lo
inet 127.0.0.1/8
```

The loopback interface never leaves the computer.

Traffic sent to:

```text
127.0.0.1
```

never reaches a physical network.

Instead, it is delivered directly back into the kernel.

Many applications use the loopback interface to communicate with services running on the same machine.

---

## MAC Addresses

Every Ethernet interface has a **MAC address**.

View it:

```bash
ip link show ens18
```

Example:

```text
link/ether 52:54:00:12:34:56
```

Remember from Part 1:

* IP addresses identify hosts on a network.
* MAC addresses identify interfaces on a local Ethernet network.

When Linux sends an Ethernet frame, it places the destination MAC address in the frame header.

---

## IP Addresses

Interfaces can have one or more IP addresses.

Show only IPv4 addresses:

```bash
ip -4 addr
```

Show only IPv6:

```bash
ip -6 addr
```

Assign a temporary address:

```bash
sudo ip addr add 192.168.100.10/24 dev ens18
```

Remove it:

```bash
sudo ip addr del 192.168.100.10/24 dev ens18
```

These changes affect the running system only.

After a reboot, persistent configuration depends on your network management software, such as NetworkManager or systemd-networkd.

---

## Interface Statistics

Linux tracks packet counters for every interface.

Display them:

```bash
ip -s link
```

Example:

```text
RX:
packets  bytes

TX:
packets  bytes
```

These counters are invaluable when troubleshooting.

They can help answer questions such as:

* Is traffic reaching the server?
* Is the interface transmitting packets?
* Are packets being dropped?
* Are errors increasing?

---

## Every Packet Has an Interface

One of the most important concepts in Linux networking is this:

> Every packet enters through one interface and leaves through another.

Whether the interface represents:

* A physical NIC
* A bridge
* A veth pair
* A VLAN
* A VXLAN tunnel

the Linux kernel processes them in the same way.

This consistent abstraction is what makes Linux networking both flexible and powerful.

In the following chapters, we'll create new interfaces ourselves and use them to build entirely virtual networks.

---

## Inspecting Interface Capabilities with ethtool

While the `ip` command manages network interfaces and their addresses, it does not expose hardware-specific information.

For that, Linux provides the **`ethtool`** utility.

`ethtool` allows you to inspect and configure properties of network interfaces, such as:

* Link speed
* Driver information
* Supported features
* Offloading capabilities
* Ring buffer sizes
* Interface statistics

When troubleshooting network performance, `ethtool` is often one of the first tools you'll use.

---

### Viewing Interface Information

Display basic information about an interface:

```bash
sudo ethtool ens18
```

Example output:

```text
Settings for ens18:
    Speed: 1000Mb/s
    Auto-negotiation: on
    Port: Twisted Pair
    Link detected: yes
```

Some important fields are:

| Field            | Description                                          |
| ---------------- | ---------------------------------------------------- |
| Speed            | Current link speed                                   |
| Auto-negotiation | Whether link parameters are negotiated automatically |
| Link detected    | Whether the physical link is active                  |

If **Link detected** is `no`, the interface is not connected to another device, regardless of whether it has an IP address configured.

---

### Viewing Interface Statistics

Many network drivers expose additional statistics beyond the standard packet counters shown by `ip`.

Display them with:

```bash
sudo ethtool -S ens18
```

Depending on the hardware, you may see counters for:

* CRC errors
* Dropped packets
* Missed packets
* Queue statistics
* Interrupt counts
* Driver-specific metrics

These statistics are invaluable when diagnosing hardware or driver problems.

---

### Changing Interface Settings

`ethtool` can also modify interface settings.

For example, to disable auto-negotiation and set a fixed speed:

```bash
sudo ethtool -s ens18 speed 1000 duplex full autoneg off
```

Changing these settings is uncommon on modern systems because interfaces usually negotiate the optimal parameters automatically.

Unless you have a specific operational requirement, it's generally best to leave these settings unchanged.

---

### When Should You Use ethtool?

Use `ethtool` when you need to answer questions such as:

* Is the Ethernet cable connected?
* What speed did the interface negotiate?
* Are there hardware-level errors or dropped packets?
* Is this a physical NIC or a virtual one?

While `ip` tells you **how Linux sees an interface**, `ethtool` tells you **how the interface itself is operating**.

---

## Summary

In this chapter, you learned that:

* Linux represents every network connection as a network interface.
* Interfaces can be physical or virtual.
* Every interface has a name, state, MAC address, and optionally one or more IP addresses.
* `ip` and `ethtool` commands are used for inspecting and managing interfaces.
* Every packet entering or leaving the system passes through a network interface.

With this foundation in place, we're ready to explore how Linux decides which interface should be used when sending a packet by examining the system's routing tables.
