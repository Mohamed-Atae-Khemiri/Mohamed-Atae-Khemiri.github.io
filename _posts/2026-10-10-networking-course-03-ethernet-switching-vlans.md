---
title: "Networking Course 03: Ethernet, Switching and VLANs"
date: 2026-10-10 11:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, switching, vlan, arp]
---
**Goal:** understand how devices talk inside one local network and how to split a network into VLANs.

## MAC addresses and Ethernet frames

A **MAC address** is a 48-bit hardware address written like `3C:52:82:AA:BB:CC`. The first three bytes identify the manufacturer (the **OUI**).


![Ethernet frame fields and a switch MAC address table](/assets/img/posts/networking/fig-04-ethernet-switch.svg){: w="700" }
_Figure 4: the fields of an Ethernet frame, and how a switch uses its MAC address table._

## How a switch learns

1. PC A (`AA:..:01`) sends a frame to PC C. The switch records "AA:..:01 is on port 1".
2. If the switch does not know C yet, it **floods** the frame out every other port.
3. C replies. The switch records "AA:..:03 is on port 3".
4. From now on, frames for C go out **port 3 only**.

A **hub** would send every frame to every port. A switch gives each port its own **collision domain**, but all ports in the same VLAN still share one **broadcast domain**.

## ARP: from IP address to MAC address

To send to `192.168.1.1`, your PC must know its MAC address. It broadcasts "Who has 192.168.1.1?" and the owner replies. The answers are cached:

```bash
arp -a
```

Example output:

```
? (192.168.1.1) at 00:1a:2b:3c:4d:5e [ether] on wlan0
? (192.168.1.40) at a4:5e:60:11:22:33 [ether] on wlan0
```

## VLANs

A **VLAN** splits one physical switch into separate logical networks.


![Two switches with VLAN 10 and VLAN 20 joined by a trunk link](/assets/img/posts/networking/fig-05-vlans.svg){: w="700" }
_Figure 5: VLAN 10 (blue) and VLAN 20 (orange) share two switches through one trunk link._

- **Access port:** belongs to one VLAN (connects a PC).
- **Trunk port:** carries several VLANs between switches using **802.1Q** tags.
- **STP** (Spanning Tree Protocol) stops switching loops.

## Worked example: configure VLANs on a Cisco switch (Packet Tracer)

```
Switch> enable
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name STAFF
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name SERVERS
Switch(config-vlan)# exit

Switch(config)# interface fastEthernet 0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# exit

Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# end
Switch# show vlan brief
```

Example result of `show vlan brief` (shortened):

```
VLAN Name      Status    Ports
10   STAFF     active    Fa0/1, Fa0/2
20   SERVERS   active    Fa0/3, Fa0/4
```

## Security at layer 2

| Attack | What it does | Defense |
|---|---|---|
| MAC flooding | fills the switch table so it acts like a hub | port security |
| ARP spoofing | fake ARP replies redirect traffic | Dynamic ARP Inspection |
| VLAN hopping | jumps into another VLAN | disable unused trunks, set access mode explicitly |
| rogue DHCP server | hands out a wrong gateway | DHCP snooping |

Port security example:

```
Switch(config)# interface fastEthernet 0/5
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 2
Switch(config-if)# switchport port-security violation shutdown
```

## Lab (Packet Tracer)

1. Build 2 switches and 4 PCs as in Figure 5.
2. Create VLAN 10 and VLAN 20, assign ports, make the link between switches a trunk.
3. Give PCs addresses in `192.168.10.0/24` and `192.168.20.0/24`.
4. Ping inside each VLAN (works) and between VLANs (fails). Explain why.

## Review questions

1. What does a switch do when it receives a frame for an unknown MAC address?
2. What is the difference between an access port and a trunk port?
3. Why do two PCs in different VLANs need a router to talk?
4. Name two defenses against ARP spoofing.

<details markdown="1">
<summary>Show answers</summary>

1. It floods the frame out every port except the one it arrived on.
2. An access port carries one VLAN for an end device; a trunk carries several VLANs between switches.
3. Each VLAN is a separate broadcast domain and IP subnet, so traffic between them must be routed (layer 3).
4. Dynamic ARP Inspection and static ARP entries (also DHCP snooping).

</details>
