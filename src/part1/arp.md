# ARP: Finding the Next Hop

In the previous chapter, we learned that applications communicate using IP addresses and ports. But this raises an important question:

> **If I know the destination IP address, how do I actually send data to it?**

The answer is that you **can't**—at least not directly.

On an Ethernet network, computers do not send data to IP addresses. They send **Ethernet frames** to **MAC addresses**.

So before Linux can transmit a packet, it must first answer another question:

> **Which MAC address belongs to the next IP address?**

This is the job of the **Address Resolution Protocol (ARP).**

---

## IP Addresses vs. MAC Addresses

Every network interface has two important addresses.

An **IP address** identifies a device on a network and is used for routing packets between networks.

A **MAC address** identifies a network interface on the local network and is used to deliver Ethernet frames.

For example:

| Type        | Example             |
| ----------- | ------------------- |
| IP Address  | `192.168.1.10`      |
| MAC Address | `52:54:00:12:34:56` |

Think of an IP address as a person's postal address, while the MAC address is the label on their mailbox. The postal system uses the address to find the correct house, but once the delivery truck arrives, it still needs the correct mailbox.

---

## The Problem

Suppose your computer wants to send a packet to:

```text
192.168.1.20
```

Your routing table says that this destination is on the same local network.

Linux now knows *where* the packet should go, but it still doesn't know **how to deliver the Ethernet frame**.

It needs the destination MAC address.

At this point, Linux checks whether it already knows the answer.

---

## The ARP Cache

Linux keeps a small table of recently discovered IP-to-MAC mappings.

This table is called the **ARP cache**, or more generally in Linux, the **neighbor table**.

You can view it with:

```bash
ip neigh
```

Example:

```text
192.168.1.20 dev eth0 lladdr 00:11:22:33:44:55 REACHABLE
192.168.1.1  dev eth0 lladdr aa:bb:cc:dd:ee:ff STALE
```

If Linux finds the destination in this table, it immediately sends the Ethernet frame to the corresponding MAC address.

If not, it must discover it.

---

## ARP Request

When Linux doesn't know the destination MAC address, it sends an **ARP Request**.

In simple terms, the request says:

> "Who has IP address 192.168.1.20? Please tell me your MAC address."

Because Linux doesn't yet know where that device is, the request is sent as a **broadcast**.

Every device on the local Ethernet network receives the request.

---

## ARP Reply

Every computer checks the requested IP address.

Only the computer that owns the address replies.

The reply says something like:

> "192.168.1.20 is at 00:11:22:33:44:55."

Linux stores this mapping in its neighbor table for future use.

Now it can finally construct an Ethernet frame and send the packet.

---

## Putting It Together

Suppose you run:

```bash
ping 192.168.1.20
```

If the MAC address is unknown, the sequence looks like this:

```text
Application
      │
      ▼
Create ICMP packet
      │
      ▼
Routing decides:
Destination is on the local network
      │
      ▼
Check neighbor table
      │
      ▼
No entry found
      │
      ▼
Broadcast ARP Request
      │
      ▼
Receive ARP Reply
      │
      ▼
Store IP → MAC mapping
      │
      ▼
Transmit Ethernet frame
```

Notice that the ICMP packet isn't sent until ARP has completed successfully.

---

## What If the Destination Is on Another Network?

Suppose your computer wants to reach:

```text
8.8.8.8
```

Linux knows this address is **not** on the local network.

Instead of looking for the MAC address of `8.8.8.8`, Linux sends the packet to its **default gateway**.

That means ARP resolves the MAC address of the **router**, not the final destination.

For example:

```text
Destination IP:
8.8.8.8

Next Hop:
192.168.1.1

MAC Needed:
Router's MAC address
```

This is why the chapter is called **"Finding the Next Hop."**

ARP never tries to find the MAC address of a device on another network. It always resolves the MAC address of the **next device that will receive the Ethernet frame**.

---

## Key Takeaways

Remember these four ideas:

* IP addresses identify where packets should go.
* MAC addresses identify where Ethernet frames should be delivered.
* ARP translates an IP address into a MAC address on the local network.
* ARP always resolves the **next hop**, not necessarily the final destination.

Understanding ARP explains many networking behaviors that initially seem mysterious. It also prepares us for the next chapter, where we'll explore **routing** and learn how Linux decides what the next hop actually is.
