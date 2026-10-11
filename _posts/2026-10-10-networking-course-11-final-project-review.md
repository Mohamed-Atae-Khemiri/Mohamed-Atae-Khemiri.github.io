---
title: "Networking Course 11: Final Project and Review Questions"
date: 2026-10-10 19:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, project, quiz]
---
## Final project: small company network

Build this network in Packet Tracer.


![A small company network with three VLANs, a router and a DMZ web server](/assets/img/posts/networking/fig-13-final-project.svg){: w="700" }
_Figure 13: the final project topology._

### Requirements

- 3 VLANs: Staff, Servers, Guests, with inter-VLAN routing.
- NAT to the Internet.
- A DMZ web server reachable from outside on port 80.
- Guests can reach the Internet but not VLAN 10 or VLAN 20.
- DHCP for staff and guests, DNS on the server VLAN.

### Addressing plan

| Network | Subnet | Gateway | Notes |
|---|---|---|---|
| VLAN 10 Staff | 192.168.10.0/24 | 192.168.10.1 | DHCP |
| VLAN 20 Servers | 192.168.20.0/24 | 192.168.20.1 | DNS, DHCP server at .10 |
| VLAN 30 Guests | 192.168.30.0/24 | 192.168.30.1 | Internet only |
| DMZ | 10.0.50.0/24 | 10.0.50.1 | web server at .10 |
| Internet link | 203.0.113.0/24 | provider | router outside IP .5 |

### Step 1: switch

```
Switch(config)# vlan 10
Switch(config-vlan)# name STAFF
Switch(config-vlan)# vlan 20
Switch(config-vlan)# name SERVERS
Switch(config-vlan)# vlan 30
Switch(config-vlan)# name GUESTS
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# switchport mode trunk
```

Then put each PC port in its VLAN with `switchport mode access` and `switchport access vlan X`.

### Step 2: router with sub-interfaces (router-on-a-stick)

```
interface gigabitEthernet 0/0
 no shutdown
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
 ip access-group GUEST-IN in
```

### Step 3: DMZ and Internet interfaces

```
interface gigabitEthernet 0/2
 ip address 10.0.50.1 255.255.255.0
 no shutdown
interface gigabitEthernet 0/1
 ip address 203.0.113.5 255.255.255.0
 ip nat outside
 no shutdown
```

### Step 4: NAT and the guest ACL

```
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 interface gigabitEthernet 0/1 overload

ip access-list extended GUEST-IN
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip 192.168.30.0 0.0.0.255 any
```

### Test checklist

| Test | Expected |
|---|---|
| Staff PC pings the server VLAN gateway | success |
| Staff PC pings a server in VLAN 20 | success |
| Guest PC pings a staff PC | **fails** (ACL) |
| Guest PC pings the Internet host | success |
| Outside host opens the DMZ web page | success |
| DMZ server pings a staff PC | should fail once you add a DMZ ACL |

Write a one-page report: your diagram, your addressing plan, your tests and what you would improve.

### Grading checklist (self-assessment)

- [ ] Correct subnets without overlap
- [ ] VLANs and trunk working
- [ ] NAT working
- [ ] ACL tested in both directions
- [ ] DMZ reachable from outside only on the allowed port
- [ ] Documentation complete

## Final review: 15 questions

1. List the 7 OSI layers and one protocol for each.
2. What happens during a three-way handshake?
3. How many usable hosts does a /28 have?
4. What is the difference between a hub, a switch and a router?
5. What does ARP do, and how can it be abused?
6. What is the difference between TCP and UDP? Give 2 uses each.
7. Explain the DNS resolution process step by step.
8. What is DORA?
9. What does NAT solve?
10. Explain stateful vs stateless firewalls.
11. Why is Telnet unsafe, and what replaces it?
12. What is a VLAN, and why use one?
13. What is the purpose of a DMZ?
14. How do WPA2 and WPA3 differ?
15. Name 5 defense-in-depth measures.

<details markdown="1">
<summary>Show answers</summary>

1. Application (HTTP), Presentation (TLS), Session (RPC), Transport (TCP), Network (IP), Data Link (Ethernet), Physical (cables).
2. SYN from the client, SYN-ACK from the server, ACK from the client; then data can flow.
3. 14 (2^4 - 2).
4. A hub repeats bits to all ports; a switch forwards frames by MAC address; a router forwards packets between networks by IP address.
5. It maps an IP address to a MAC address; it can be abused by spoofing replies (man-in-the-middle).
6. TCP is reliable and connection-oriented (web, SSH); UDP is fast and connectionless (DNS, VoIP).
7. The resolver asks the root server, then the TLD server, then the authoritative server, caches the answer and returns it to the client.
8. Discover, Offer, Request, Acknowledge: the four DHCP messages.
9. It lets many private hosts share limited public IPv4 addresses.
10. A stateless filter checks each packet alone; a stateful firewall tracks connections and allows return traffic automatically.
11. Telnet sends data, including passwords, in clear text; SSH replaces it with encryption.
12. A VLAN is a logical network on a switch; it separates broadcast domains, improves security and organization.
13. To isolate public-facing servers from the internal LAN.
14. WPA3 has a stronger key exchange and protects against offline password guessing; WPA2 is still good with a strong passphrase.
15. Segmentation, firewalls, IDS/IPS, patching, MFA, least privilege, encryption, logging, backups (any five).

</details>
