---
title: "High Performance Browser Networking"
date: 2016-12-14T22:38:02+09:00
lastmod: 2017-12-14T22:38:02+09:00
draft: false
keywords: []
description: ""
tags: ['networking', 'http', 'browsers']
categories: ['Reading']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

<!-- toc -->

# Ch01. Primer on Latency and Bandwidth

> **Latency**
>
> The time from the source sending a packet to the destination receiving it

> **Bandwidth**
>
> Maximum throughput of a logical or physical communication path

![Bandwidth and latency](/images/high-performance-browser-networking/BandwidthAndLatency.png)

Latency, not bandwidth, is the performance bottleneck for most web‐ sites! 

To increase bandwidth, while it might not be cheap, we have multiple strategies available, like adding more fiber links.

Improving latency, on the other hand, is a very different story. Since we can’t make light travel faster, we can make the distance shorter. However, laying new cables is also not always possible due to the constraints imposed by the physical terrain, social and political reasons, and of course, the associated costs.

As a result, to improve performance of our applications, we need to architect and optimize our protocols and networking code with explicit awareness of the limitations of available bandwidth and the speed of light: we need to reduce round trips, move the data closer to the client, and build applications that can hide the latency through caching, pre-fetching, and a variety of similar techniques

# Ch02. Building Blocks of TCP

TCP provides an effective abstraction of a reliable network running over an unreliable channel, hiding most of the complexity of network communication from our applications: 

* retransmission of lost data
* in-order delivery
* data integrity

and more. When you work with a TCP stream, you are guaranteed that all bytes sent will be identical with bytes received and that they will arrive in the same order to the client.  

## Three-way handshake

![Illustration](/images/high-performance-browser-networking/threewayHandshake.png)

This startup process applies to every TCP connection and carries an important implication for performance of all network applications using TCP: each new connection will have a full roundtrip of latency before any application data can be transferred. Note that bandwidth of the connection plays no role here. Instead, the delay is governed by the latency between the client and the server.

## Congestion avoidance and control

Once the roundtrip time exceed the maximum retransmission interval for any host, that host will begin to introduce more and more copies of the same datagrams into the net. 

Mechanisms to address this issue:

* flow control
* congestion control
* congestion avoidance

### Flow control
Each side of the TCP connection advertises its own receive window (rwnd), which communicates the size of the available buffer space to hold the incoming data.

![Illustration](/images/high-performance-browser-networking/flowControl.png)

Flow control prevented the sender from overwhelming the receiver, but there was no mechanism to prevent either side from overwhelming the underlying network.

### Slow start

The only way to estimate the available capacity between the client and the server is to measure it by exchanging data. To start, the server initializes a new congestion window (cwnd) variable per TCP connection and sets its initial value to a conservative, system-specified value.

The cwnd variable is not advertised or exchanged between the sender and receiver—in this case, it will be a private variable maintained by the server.

Further, a new ￼￼￼￼Congestion Avoidance and Control rule is introduced: the maximum amount of data in flight (not ACKed) between the client and the server is the minimum of the rwnd and cwnd variables. We start with a small congestion window and double it for every roundtrip.

The performance of many web applications is often limited by the roundtrip time between server and client: slow-start limits the available bandwidth throughput, which has an adverse effect on the performance of small transfers.

### Congestion avoidance

Slow-start initializes the connection with a conservative window and, for every roundtrip, doubles the amount of data in flight until it exceeds 

* the receiver’s flow-control window, 
* a system-configured congestion threshold (ssthresh) window, 
* or until a packet is lost, 

at which point the congestion avoidance algorithm takes over. 

![Illustration](/images/high-performance-browser-networking/avoidance.png)

The implicit assumption in congestion avoidance is that packet loss is indicative of network congestion: somewhere along the path we have encountered a congested link or a router, which was forced to drop the packet, and hence we need to adjust our window to avoid inducing more packet loss to avoid overwhelming the network.

At a certain point, another packet loss event will occur, and the process will repeat once over.

