# Observing Network Traffic on Linux

In the previous chapter, we learned how Linux decides where a packet should go by consulting its routing table.

But routing decisions are only part of the story.

To truly understand Linux networking, you need to observe packets as they move through the system.

One of the most valuable networking skills is the ability to observe what is actually happening on the wire.

When diagnosing networking problems, experienced engineers often follow a simple principle:

> **Don't guess. Capture the traffic.**

In this chapter, we'll learn how to observe network traffic on Linux using industry-standard tools and develop a systematic approach to troubleshooting network issues.

---

## The Linux Networking Toolbox

Linux provides several tools for observing different parts of the networking stack.

| Tool         | Purpose                          |
| ------------ | -------------------------------- |
| `ip link`    | Inspect network interfaces       |
| `ip -s link` | View interface statistics        |
| `ip neigh`   | Inspect the neighbor (ARP) table |
| `tcpdump`    | Capture packets                  |
| `ping`       | Generate ICMP traffic            |
| `curl`       | Generate TCP/HTTP traffic        |

Rather than using these tools independently, we'll combine them to understand exactly what Linux is doing.

---

## Packet Capture

Every packet that enters or leaves a Linux network interface can be captured and inspected.

Packet capturing allows you to observe:

* Which hosts are communicating
* Which protocols are in use
* Source and destination IP addresses
* TCP and UDP ports
* DNS queries
* HTTP requests
* TLS handshakes
* ICMP messages
* ARP requests and replies

Instead of relying on assumptions, you can examine the packets themselves.

---

## tcpdump

The most widely used packet capture tool on Linux is **tcpdump**.

Although graphical tools like Wireshark are excellent for packet analysis, `tcpdump` is available on nearly every Linux server and is the first tool most administrators reach for when troubleshooting production systems.

---

## Listing Available Interfaces

Before capturing traffic, identify the available interfaces.

```bash
tcpdump -D
```

Example:

```text
1.eth0
2.lo
3.any
```

You can also use:

```bash
ip -br link
```

to identify interface names.

---

## Capturing Traffic

Capture packets on an interface:

```bash
sudo tcpdump -i ens18
```

Immediately you'll begin seeing packets similar to:

```text
14:35:12 IP 192.168.1.10.51234 > 8.8.8.8.53: UDP
14:35:12 IP 8.8.8.8.53 > 192.168.1.10.51234: UDP
```

Each line represents a packet captured from the network.

By default, `tcpdump` resolves hostnames and service names, which can make the output harder to read.

For troubleshooting, it's usually better to disable name resolution.

```bash
sudo tcpdump -n -i ens18
```

The `-n` option tells `tcpdump` to display numeric IP addresses instead of performing DNS lookups.

---

## Capturing on All Interfaces

Sometimes you're not sure which interface is carrying the traffic.

Linux provides a special pseudo-interface called `any`.

```bash
sudo tcpdump -i any
```

This captures traffic from every interface on the system.

It's particularly useful on servers with multiple NICs or virtual networking devices.

---

## Limiting the Number of Packets

Capturing traffic indefinitely isn't always practical.

Capture only ten packets:

```bash
sudo tcpdump -c 10 -i ens18
```

After ten packets, `tcpdump` exits automatically.

---

## Packet Capture Filters

Without filters, a busy server may generate thousands of packets every second.

Fortunately, `tcpdump` supports powerful capture filters.

Capture only ICMP:

```bash
sudo tcpdump -i ens18 icmp
```

Only ARP:

```bash
sudo tcpdump -i ens18 arp
```

Only DNS:

```bash
sudo tcpdump -i ens18 port 53
```

Only SSH:

```bash
sudo tcpdump -i ens18 port 22
```

Only traffic to a specific host:

```bash
sudo tcpdump -i ens18 host 192.168.1.20
```

Only incoming traffic:

```bash
sudo tcpdump -i ens18 dst host 192.168.1.20
```

Only outgoing traffic:

```bash
sudo tcpdump -i ens18 src host 192.168.1.20
```

Capture only TCP:

```bash
sudo tcpdump -i ens18 tcp
```

Capture only UDP:

```bash
sudo tcpdump -i ens18 udp
```

Filters can also be combined.

```bash
sudo tcpdump -i ens18 tcp port 443
```

Learning a handful of filters dramatically improves the usefulness of packet captures.

---

## Reading Packet Summaries

Consider this packet:

```text
192.168.1.10.45132 > 8.8.8.8.53: UDP
```

Reading from left to right:

| Field          | Meaning            |
| -------------- | ------------------ |
| `192.168.1.10` | Source IP          |
| `45132`        | Source port        |
| `8.8.8.8`      | Destination IP     |
| `53`           | Destination port   |
| `UDP`          | Transport protocol |

Understanding this summary is often enough to determine whether traffic is flowing correctly.

---

## Capturing Packet Contents

To inspect packet payloads:

```bash
sudo tcpdump -X -i ens18
```

or

```bash
sudo tcpdump -XX -i ens18
```

These options display both hexadecimal and ASCII representations of the packet.

