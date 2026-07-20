# The TCP/IP Stack

In the previous chapter, we learned that computers exchange binary data by breaking it into smaller pieces and transmitting those pieces across a network. But one important question remains:

> **How do computers know how to create, send, receive, and interpret those pieces of data?**

The answer is **protocols**.

A protocol is simply a set of rules that defines how computers communicate. Just as two people must speak the same language to understand each other, computers must follow the same protocols to exchange data successfully.

Modern computer networks are built on a collection of protocols known as the **TCP/IP protocol suite**, or simply the **TCP/IP stack**.

---

## Why Do We Need a Stack?

Imagine you're mailing a package to someone in another country.

The process involves multiple independent steps:

* You write the message.
* You place it in an envelope.
* You write the destination address.
* A postal service transports it.
* The recipient receives and opens it.

Each step has a different responsibility.

Computer networking works in exactly the same way.

Instead of having one enormous protocol responsible for everything, networking is divided into multiple layers. Each layer performs one specific job and passes its data to the next layer.

This design makes networking simpler, more flexible, and easier to extend.

---

## The Four Layers of TCP/IP

The TCP/IP model consists of four layers.

| Layer       | Responsibility              | Common Protocols            |
| ----------- | --------------------------- | --------------------------- |
| Application | Network applications        | HTTP, HTTPS, DNS, SSH, SMTP |
| Transport   | End-to-end communication    | TCP, UDP                    |
| Internet    | Addressing and routing      | IP, ICMP                    |
| Link        | Local network communication | Ethernet, Wi-Fi, ARP        |

Each layer provides services to the layer above it while relying on the layer below it.

For example, a web browser doesn't need to know how Ethernet works. It simply asks the transport layer to deliver data to another computer.

---

## Encapsulation

Suppose you visit:

```text
https://google.com
```

Your browser creates an HTTP request.

As the request moves down the TCP/IP stack, each layer adds its own information.

```
Application Layer
┌──────────────────────────┐
│ HTTP Request             │
└──────────────────────────┘
             │
             ▼
Transport Layer
┌──────────────────────────┐
│ TCP Header               │
│ HTTP Request             │
└──────────────────────────┘
             │
             ▼
Internet Layer
┌──────────────────────────┐
│ IP Header                │
│ TCP Header               │
│ HTTP Request             │
└──────────────────────────┘
             │
             ▼
Link Layer
┌──────────────────────────┐
│ Ethernet Header          │
│ IP Header                │
│ TCP Header               │
│ HTTP Request             │
└──────────────────────────┘
```

Each layer wraps the data produced by the layer above it.

This process is called **encapsulation**.

When the data reaches the destination, the opposite happens.

Each layer removes its own header until the original HTTP request reaches the web server.

This reverse process is called **decapsulation**.

---

## What Does Each Layer Actually Do?

### Application Layer

This is where applications live.

Your browser, SSH client, email client, or database client all operate here.

Application protocols define the meaning of the data being exchanged.

Examples include:

* HTTP
* HTTPS
* DNS
* SSH
* SMTP
* FTP

The application layer does **not** worry about routing, packet delivery, or network hardware.

Its only concern is the application data.

---

### Transport Layer

The transport layer enables communication between two applications running on different computers.

Its responsibilities include:

* Identifying applications using ports
* Breaking large messages into smaller pieces
* Detecting lost data
* Reordering data if necessary
* Ensuring reliable delivery (TCP)
* Providing lightweight delivery (UDP)

The two most common transport protocols are:

* TCP
* UDP

We'll explore both in detail in the next chapter.

---

### Internet Layer

The Internet layer is responsible for moving data between networks.

It answers questions such as:

* Where should this packet go?
* Which network is the destination on?
* Which router should receive it next?

The primary protocol here is **IP (Internet Protocol)**.

Unlike TCP, IP makes no guarantees that packets will arrive or arrive in order.

Its job is simply to move packets toward their destination.

---

### Link Layer

The Link layer is responsible for communication within a local network.

This includes technologies such as:

* Ethernet
* Wi-Fi

At this layer, computers communicate using hardware addresses (MAC addresses) instead of IP addresses.

We'll revisit this layer when we discuss Ethernet, Linux bridges, VLANs, and virtual networking.

---

## A Packet's Journey

Imagine you run:

```bash
curl https://google.com
```

Here's what happens:

1. `curl` creates an HTTP request.
2. The transport layer adds a TCP header.
3. The Internet layer adds an IP header.
4. The Link layer creates an Ethernet frame.
5. The frame is transmitted across the network.
6. The receiving computer removes the Ethernet header.
7. The IP layer processes the packet.
8. TCP reassembles the data if necessary.
9. The HTTP request reaches the web server.

Each layer performs exactly one job before passing the data to the next layer.

This layered approach is one of the primary reasons the Internet can scale to billions of devices.

---

## TCP/IP vs. OSI

You may have heard of the **OSI model**, which divides networking into seven layers.

While the OSI model is useful as a conceptual framework, modern operating systems and Internet protocols are based on the TCP/IP model.

Throughout this book, we'll use the TCP/IP model because it more accurately reflects how Linux networking is implemented.

---

## Looking Ahead

Understanding the TCP/IP stack is essential because nearly every networking technology you'll encounter fits somewhere within it.

As you continue through this book, you'll learn how:

* TCP and UDP operate at the Transport layer.
* IP and routing operate at the Internet layer.
* Ethernet, bridges, and VLANs operate at the Link layer.
* DNS, HTTP, and SSH operate at the Application layer.

By understanding the responsibilities of each layer, you'll be able to reason about networking problems systematically instead of treating the network as a black box.