## Bandwidth-Delay Product
If either the sender or receiver are frequently forced to stop and wait for ACKs for previous packets, then this would create gaps in the data flow,

![Data gap](/images/high-performance-browser-networking/dataGap.png)

which would consequently limit the maximum throughput of the connection. To address this problem, the window sizes should be made just big enough, such that either side can continue sending data until an ACK arrives back from the client for an earlier packet—no gaps,maximum throughput. The optimal window size can be calculated by bandwidth-delay product.

> **Bandwidth-delay product (BDP)**
>
>Product of data link’s capacity and its end-to-end delay. The result is the maximum amount of unacknowledged data that can be in flight at any point in time.



## Head-of-Line Blocking

While TCP is a popular choice, it is not the only, nor necessarily the best choice for every occasion. Specifically, some of the features, such as in-order and reliable packet delivery, are not always necessary and can introduce unnecessary delays and negative performance implications.

That is because if one of the packets is lost in route to the receiver, then all subsequent packets must be held in the receiver’s TCP buffer until the lost packet is retransmitted and arrives at the receiver.

![Head-of-Line](/images/high-performance-browser-networking/TCP_HOL.png)

This effect is known as TCP head-of-line (HOL) blocking

The delay imposed by head-of-line blocking allows our applications to avoid having to deal with packet reordering and reassembly, which makes our application code much simpler. However, this is done at the cost of introducing unpredictable latency variation in the packet arrival times, commonly referred to as jitter, which can negatively impact the performance of the application.

## Optimizing for TCP

TCP is an adaptive protocol designed to be fair to all network peers and to make themost efficient use of the underlying network. Thus, the best way to optimize TCP is to tune how TCP senses the current network conditions and adapts its behavior based on the type and the requirements of the layers below and above it: e.g., wireless networks may need different congestion algorithms.

While the specific details of each algorithm and feedback mechanismwill continue to evolve, the core principles and their implications remain unchanged:

* TCP three-way handshake introduces a full roundtrip of latency.
* TCP slow-start is applied to every new connection.
* TCP flow and congestion control regulate throughput of all connections.
* TCP throughput is regulated by current congestion window size.

As a result, the rate with which a TCP connection can transfer data in modern high-speed networks is often limited by the roundtrip time between the receiver and sender.

# Ch03. Building Blocks of UDP

The primary feature and appeal of UDP is not in what it introduces, but rather in all the features it chooses to omit.

The most well-known use of UDP, and one that every browser and Internet application depends on, is the Domain Name System (DNS). 

The Web Real-Time Communication (WebRTC) standards are enabling real-time communication, such as voice and video calling and other forms of peer-to-peer (P2P) communication, natively within the browser via UDP.

## Null Protocol Services

At its core, UDP simply provides “application multiplexing” on top of IP by embedding the source and the target application ports of the communicating hosts. With that in mind, we can now summarize all the UDP non-services:

* No guarantee of message delivery
* No guarantee of order of delivery
* No connection state tracking
* No congestion control

TCP is a byte-stream oriented protocol capable of transmitting application messages spread across multiple packets without any explicit message boundaries within the packets themselves. To achieve this, connection state is allocated on both ends of the connection, and each packet is sequenced, retransmitted when lost, and delivered in order. UDP datagrams, on the other hand, have definitive boundaries: each datagram is carried in a single IP packet, and each application read yields the full message; datagrams cannot be fragmented.

## UDP and Network Address Translators

The proposed IP reuse solution was to introduce NAT devices at the edge of the network, each of which would be responsible for maintaining a table mapping of local IP and port tuples to one or more globally unique (public) IP and port tuples.

![Figure](/images/high-performance-browser-networking/NAT.png)

The local IP address space behind the translator could then be reused among many different networks, thus solving the address depletion problem.

Each TCP connection has a well-defined protocol state machine, which begins with a handshake, followed by application data transfer, and a well-defined exchange to close the connection. Given this flow, each middlebox can observe the state of the connection and create and remove the routing entries as needed. With UDP, there is no handshake or connection termination, and hence there is no connection state machine to monitor.



