# How DNS Really Works

Imagine opening your browser and visiting:

```text
https://google.com
```

Within a few milliseconds, your browser connects to the correct web server somewhere on the Internet.

But computers don't communicate using names like `google.com`.

They communicate using **IP addresses**.

So before a connection can be established, one important question must be answered:

> **What is the IP address of `google.com`?**

The answer comes from the **Domain Name System (DNS)**.

DNS is often described as the Internet's phone book, but it's better thought of as a **distributed database** that translates human-readable names into IP addresses.

Without DNS, you would need to remember IP addresses instead of names for every website and service you use.

---

# Why Do We Need DNS?

Humans are good at remembering names.

Computers are good at using numbers.

For example, which is easier to remember?

```text
google.com
```

or

```text
93.184.216.34
```

Names are also flexible.

If a company moves its website to another server, the IP address may change, but users can continue visiting the same domain name.

DNS provides this layer of indirection.

Applications use names.

Networks use IP addresses.

DNS connects the two.

---

# Domain Names

A domain name is organized as a hierarchy.

For example:

```text
api.google.com
```

can be broken into three parts:

```text
api      example      com
 │           │         │
Host     Domain      Top-Level Domain (TLD)
```

Reading from right to left:

* **`.com`** is the **Top-Level Domain (TLD)**.
* **`example`** is the domain registered under `.com`.
* **`api`** is a subdomain (often representing a specific service).

This hierarchical structure allows the Internet to scale without requiring a single central database.

---

# DNS Records

DNS stores different types of information called **records**.

Some of the most common record types are:

| Record | Purpose                                                           |
| ------ | ----------------------------------------------------------------- |
| A      | Maps a hostname to an IPv4 address                                |
| AAAA   | Maps a hostname to an IPv6 address                                |
| CNAME  | Alias for another hostname                                        |
| MX     | Mail server for a domain                                          |
| NS     | Name servers responsible for the domain                           |
| TXT    | Arbitrary text, commonly used for verification and email security |

For example:

```text
google.com.     A      93.184.216.34
```

This record tells clients that `google.com` resolves to the IPv4 address `93.184.216.34`.

---

# DNS Resolution

Suppose you run:

```bash
curl https://google.com
```

Before TCP can establish a connection, Linux must first discover the server's IP address.

The process looks like this:

```text
Application
      │
      ▼
Need IP address
      │
      ▼
Query DNS Resolver
      │
      ▼
Receive IP address
      │
      ▼
Create TCP connection
```

Notice that **DNS happens before any TCP connection to the web server is created**.

Without the destination IP address, the operating system has nowhere to send the packet.

---

# Recursive Resolvers

Your computer usually doesn't contact the authoritative DNS server directly.

Instead, it asks a **recursive resolver**.

This resolver may belong to:

* Your ISP
* Your company
* A public DNS provider such as Google Public DNS or Cloudflare DNS

The resolver performs the work of finding the answer and returns the final IP address to your computer.

From the client's perspective, DNS appears to be a simple question-and-answer service.

---

# Where Does the Resolver Find the Answer?

If the recursive resolver doesn't already know the answer, it performs a series of queries.

Suppose we're looking up:

```text
api.google.com
```

The resolver follows the DNS hierarchy.

1. Ask a **Root Name Server**:

> "Who is responsible for `.com`?"

The root server replies with the name servers for `.com`.

↓

2. Ask a **`.com` Name Server**:

> "Who is responsible for `google.com`?"

The `.com` server replies with the authoritative name servers for `google.com`.

↓

3. Ask the **Authoritative Name Server**:

> "What is the IP address of `api.google.com`?"

The authoritative server returns the requested DNS record.

↓

4. Return the answer to the client.

The client now knows the server's IP address and can begin the TCP connection.

---

# DNS Caching

Performing the full lookup for every request would be slow.

Instead, DNS uses caching extensively.

Your operating system may cache results.

Your browser may cache results.

Your company's resolver may cache results.

Your ISP may cache results.

Each DNS record contains a value called **TTL (Time To Live)**.

TTL specifies how long a cached answer may be reused before it should be queried again.

For example:

```text
google.com. 300 IN A 93.184.216.34
```

A TTL of **300 seconds** means the result may be cached for five minutes.

Caching dramatically reduces DNS traffic and improves performance across the Internet.

---

# DNS Uses Both UDP and TCP

One interesting detail about DNS is that it uses **both UDP and TCP**.

Most DNS queries use **UDP port 53** because requests and responses are usually small, making UDP fast and efficient.

However, DNS switches to **TCP port 53** in situations such as:

* Large responses
* Zone transfers between DNS servers
* When a UDP response is too large

Most users never notice this transition because DNS software handles it automatically.

---

# Common DNS Tools

Linux provides several utilities for querying DNS.

To look up a hostname:

```bash
dig google.com
```

or

```bash
host google.com
```

or

```bash
nslookup google.com
```

Among these tools, **`dig`** is generally preferred because it provides the most detailed output.

We'll use these commands later in the Linux networking section of this book.

---

# Putting It All Together

When you visit:

```text
https://google.com
```

the following events occur:

```text
Browser
    │
    ▼
DNS Lookup
    │
    ▼
Receive IP Address
    │
    ▼
Routing
    │
    ▼
ARP
    │
    ▼
TCP Connection
    │
    ▼
HTTPS Request
    │
    ▼
Web Server Response
```

Notice that everything we've learned in Part 1 works together.

* DNS translates a name into an IP address.
* Routing determines where packets should go.
* ARP discovers the next hop's MAC address.
* TCP establishes a reliable connection.
* The application finally exchanges data.

By this point, you have the fundamental concepts needed to understand how two computers communicate across a network. The rest of this book builds on these foundations by exploring how Linux implements networking internally and how these same concepts are used to build modern virtual networks.
