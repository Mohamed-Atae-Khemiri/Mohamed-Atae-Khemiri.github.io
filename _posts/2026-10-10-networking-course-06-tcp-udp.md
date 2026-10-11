---
title: "Networking Course 06: Transport Layer, TCP and UDP"
date: 2026-10-10 14:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, tcp, udp, ports, nmap]
---
**Goal:** understand ports, connections and the TCP handshake, and read what is open on a host.

## TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Connection | connection-oriented | connectionless |
| Reliability | acknowledgments and retransmission | none |
| Order | guaranteed | not guaranteed |
| Speed | slower | faster |
| Uses | web, email, SSH | DNS queries, VoIP, streaming, games |

## Ports

A port (0 to 65535) tells the OS which program should receive the data. A **socket** is an IP address plus a port, for example `203.0.113.10:443`.

| Port | Service | Port | Service |
|---|---|---|---|
| 20/21 | FTP | 110 | POP3 |
| 22 | SSH | 143 | IMAP |
| 23 | Telnet | 443 | HTTPS |
| 25 | SMTP | 445 | SMB |
| 53 | DNS | 3389 | RDP |
| 67/68 | DHCP | 80 | HTTP |

## The three-way handshake


![TCP three-way handshake between a client and a server](/assets/img/posts/networking/fig-08-tcp-handshake.svg){: w="700" }
_Figure 8: SYN, SYN-ACK, ACK. The numbers are example sequence numbers._

1. **SYN:** the client says "I want to connect" (sequence 1000).
2. **SYN-ACK:** the server agrees (sequence 5000, acknowledges 1001).
3. **ACK:** the client confirms (acknowledges 5001). Data can flow.

Closing uses **FIN** (graceful) or **RST** (abrupt). Other flags: PSH, URG.

## Worked example: what is listening on my machine?

```bash
ss -tuln
```

Example output:

```
Netid State  Local Address:Port
tcp   LISTEN 0.0.0.0:22
tcp   LISTEN 127.0.0.1:631
udp   UNCONN 0.0.0.0:68
```

Reading it: SSH is open to everyone (`0.0.0.0:22`), the printer service listens only on the machine itself (`127.0.0.1`), and the DHCP client uses UDP 68. Every listening port is something to protect or close.

## Worked example: scan your own lab machine

Only scan systems you own or have permission to test.

```bash
nmap -sV 192.168.1.50
```

Example output:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2
80/tcp open  http    nginx 1.24
443/tcp closed https
```

`open` means a service answered, `closed` means the host answered with a reset, and `filtered` (not shown) means a firewall dropped the probe.

## Worked example: filters in Wireshark

| Filter | Shows |
|---|---|
| `tcp.flags.syn==1 && tcp.flags.ack==0` | connection attempts (first handshake packet) |
| `tcp.port==443` | HTTPS traffic |
| `udp.port==53` | DNS queries |
| `tcp.flags.reset==1` | connections that were refused or reset |


> A burst of SYN packets to many ports from one address in a short time is the signature of a port scan.
{: .prompt-info }

## Security notes

- **Port scanning** reads which ports answer to find services.
- **SYN flood** sends many SYNs and never completes the handshake, filling the server's half-open table. Defenses: SYN cookies and rate limits.
- Close services you do not need and use a firewall.

## Lab

1. Start a Wireshark capture and visit a website.
2. Apply `tcp.flags.syn==1` and find the three handshake packets.
3. Compare the sequence and acknowledgment numbers with Figure 8.
4. Run `ss -tuln` and list the services your machine exposes.

## Review questions

1. Why does DNS usually use UDP?
2. In the handshake, which side sends the first SYN?
3. What does a `filtered` port mean in an Nmap scan?
4. Name the three packets of the handshake in order.

<details markdown="1">
<summary>Show answers</summary>

1. Queries and answers are small and need speed; UDP avoids the connection setup cost (TCP is used for large answers).
2. The client.
3. A firewall is dropping or blocking the probe, so no answer came back.
4. SYN, SYN-ACK, ACK.

</details>