## Optimizing for UDP
Unlike TCP, which ships with built-in flow and congestion control and congestion avoidance, UDP applications must implement these mechanisms on their own. Congestion insensitive UDP applications can easily overwhelm the network, which can lead to degraded network performance and, in severe cases, to network congestion/collapse.


# Ch04. Transport Layer Security (TLS)

When the SSL (Secure Sockets Layer) protocol was standardized by the IETF, it was renamed to Transport Layer Security (TLS). So basically they are just different versions of the protocol.

TLS protocol is between the application layer and above the transport layer. 

![TLS](/images/high-performance-browser-networking/TLS.png)

## Encryption, Authentication, and Integrity

The TLS protocol is designed to provide three essential services to all applications running above it: 

* encryption
* authentication
* data integrity

In order to establish a cryptographically secure data channel, TLS protocol specifies a well-defined handshake sequence, which uses asymmetric key cryptography.

As part of the TLS handshake, the protocol also allows both connection peers to authenticate their identity. It allows the client to verify that the server is who it claims to be (e.g., your bank) and not someone simply pretending to be the destination by spoofing its name or IP address. This verification is based on the established chain of trust.

TLS protocol also provides its own message framing mechanism and signs each message with a message authentication code (MAC). The MAC algorithm is a one-way cryptographic hash function (effectively a checksum) to ensure message integrity and authenticity.

## TLS Handshake

Before the client and the server can begin exchanging application data over TLS, the encrypted tunnel must be negotiated: the client and the server must agree on the version of the TLS protocol, choose the ciphersuite, and verify certificates if necessary. Unfortunately, each of these steps requires new packet roundtrips between the client and the server, which adds startup latency to all TLS connections.

### Detailed Steps
1. TLS runs on TCP, so a TCP 3-way handshake is first completed.
2. (PlainText) Client sends some specs, such as TLS version, supported ciphersuites.
3. Server decides the specs to use. Server sends its certificate to client.
4. Client verifies certificate. Client generates a new symmetric key, encrypts it with server's public key, and then sends it to server.
5. Server decrypts the symmetric key, checks integrity with MAC, and send a "Finished" message encrypted to client.
6. Client decrypts the message, verifies the MAC.
7. If all is well, the secure tunnel is established.

## TLS Session Resumption

To help mitigate some of the extra latency and computational cost, TLS provides an ability to resume or share the same negotiated secret key data between multiple connections.

### Session Identifiers

Session Identifiers resumption mechanism allowed the server to create and send a 32-byte session identifier as part of its “ServerHello” message during the full TLS negotiation

Internally, the server could then maintain a cache of session IDs and the negotiated session parameters for each peer. In turn, the client could then also store the session ID information and include the ID in the “ClientHello” message for a subsequent session, which serves as an indication to the server that the client still remembers the negotiated cipher suite and keys from previous handshake and is able to reuse them.

However, one of the practical limitations of the Session Identifiers mechanism is the requirement for the server to create and maintain a session cache for every client.

### Session Tickets

Session Ticket replacement mechanism removes the requirement for the server to keep per-client session state. Instead, in the last exchange of the full TLS handshake, the server can include a New Session Ticket record, which includes all of the session data encrypted with a secret key known only by the server. This session ticket is then stored by the client .

## Chain of Trust and Certificate Authorities

![CertificateChain](/images/high-performance-browser-networking/CertificateChain.png)


# Ch05. Introduction to Wireless Networks

## Performance Fundamentals

### Bandwidth

The overall channel bitrate is directly proportional to the assigned range. Hence, all else being equal, a doubling in available frequency range will double the data rate— e.g., going from 20 to 40 MHz of bandwidth can double the channel data rate, which is exactly how 802.11n is improving its performance over earlier WiFi standards.

It is also worth noting that not all frequency ranges offer the same performance. Low-frequency signals travel farther and cover large areas (macrocells), but at the cost of requiring larger antennas and having more clients competing for access. On the other hand, high-frequency signals can transfer more data but won’t travel as far, resulting in smaller coverage areas (microcells) and a requirement for more infrastructure.

