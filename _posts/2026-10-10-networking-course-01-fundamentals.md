---
title: "Networking Course 01: Network Fundamentals"
date: 2026-10-10 09:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, basics]
---
**Goal:** understand what a network is, the devices in it, and how to measure it.

## What is a network?

A **network** is a group of devices (called **hosts**) connected so they can share data. A host is any device with an IP address: a PC, phone, printer or server.

**Network types**

| Type | Meaning | Example |
|---|---|---|
| PAN | personal area | phone with Bluetooth earbuds |
| LAN | local area | your home or office network |
| MAN | metropolitan area | a city-wide network |
| WAN | wide area | the Internet |
| WLAN | wireless LAN | Wi-Fi at home |

## Topologies


![Four network topologies: star, bus, ring and mesh](/assets/img/posts/networking/fig-01-topologies.svg){: w="700" }
_Figure 1: the four classic topologies. Star is what homes and offices use today._

| Topology | Advantage | Weakness |
|---|---|---|
| Star | one failed cable affects one device; easy to manage | the central switch is a single point of failure |
| Bus | cheap, little cable | one cable break stops everyone (old technology) |
| Ring | orderly access | one break can stop the loop |
| Mesh | very reliable (many paths) | expensive, many links |

## Network devices

| Device | Job |
|---|---|
| NIC | network card inside a host |
| Hub | repeats bits to every port (obsolete) |
| Switch | forwards frames inside a LAN using MAC addresses |
| Router | connects different networks using IP addresses |
| Access point | gives Wi-Fi devices access to the wired network |
| Firewall | allows or blocks traffic by rules |
| Modem | converts the provider's signal (fiber, DSL, cable) to Ethernet |

## Performance vocabulary

- **Bandwidth:** maximum capacity (for example 100 Mbit/s).
- **Throughput:** speed you actually get.
- **Latency:** time for a packet to travel (ms).
- **Jitter:** variation in latency (bad for calls and games).
- **Packet loss:** packets that never arrive.

## Worked example 1: look at your own host

On Linux:

```bash
ip a
```

Example output (shortened):

```
2: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 state UP
    link/ether 3c:52:82:aa:bb:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.23/24 brd 192.168.1.255 scope global dynamic wlan0
```

How to read it: `link/ether` is the **MAC address**, `inet` is the **IPv4 address** `192.168.1.23` with the **/24** mask, and `brd` is the broadcast address. On Windows, `ipconfig /all` shows the same information.

## Worked example 2: measure latency and loss

```bash
ping -c 4 192.168.1.1
```

Example output:

```
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=2.1 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=1.9 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=2.4 ms
64 bytes from 192.168.1.1: icmp_seq=4 ttl=64 time=3.0 ms

4 packets transmitted, 4 received, 0% packet loss
rtt min/avg/max/mdev = 1.9/2.3/3.0/0.4 ms
```

`time=` is the latency (about 2 ms inside a LAN), `0% packet loss` is healthy, and the `mdev` value is a rough measure of jitter. A `ttl` of 64 hints the target is a Linux or network device.

## Lab: map your home network

1. Draw your network: modem, router, and every device you own.
2. Write down each device's IP address (`ip a` / `ipconfig`) and your gateway address.
3. Ping the gateway, then ping `8.8.8.8`. Compare the latency.
4. Label your drawing: which topology is it?

## Review questions

1. What is the difference between a switch and a router?
2. What does "0% packet loss" tell you?
3. Which topology is used in most LANs today?
4. What is the difference between bandwidth and throughput?

<details markdown="1">
<summary>Show answers</summary>

1. A switch forwards frames inside one network using MAC addresses; a router connects different networks using IP addresses.
2. None of the test packets were lost, so the link is healthy.
3. Star.
4. Bandwidth is the maximum capacity; throughput is the real speed you get.

</details>
