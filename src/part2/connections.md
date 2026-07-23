# Understanding Linux Network Connections

So far in this book, we've focused on the network itself.

We've learned how Linux selects routes, resolves MAC addresses, and transmits packets through network interfaces.

But networking doesn't exist just for the kernel—it exists for applications.

When a web server listens on port 443 or a browser connects to a website, they don't interact with network interfaces directly. Instead, they communicate through an operating system abstraction called a **socket**.

Understanding sockets is essential for troubleshooting Linux systems because almost every networking problem eventually comes down to answering questions like:

* Which process is listening on this port?
* Is the application accepting connections?
* Is data flowing?
* Why is the connection stuck?
* Is the application keeping up with incoming traffic?

In this chapter, we'll explore how Linux represents network connections and learn how to inspect them using modern Linux tools.

---

## From Packets to Sockets

Consider a web server.

```text
          Client
             │
             │ TCP
             ▼
      Network Interface
             │
             ▼
      Linux Networking Stack
             │
             ▼
          Socket
             │
             ▼
        Web Server
```

Packets arrive through a network interface.

The Linux networking stack processes them.

Eventually, the payload is delivered to a socket owned by an application.

Similarly, when an application sends data, it writes to a socket.

The kernel handles everything else.

---

## What Is a Socket?

A socket is an endpoint for network communication.

Applications do not send Ethernet frames or IP packets directly.

Instead, they perform operations such as:

* `socket()`
* `bind()`
* `listen()`
* `accept()`
* `connect()`
* `send()`
* `recv()`

These system calls interact with sockets managed by the Linux kernel.

You don't need to be a programmer to troubleshoot sockets, but understanding this abstraction makes many networking tools easier to understand.

---

## Socket Types

Linux supports several socket families and types.

The most common are:

| Type | Purpose                            |
| ---- | ---------------------------------- |
| TCP  | Reliable stream communication      |
| UDP  | Connectionless datagrams           |
| UNIX | Local inter-process communication  |
| RAW  | Direct access to network protocols |

In this chapter, we'll focus primarily on TCP and UDP sockets.

---

## Inspecting Connections with ss

The standard tool for inspecting sockets is `ss`.

Display all TCP connections:

```bash
ss -t
```

Display all UDP sockets:

```bash
ss -u
```

Display both listening and established sockets:

```bash
ss -tun
```

Unlike the older `netstat` utility, `ss` communicates directly with the kernel and is significantly faster on busy systems.

---

## Understanding ss Output

Example:

```text
State      Recv-Q Send-Q Local Address:Port      Peer Address:Port

ESTAB      0      0      192.168.1.10:22         192.168.1.20:52144
```

Each column has a specific meaning.

| Column        | Description                                     |
| ------------- | ----------------------------------------------- |
| State         | Current connection state                        |
| Recv-Q        | Bytes waiting to be read by the application     |
| Send-Q        | Bytes waiting to be transmitted or acknowledged |
| Local Address | Local IP and port                               |
| Peer Address  | Remote IP and port                              |

Understanding these fields is often enough to diagnose many networking problems.

---

## Listening Sockets

Servers create listening sockets while waiting for clients.

Display listening sockets:

```bash
ss -ltn
```

Example:

```text
LISTEN 0 4096 0.0.0.0:443
```

This means:

* TCP socket
* Listening
* Port 443
* Accepting connections on every IPv4 interface

To include process information:

```bash
sudo ss -ltnp
```

Example:

```text
LISTEN 0 4096 0.0.0.0:443 users:(("nginx",pid=2145))
```

Now we know exactly which process owns the socket.

---

## Receive Queue (Recv-Q)

One of the most misunderstood fields in `ss` is **Recv-Q**.

Recv-Q represents data that has already arrived from the network but has **not yet been read by the application**.

For example:

```text
State     Recv-Q 
ESTAB     20480  
```

This indicates:

* Packets have successfully reached the server.
* The kernel has buffered the data.
* The application has not consumed it yet.

A growing receive queue often suggests that:

* The application is slow.
* The application is blocked.
* The application is overloaded.

The network is usually **not** the problem.

---

## Send Queue (Send-Q)

Send-Q represents data that the application has written to the socket but has **not yet been fully transmitted or acknowledged**.

Possible causes include:

* Network congestion
* Slow receivers
* Packet loss
* Small TCP receive windows

A constantly increasing Send-Q deserves investigation.

Unlike Recv-Q, a large Send-Q often points toward network conditions rather than application performance.

Suppose we observe:

```text
State     Recv-Q    Send-Q 
ESTAB     524288    0
```

Interpretation:

* Clients are sending data.
* Linux is receiving it.
* The application is not reading it fast enough.

Now consider:

```text
State     Recv-Q    Send-Q 
ESTAB     0        524288
```

Interpretation:

* The application is producing data.
* The peer is not receiving it quickly enough.
* The network or receiver may be the bottleneck.

Learning to interpret these two numbers is an essential troubleshooting skill.

---

## Displaying Detailed Socket Information

For more information:

```bash
ss -ti
```

Example output includes:

* Congestion control algorithm
* RTT (Round Trip Time)
* Retransmissions
* Receive window
* Send window
* MSS
* Congestion window

These statistics are extremely useful when diagnosing TCP performance issues.

---

## Monitoring Connections

Watch connections in real time:

```bash
watch -n 1 ss -tun
```

This refreshes the connection table every second.

It's useful for observing:

* Incoming client connections
* Connection spikes
* Short-lived connections
* Idle sessions

---

## Identifying Which Process Owns a Connection

One of the most common troubleshooting tasks is determining which process is using a port.

```bash
sudo ss -tunp
```

Example:

```text
ESTAB

users:(("postgres",pid=1523))
```

Now you know:

* Process name
* Process ID
* Connection details

This is often faster than searching through process lists manually.

---

## Inspecting UNIX Sockets

Not all sockets communicate over a network.

Many Linux services communicate locally using UNIX domain sockets.

Display them:

```bash
ss -xl
```

Example:

```text
u_str LISTEN /run/systemd/private
```

UNIX sockets avoid network overhead and are widely used by:

* Docker
* systemd
* PostgreSQL
* MySQL
* Redis
* containerd

Although they don't use IP addresses, they share the same socket abstraction.

---

## Common Troubleshooting Questions

### Is the application listening?

```bash
ss -ltn
```

---

### Which process owns port 443?

```bash
sudo ss -ltnp | grep :443
```

---

### Is the connection established?

```bash
ss -tan
```

---

### Is the application reading data?

Check **Recv-Q**.

---

### Is the network preventing data from being delivered?

Check **Send-Q**.

---

### Is the connection closing correctly?

Inspect the TCP state.

---

## A Practical Troubleshooting Workflow

When investigating an application connectivity problem:

1. Verify the service is listening.
2. Confirm the correct process owns the socket.
3. Verify the TCP connection reaches the ESTABLISHED state.
4. Inspect Recv-Q and Send-Q.
5. Capture packets with `tcpdump` if necessary.
6. Verify routing and neighbor resolution if packets are missing.

Notice how every chapter in this part builds upon the previous one.

---

## Looking Ahead

So far we've explored Linux networking on a single host.

We've seen interfaces, routing tables, neighbor tables, packet captures, and sockets.

In the next chapter, we'll bring these concepts together by building a small Linux network from scratch. We'll create multiple hosts, configure addressing and routing, generate traffic, and use the tools we've learned to observe every step of the communication process.