# Ch09. Brief History of HTTP

## HTTP 0.9

```
 $> telnet google.com 80

    Connected to 74.125.xxx.xxx

    GET /about/

    (hypertext response)
    (connection closed)
```

* ASCII only
* Response is a single HTML document
* Connection closes after document transfer


## HTTP 1.0

```
$> telnet website.org 80

    Connected to xxx.xxx.xxx.xxx

    GET /rfc/rfc1945.txt HTTP/1.0
    User-Agent: CERN-LineMode/2.15 libwww/2.17b3
    Accept: */*

    HTTP/1.0 200 OK
    Content-Type: text/plain
    Content-Length: 137582
    Expires: Thu, 01 Dec 1997 16:00:00 GMT
    Last-Modified: Wed, 1 May 1996 12:45:26 GMT
    Server: Apache 0.84

    (plain-text response)
    (connection closed)
```

* Request may consist of multiple newline separated header fields.
* Response object is prefixed with a response status line.
* Response object has its own set of newline separated header fields.
* Response object is not limited to hypertext (html, plain text, image, etc.)
* The connection between server and client is closed after every request.


## HTTP 1.1

HTTP 1.1 introduced a number of critical performance optimizations: keep-alive connections, chunked encoding transfers, byte-range requests, additional caching mechanisms, transfer encodings, and request pipelining.

```
$> telnet website.org 80
    Connected to xxx.xxx.xxx.xxx

    GET /index.html HTTP/1.1
    Host: website.org
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4)... (snip)
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.3
    Cookie: __qca=P0-800083390... (snip)
    
    HTTP/1.1 200 OK
    Server: nginx/1.0.11
    Connection: keep-alive
    Content-Type: text/html; charset=utf-8
    Via: HTTP/1.1 GWA
    Date: Wed, 25 Jul 2012 20:23:35 GMT
    Expires: Wed, 25 Jul 2012 20:23:35 GMT
    Cache-Control: max-age=0, no-cache
    Transfer-Encoding: chunked

    100
    <!doctype html>
    (snip)
    
    100
    (snip)

    0

    GET /favicon.ico HTTP/1.1
    Host: www.website.org
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_4)... (snip)
    Accept: */*
    Referer: http://website.org/
    Connection: close
    Accept-Encoding: gzip,deflate,sdch
    Accept-Language: en-US,en;q=0.8
    Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.3
    Cookie: __qca=P0-800083390... (snip)
    
    HTTP/1.1 200 OK
    Server: nginx/1.0.11
    Content-Type: image/x-icon
    Content-Length: 3638
    Connection: close
    Last-Modified: Thu, 19 Jul 2012 17:51:44 GMT
    Cache-Control: max-age=315360000
    Accept-Ranges: bytes
    Via: HTTP/1.1 GWA
    Date: Sat, 21 Jul 2012 21:35:22 GMT
    Expires: Thu, 31 Dec 2037 23:55:55 GMT
    Etag: W/PSA-GAu26oXbDi
    
    (icon data)
    (connection closed)
```

We have two object requests, one for an HTML page and one for an image, both delivered over a single connection. This is connection keep-alive.

Additionally, the HTTP 1.1 protocol added content, encoding, character set, and even language negotiation, transfer encoding, caching directives, client cookies, plus a dozen other capabilities that can be negotiated on each request.


# Ch10. Primer on Web Performance

## Hypertext, Web pages, and Web Applications

In HTTP 0.9 era (hypertext only), where only a single HTML document is transferred, tuning for performance was as simple as optimizing for a single HTTP request over a short-lived TCP connection.

In web page era, where multiple TCP connections are now potentially at play, and the key performance metric has shifted from document load time to **page load time**. 

> Page load time is time to **onload** event in the browser, which is an event fired by the browser once the document and all of its dependent resources (JavaScript, images, etc.) have finished loading.

Finally, in web application era, although page load time is still important, we are now interested in answering application-specific questions, which are tightly related to user interactions.

## Speed, Performance, and Human Perception

