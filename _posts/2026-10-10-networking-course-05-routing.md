---
title: "Networking Course 05: Routing"
date: 2026-10-10 13:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, routing, ospf]
---
**Goal:** understand how packets cross between networks and configure basic routes.

## What a router does

A router receives a packet, looks up the **destination IP address** in its **routing table**, and forwards the packet to the best **next hop**.


![Two LANs connected through two routers with their routing tables](/assets/img/posts/networking/fig-07-routing.svg){: w="700" }
_Figure 7: PC-A reaches PC-B only if each router knows the way to the other LAN._

## Reading a routing table

On Linux:

```bash
ip route
```

Example output:

```
default via 192.168.1.1 dev wlan0 proto dhcp metric 600
192.168.1.0/24 dev wlan0 proto kernel scope link src 192.168.1.23
```

- `192.168.1.0/24 ... scope link` is a **connected** network: your PC reaches it directly.
- `default via 192.168.1.1` is the **default route**: anything else goes to the gateway.

## The longest-prefix-match rule

If several routes match, the **most specific** (longest prefix) wins.

| Destination `10.1.2.5` matches | Prefix length |
|---|---|
| `10.0.0.0/8` | 8 |
| `10.1.0.0/16` | 16 |
| `10.1.2.0/24` | **24** (winner) |

## Static, dynamic and default routes

| Type | Idea | Good for |
|---|---|---|
| Static | you type each route | small, simple networks |
| RIP | distance-vector, counts hops (max 15) | learning, very old networks |
| OSPF | link-state, uses cost | companies and campuses |
| BGP | path-vector between organizations | the Internet |

## Worked example: static routes for Figure 7 (Cisco)

On R1 (to reach the LAN behind R2):

```
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

On R2 (to reach the LAN behind R1):

```
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

Check it:

```
R1# show ip route
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
C    10.0.0.0/30 is directly connected, GigabitEthernet0/1
S    192.168.2.0/24 [1/0] via 10.0.0.2
```

`C` = connected, `S` = static. A **default route** to the Internet looks like `ip route 0.0.0.0 0.0.0.0 <next-hop>`.

## Worked example: OSPF in 4 lines

```
R1(config)# router ospf 1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.0.0 0.0.0.3 area 0
```

Do the same on R2 with its own networks. The routers now learn each other's routes automatically (`O` in `show ip route`).

## Worked example: traceroute

```bash
traceroute 192.168.2.10
```

Example output:

```
 1  192.168.1.1   1.2 ms
 2  10.0.0.2      2.0 ms
 3  192.168.2.10  2.9 ms
```

Each line is one router (hop) on the path. Windows uses `tracert`.

## Lab

1. Build Figure 7 in Packet Tracer.
2. Before adding routes, ping PC-B from PC-A and note the failure.
3. Add the two static routes and ping again.
4. Run `tracert` from PC-A and match each hop to the figure.

## Review questions

1. What does the default route do?
2. Which route wins for `172.16.4.9`: `172.16.0.0/16` or `172.16.4.0/24`?
3. What does `S` mean in `show ip route`?
4. Which routing protocol connects different organizations on the Internet?

<details markdown="1">
<summary>Show answers</summary>

1. It sends all traffic that matches no other route to one next hop (usually the Internet gateway).
2. `172.16.4.0/24`, the longer prefix.
3. The route was configured statically.
4. BGP.

</details>