This is useful when examining protocols such as HTTP or DNS.

Keep in mind that encrypted protocols such as HTTPS or SSH will not display readable application data.

---

## The Neighbor Table

In Part 1, we learned that ARP translates IP addresses into MAC addresses.

Linux stores these mappings in the **neighbor table**.

Display it:

```bash
ip neigh
```

Example:

```text
192.168.1.1 dev ens18 lladdr 52:54:00:aa:bb:cc REACHABLE
192.168.1.20 dev ens18 lladdr 52:54:00:11:22:33 STALE
```

Each entry associates an IP address with a MAC address on the local network.

Before transmitting an Ethernet frame, Linux checks this table.

If the required entry already exists, Linux immediately knows the destination MAC address.

Otherwise, it must perform ARP.

---

## Watching ARP in Action

Let's observe this process.

First, clear the neighbor table.

```bash
sudo ip neigh flush all
```

Verify that it is empty.

```bash
ip neigh
```

Now start a packet capture:

```bash
sudo tcpdump -n -i ens18 arp
```

In another terminal, run:

```bash
ping 192.168.1.1
```

You should observe packets similar to:

```text
ARP, Request who-has 192.168.1.1 tell 192.168.1.10

ARP, Reply 192.168.1.1 is-at 52:54:00:aa:bb:cc
```

Stop the capture and inspect the neighbor table again.

```bash
ip neigh
```

Example:

```text
192.168.1.1 dev ens18 lladdr 52:54:00:aa:bb:cc REACHABLE
```

Notice the relationship:

1. The neighbor table was empty.
2. Linux transmitted an ARP Request.
3. The router replied.
4. Linux stored the mapping in the neighbor table.
5. The ICMP Echo Request was finally transmitted.

This is exactly the ARP process we studied in Part 1, now observed on a live Linux system.

---

## Observing a DNS Lookup

Let's combine several tools.

Start a packet capture:

```bash
sudo tcpdump -n -i ens18 port 53
```

In another terminal:

```bash
dig example.com
```

You'll observe DNS queries and replies.

---

## Observing a Web Request

Now let's observe a complete TCP connection.

Start:

```bash
sudo tcpdump -n -i ens18 tcp port 443
```

Generate traffic:

```bash
curl https://example.com
```

You'll observe:

* TCP three-way handshake
* TLS handshake packets
* Application data
* Connection teardown

At this point, you're observing the concepts introduced throughout Part 1 in a real Linux system.

---

## Writing Packet Captures to a File

In production environments, packet captures are often saved for later analysis.

Write packets to a file:

```bash
sudo tcpdump -i ens18 -w capture.pcap
```

Stop the capture with `Ctrl+C`.

The resulting `.pcap` file can later be analyzed using:

* Wireshark
* tcpdump
* tshark

Saving captures allows you to investigate problems without remaining connected to the affected system.

---

## Reading Capture Files

To inspect a previously captured file:

```bash
tcpdump -r capture.pcap
```

All of the filtering and display options can also be applied when reading capture files.

---

## Common Troubleshooting Examples

### Is DNS Working?

```bash
sudo tcpdump -n -i ens18 port 53
```

If you see outgoing DNS queries but no replies, the problem is likely between the client and the DNS server.

---

### Is ARP Working?

```bash
sudo tcpdump -n -i ens18 arp
```

You should observe ARP requests followed by ARP replies.

Repeated ARP requests with no replies usually indicate a Layer 2 connectivity problem.

---

### Is the Server Receiving Requests?

```bash
sudo tcpdump -n -i ens18 tcp port 443
```

If no packets appear, the traffic is probably being blocked or never reaching the server.

---

### Is Traffic Leaving the Server?

```bash
sudo tcpdump -n -i ens18 host 192.168.1.20
```

Watching both requests and responses helps determine whether the problem is inbound, outbound, or both.

---

## Understanding What You're Capturing

One important point is often overlooked.

`tcpdump` captures packets **at the interface where you are listening**.

If you capture on:

```text
lo
```

you'll see loopback traffic.

If you capture on:

```text
ens18
```

you'll see packets entering and leaving that physical interface.

Later in this book, we'll capture traffic on bridges, veth interfaces, network namespaces, and VXLAN devices.

The same packet may appear on multiple interfaces as it moves through the Linux networking stack.

Understanding **where** you capture is just as important as understanding **what** you capture.

---

## Building a Troubleshooting Workflow

Rather than capturing packets randomly, develop a repeatable workflow.

1. Verify the correct interface.
2. Capture traffic with `tcpdump`.
3. Confirm that packets leave the source.
4. Confirm that replies return.
5. Narrow the capture using filters.
6. Save captures for later analysis if necessary.

Following the same process every time makes troubleshooting faster and more reliable.

---

## Looking Ahead

So far, we've focused on packets moving through network interfaces.

The next question is:

> **Which application owns these network connections?**

In the next chapter, we'll explore Linux sockets and learn how to inspect active TCP and UDP connections using tools such as `ss`, allowing us to connect observed network traffic to the processes generating it.
