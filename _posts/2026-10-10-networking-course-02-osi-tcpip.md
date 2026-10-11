---
title: "Networking Course 02: The OSI and TCP/IP Models"
date: 2026-10-10 10:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, osi, tcp-ip]
---
**Goal:** know the layers, what happens to data as it moves down them, and how to troubleshoot layer by layer.

## Encapsulation

When you send data, each layer wraps it with its own **header**. The receiver unwraps it in reverse order.


![Encapsulation: data becomes a segment, a packet, a frame and bits](/assets/img/posts/networking/fig-02-encapsulation.svg){: w="700" }
_Figure 2: as data goes down, each layer adds a header. The data-link layer also adds a trailer (FCS)._

## The OSI and TCP/IP models


![OSI model compared with the TCP/IP model](/assets/img/posts/networking/fig-03-osi-tcpip.svg){: w="700" }
_Figure 3: the 7-layer OSI model next to the 4-layer TCP/IP model used in practice._

| # | OSI layer | Role | Examples | PDU |
|---|---|---|---|---|
| 7 | Application | services for apps | HTTP, DNS, SMTP | Data |
| 6 | Presentation | format, compression, encryption | TLS, JPEG | Data |
| 5 | Session | open, keep and close sessions | NetBIOS, RPC | Data |
| 4 | Transport | end-to-end delivery, ports | TCP, UDP | Segment |
| 3 | Network | addressing and routing | IP, ICMP | Packet |
| 2 | Data Link | local delivery, MAC addresses | Ethernet, ARP | Frame |
| 1 | Physical | bits on the medium | cables, radio | Bits |

**Memory aid (layer 7 down to 1):** All People Seem To Need Data Processing.

## Worked example: following one web request

You open `https://example.com`.

| Layer | What happens |
|---|---|
| 7 Application | the browser builds an HTTP request; DNS finds the IP address |
| 6 Presentation | TLS encrypts the request |
| 4 Transport | TCP splits data into segments, source port 51544, destination port 443 |
| 3 Network | IP adds source `192.168.1.23` and destination `203.0.113.10` |
| 2 Data Link | Ethernet adds the MAC of your **router** (the next hop) |
| 1 Physical | bits travel over Wi-Fi or the cable |

## Worked example: reading a Wireshark packet

Wireshark shows each layer in its own line. Example for one packet:

```
Frame 12: 74 bytes on wire
Ethernet II, Src: 3c:52:82:aa:bb:cc, Dst: 00:1a:2b:3c:4d:5e
Internet Protocol Version 4, Src: 192.168.1.23, Dst: 203.0.113.10
Transmission Control Protocol, Src Port: 51544, Dst Port: 443
Transport Layer Security
```

The lines match the encapsulation figure: frame (layer 2), IP (layer 3), TCP (layer 4), TLS (layers 6-7).

## Troubleshooting by layer

| Symptom | Likely layer | Check |
|---|---|---|
| no link light, "cable unplugged" | 1 Physical | cable, port, Wi-Fi on |
| link is up but no IP address | 2 / 3 | DHCP, correct VLAN |
| can ping the gateway but not `8.8.8.8` | 3 Network | routing, default gateway, ISP |
| can ping `8.8.8.8` but not `example.com` | 7 (DNS) | DNS server settings |
| site opens but is very slow | 4 / 7 | packet loss, overloaded server |


> Start at layer 1 and move up. This one habit solves most network problems quickly.
{: .prompt-tip }

## Why layers matter for security

Every attack and every defense lives at a layer: ARP spoofing (2), IP spoofing (3), SYN flood (4), SQL injection (7).

## Lab: your first capture

1. Open Wireshark, choose your active interface and start capturing.
2. Visit a website, then stop the capture.
3. Type the filter `dns`, then `tcp`, then `http` or `tls`.
4. Click one packet and expand the Ethernet, IP and TCP lines. Write down the MAC, IP and port values.

## Review questions

1. What is the PDU name at layers 4, 3 and 2?
2. At which layer does a router work? A switch?
3. Which layer would you check first if the link light is off?
4. Which TCP/IP layer covers OSI layers 5, 6 and 7?

<details markdown="1">
<summary>Show answers</summary>

1. Segment, packet, frame.
2. A router works at layer 3; a normal switch at layer 2.
3. Layer 1, the physical layer.
4. The TCP/IP Application layer.

</details>
