# TCP, UDP, Ports and Sockets

In the previous chapter, we learned that the **Transport layer** is responsible for communication between applications running on different computers. But how does that communication actually work?

Imagine a server running multiple applications:

* A web server listening for HTTP requests
* An SSH server accepting remote logins
* A database waiting for queries

All of these applications share the same network interface and the same IP address. When a packet arrives, how does Linux know which application should receive it?

The answer lies in four closely related concepts:

* **TCP**
* **UDP**
* **Ports**
* **Sockets**

Understanding these concepts is essential because almost every network application depends on them.

---

## TCP vs. UDP

The two most common transport protocols are **TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)**.

Although both transport data between applications, they are designed for different purposes.

| TCP                     | UDP                              |
| ----------------------- | -------------------------------- |
| Connection-oriented     | Connectionless                   |
| Reliable delivery       | Best-effort delivery             |
| Guarantees packet order | No ordering guarantee            |
| Detects lost packets    | Does not retransmit lost packets |
| Higher overhead         | Lower overhead                   |
| Slightly higher latency | Very low latency                 |

The choice between TCP and UDP depends on what the application needs.

---

## TCP: Reliable Communication

TCP is designed for situations where every byte of data matters.

Before any data is exchanged, the two computers establish a connection.

During communication, TCP:

* Detects lost packets
* Retransmits missing data
* Delivers data in the correct order
* Prevents duplicate data
* Adjusts transmission speed based on network conditions

These guarantees make TCP ideal for applications such as:

* Web browsing (HTTP/HTTPS)
* SSH
* Email
* Databases
* File transfers

Reliability comes at a cost.

TCP requires additional processing and control messages, making it slightly slower than UDP.

---

## UDP: Fast and Lightweight

UDP takes a much simpler approach.

It sends data without first establishing a connection.

There is no guarantee that:

* Packets arrive
* Packets arrive only once
* Packets arrive in order

If a packet is lost, UDP does nothing.

This simplicity makes UDP extremely fast and efficient.

Applications that use UDP either tolerate packet loss or implement their own reliability mechanisms.

Common examples include:

* DNS
* Voice over IP (VoIP)
* Video conferencing
* Online gaming
* Streaming
* DHCP

For these applications, receiving data quickly is often more important than receiving every packet.

---

## Ports

An IP address identifies a computer.

A **port** identifies an application running on that computer.

Think of an apartment building.

The building's street address identifies the building.

The apartment number identifies the individual resident.

Likewise:

```text
IP Address → Computer

Port → Application
```

For example:

```text
192.168.1.10:22
```

means:

* Computer: `192.168.1.10`
* Application listening on port `22` (typically SSH)

A single computer can have thousands of ports open simultaneously, allowing many applications to communicate at the same time.

---

## Well-Known Ports

Many common services use standardized port numbers.

| Port | Protocol | Typical Service |
| ---- | -------- | --------------- |
| 22   | TCP      | SSH             |
| 53   | TCP/UDP  | DNS             |
| 80   | TCP      | HTTP            |
| 443  | TCP      | HTTPS           |
| 3306 | TCP      | MySQL           |
| 5432 | TCP      | PostgreSQL      |
| 6379 | TCP      | Redis           |

Using standard ports allows clients to locate services without additional configuration.

Applications are free to use different ports, but these defaults are widely recognized.

---

## What Is a Socket?

A **socket** is the endpoint of a network connection.

It combines:

* An IP address
* A port number
* A transport protocol (TCP or UDP)

For example:

```text
192.168.1.10:443/TCP
```

identifies a specific TCP socket.

Applications create sockets to send and receive network data.

From the application's perspective, networking is performed through sockets rather than directly through TCP or UDP.

---

## Client and Server Sockets

When you open a website, two sockets are involved.

The server listens on a well-known port:

```text
203.0.113.10:443
```

Your computer creates a temporary client socket:

```text
192.168.1.25:51432
```

The complete connection is identified by four values:

```text
Client IP
Client Port
Server IP
Server Port
```

For example:

```text
192.168.1.25:51432
        ↓
203.0.113.10:443
```

This allows thousands of clients to communicate with the same server simultaneously, even though they are all connecting to port 443.

Each connection has a unique combination of client and server addresses and ports.

---

## Listening and Established Connections

Before a server can accept connections, it must create a **listening socket**.

For example:

```bash
ss -lnt
```

might show:

```text
State   Local Address:Port

LISTEN  0.0.0.0:22
LISTEN  0.0.0.0:80
LISTEN  0.0.0.0:443
```
TODO: more detail about sockets
When a client connects, Linux creates a new socket representing that specific connection.

This is why a web server can serve thousands of clients at the same time while listening on a single port.

---

## Putting It All Together

Suppose you run:

```bash
curl https://google.com
```

Behind the scenes:

1. `curl` creates a TCP socket.
2. Linux assigns a temporary source port.
3. A connection is established with the server's port `443`.
4. HTTP data is exchanged through the socket.
5. When communication is complete, the socket is closed.

From the application's point of view, everything happens through the socket. The operating system handles the underlying TCP communication.

---

## Looking Ahead

So far we've focused on communication between applications running on different computers.

But one important question remains:

> **How does Linux know where to send a packet once it leaves a socket?**

To answer that, we need to understand **routing**—the process Linux uses to decide where every packet should go next.