![TimeAndUserPerception](/images/high-performance-browser-networking/TimeAndUserPerception.png)

For an application to feel instant, a perceptible response to user input must be provided within hundreds of milliseconds. After a second or more, the user’s flow and engagement with the initiated task is broken, and after 10 seconds have passed, unless progress feedback is provided, the task is frequently abandoned.

Now, add up the network latency of a DNS lookup, followed by a TCP handshake, and another few roundtrips for a typical web page request, and much, if not all, of our 100– 1,000 millisecond latency budget can be easily spent on just the networking overhead

## Analyzing the Resource Waterfall

1. While the content of the web document is being fetched, new HTTP requests are being dispatched: HTML parsing is performed incrementally, allowing the browser to discover required resources early and dispatch the necessary requests in parallel. Hence, the scheduling of when the resource is fetched is in large part determined by the structure of the markup.

2. The rendering process starts well before all the resources are fully loaded, allowing the user to begin interacting with the page while the page is being built.


Optimizing the resource waterfall (by rearranging resources in HTML structure or so on) is also referred as *front-end performance optimization*. However, in reality, the server response times and network latency (“back-end performance”) are no less critical for optimizing the resource waterfall.

Usually download time is only a small fraction of the total latency of each connection. There are a lot of network latency while waiting to receive the first byte of each response.

Bandwidth is not the limiting performance factor for most web applications. Instead, the bottleneck is the network roundtrip latency between the client and the server.

## Browser Optimization

* Critical resources such as CSS and JavaScript should be discoverable as early as possible in the document.
* CSS should be delivered as early as possible to unblock rendering and JavaScript execution.
* Noncritical JavaScript should be deferred to avoid blocking DOM and CSSOM construction.
* The HTML document is parsed incrementally by the parser; hence the document should be periodically flushed for best performance.


# Ch11. HTTP 1.X

HTTP 1.1 introduced a large number of critical performance enhancements and features, such as:

* Persistent connections to allow connection reuse
* Chunked transfer encoding to allow response streaming
* Request pipelining to allow parallel request processing
* Byte serving to allow range-based resource requests
* Improved and much better-specified caching mechanisms

## Benefits of Keepalive Connections

Reuse HTTP connection to save roundtrip latency.

## HTTP Pipelining

* Without HTTP pipelining:
    * dispatch request, wait for the full response, dispatch next request from the client queue.
* With HTTP pipelining:
    * the FIFO queue is relocated from the client (request queuing) to the server (response queuing).


# Ch14. Primer on Browser Networking

In modern browser, there are self-managed socket pools used for connection management. Sockets are grouped by origin, and each pool enforces its own connection limits and security constraints. 

![SocketPools](/images/high-performance-browser-networking/SocketPools.png)

Such architecture offers several performance benefits:

* The browser can service queued requests in priority order.
* The browser can reuse sockets to minimize latency and improve throughput.
* The browser can be proactive in opening sockets in anticipation of request.
* The browser can optimize when idle sockets are closed.
* The browser can optimize bandwidth allocation across all sockets.

# Ch15. XMLHttpRequest

XMLHttpRequest (XHR) is a browser-level API that enables the client to script data transfers via JavaScript.

The power of XHR is not only that it enabled asynchronous communication within the browser, but also that it made it simple. Since XHR is an application API provided by the browser, the browser automatically takes care of all the low- level connection management, protocol negotiation, formatting of HTTP requests, and much more:

* The browser manages connection establishment, pooling, and termination.
* The browser determines the best HTTP(S) transport (HTTP 1.0, 1.1, 2.0).
* The browser handles HTTP caching, redirects, and content-type negotiation.
* The browser enforces security, authentication, and privacy constraints.
* And more...

However, XHR also has its limitations. As we saw, streaming has never been an official use case in the XHR standard, and the support is limited.

Similarly, there is no one best strategy for delivering real-time updates with XHR. Periodic polling incurs high overhead and message latency delays. Long-polling delivers low latency but still has the same per-message overhead; each message is its own HTTP request.
