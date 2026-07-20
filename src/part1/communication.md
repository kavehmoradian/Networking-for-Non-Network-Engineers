# How Computers Actually Communicate

Before we dive into protocols, routing, or Linux networking, let's answer a much more fundamental question:

> **How does one computer send information to another?**

Whether you're opening a website, connecting to a database, or sending a message over SSH, every network operation begins with one computer sending information to another. While the technologies involved can become complex, the basic idea is surprisingly simple: computers exchange data by transmitting a sequence of electrical, optical, or radio signals over a communication medium.

## Everything Starts with Data

Computers don't understand text, images, or videos.

They understand only **binary data**—a sequence of **0s** and **1s**, known as **bits**.

For example, the text:

```text
Hello
```

is stored as binary data:

```text
01001000 01100101 01101100 01101100 01101111
```

Fortunately, humans never work directly with these bits. Software converts them into formats that computers can store, process, and transmit.

## From Bits to Bytes

A single bit can represent only two values: `0` or `1`.

To represent more complex information, bits are grouped together.

The most common grouping is the **byte**, which consists of **8 bits**.

For example:

| Binary   | Decimal | Character |
| -------- | ------- | --------- |
| 01000001 | 65      | A         |
| 01000010 | 66      | B         |
| 00110001 | 49      | 1         |

Everything stored or transmitted by a computer—documents, images, videos, and network traffic—is ultimately represented as bytes.

## Sending Data Across a Network

Imagine Computer A wants to send the word:

```text
Hello
```

to Computer B.

The application first converts the text into bytes.

Those bytes must then travel across some form of communication medium, such as:

* An Ethernet cable
* Fiber optic cable
* Wi-Fi
* A cellular network

Regardless of the medium, the goal is always the same:

> Convert bytes into signals that another computer can receive and convert back into bytes.

At the lowest level, networks don't send "messages" or "files."

They send signals that represent bits.

## Why Data Is Broken Into Pieces

Suppose you want to send a 2 GB file.

Sending it as one enormous block would be inefficient.

If a single bit were corrupted during transmission, the entire file would have to be sent again.

Instead, computers divide data into many smaller pieces before transmitting it.

Each piece can be transmitted independently and, if necessary, retransmitted without affecting the rest of the data.

These pieces are known by different names depending on where they are in the networking stack:

* **Frame** – a unit of data at the data-link layer.
* **Packet** – a unit of data at the network layer.
* **Segment** – a unit of data at the transport layer.

You'll learn what each of these means in the following chapters. For now, it's enough to understand that large amounts of data are always divided into smaller units before being transmitted.

## A Journey Across the Network

Imagine you're opening a web page.

Although it appears to be a single action, a series of events takes place behind the scenes:

1. Your browser generates a request.
2. The request is converted into bytes.
3. The bytes are divided into smaller pieces.
4. Those pieces are transmitted across the network.
5. Routers and switches forward them toward the destination.
6. The destination computer receives the pieces.
7. The original data is reconstructed.
8. The web server processes the request and sends a response using the same process.

This entire sequence usually completes in just a few milliseconds.

## Building on This Foundation

At this point, you only need to remember three key ideas:

* Computers communicate by exchanging binary data.
* That data is transmitted as electrical, optical, or radio signals.
* Large amounts of data are divided into smaller pieces before being sent across a network.

The next chapter introduces the **TCP/IP stack**, which defines how those pieces are organized, addressed, transmitted, and reassembled so that computers around the world can communicate reliably.
