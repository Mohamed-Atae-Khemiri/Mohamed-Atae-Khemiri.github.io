---
title: "Networking Course 04: IP Addressing and Subnetting"
date: 2026-10-10 12:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, ip, subnetting]
---
**Goal:** read any IP address, calculate subnets quickly, and plan an addressing scheme.

## IPv4 basics

An IPv4 address has 32 bits written as 4 octets: `192.168.10.5`.

**Binary example:**

| Octet | 192 | 168 | 10 | 5 |
|---|---|---|---|---|
| Binary | 11000000 | 10101000 | 00001010 | 00000101 |

Special ranges:

| Range | Use |
|---|---|
| `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | private (not routed on the Internet) |
| `127.0.0.0/8` | loopback (`127.0.0.1` is "this machine") |
| `169.254.0.0/16` | APIPA: no DHCP answered |
| `224.0.0.0/4` | multicast |

## The subnet mask

The mask tells which bits are the **network part** and which are the **host part**.

| CIDR | Mask | Usable hosts |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /28 | 255.255.255.240 | 14 |
| /29 | 255.255.255.248 | 6 |
| /30 | 255.255.255.252 | 2 |


![A /24 network split into four /26 subnets with the binary mask](/assets/img/posts/networking/fig-06-subnetting.svg){: w="700" }
_Figure 6: one /24 split into four /26 subnets. The /26 mask has 26 network bits and 6 host bits._

**Formula:** usable hosts = 2^(host bits) - 2 (one address is the network, one is the broadcast).

## Worked example 1: find the network, broadcast and range

Address `192.168.10.77/26`.

1. /26 means 6 host bits, so each block has 2^6 = **64** addresses.
2. Blocks start at 0, 64, 128, 192. The address 77 falls in the block that starts at **64**.
3. Network `192.168.10.64`, broadcast `192.168.10.127`.
4. Usable range `192.168.10.65` to `192.168.10.126` (62 hosts).

## Worked example 2

Address `172.16.5.130/25`: block size 128, so the network is `172.16.5.128` and the broadcast is `172.16.5.255`.

## Worked example 3: VLSM (different sizes from one block)

You have `192.168.1.0/24` and need: Staff 100 hosts, Servers 50, Guests 25, Management 10. Assign **largest first**.

| Subnet | Hosts needed | Prefix | Network | Usable range | Broadcast |
|---|---|---|---|---|---|
| Staff | 100 | /25 (126) | 192.168.1.0 | .1 - .126 | .127 |
| Servers | 50 | /26 (62) | 192.168.1.128 | .129 - .190 | .191 |
| Guests | 25 | /27 (30) | 192.168.1.192 | .193 - .222 | .223 |
| Management | 10 | /28 (14) | 192.168.1.224 | .225 - .238 | .239 |

Nothing overlaps, and `192.168.1.240` to `.255` is still free for growth.

## IPv6 in short

An IPv6 address has 128 bits written in hexadecimal. Shorten it by removing leading zeros and replacing one run of zeros with `::`.

```
2001:0db8:0000:0000:0000:0000:0000:0001   becomes   2001:db8::1
```

| Type | Example | Use |
|---|---|---|
| Global unicast | `2001:db8::/32` range | routable addresses |
| Link-local | `fe80::/10` | local link, automatic |
| Multicast | `ff00::/8` | one-to-many |

There is no broadcast in IPv6 (multicast replaces it).

## Exercises

1. How many usable hosts are in `10.0.0.0/8`?
2. Find the network, broadcast and range of `192.168.1.77/28`.
3. Is `192.168.1.130` or `192.168.1.200` inside `192.168.1.128/26`?
4. What is the mask of a /20, and how many hosts does it hold?
5. Split `192.168.1.0/24` into 4 equal subnets and write all ranges.

<details markdown="1">
<summary>Show answers</summary>

1. 16,777,214 (2^24 - 2).
2. Block size 16: network `192.168.1.64`, broadcast `192.168.1.79`, range `.65` to `.78`.
3. `.130` is inside (range .128 to .191); `.200` is outside.
4. 255.255.240.0, with 4,094 usable hosts.
5. Four /26 subnets: .0-.63, .64-.127, .128-.191, .192-.255 (see Figure 6).

</details>

## Lab

Do the exercises on paper, then check each with `ipcalc` on Linux, for example `ipcalc 192.168.1.77/28`. Then set up two PCs in Packet Tracer in different /26 subnets and prove they need a router.
