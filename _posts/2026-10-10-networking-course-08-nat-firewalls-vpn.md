---
title: "Networking Course 08: NAT, Firewalls, VPN and Proxies"
date: 2026-10-10 16:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, firewall, vpn, nat, acl]
---
**Goal:** understand the services that connect networks to the Internet and protect them.


![A firewall with NAT, a DMZ web server and an internal LAN](/assets/img/posts/networking/fig-11-nat-dmz.svg){: w="700" }
_Figure 11: the firewall translates addresses (NAT) and separates the DMZ from the internal LAN._

## NAT and PAT

Private addresses cannot be routed on the Internet. **NAT** (here **PAT**, "overload") lets many private hosts share one public address by rewriting the source address and port.

Example NAT table:

| Inside (private) | Outside (public) |
|---|---|
| 192.168.1.10:51000 | 203.0.113.5:40001 |
| 192.168.1.11:51022 | 203.0.113.5:40002 |
| 192.168.1.12:49880 | 203.0.113.5:40003 |

Replies come back to the public address and port, and the router translates them back using this table. NAT also hides the internal layout, but it is **not** a firewall by itself.

## Firewalls

| Type | Looks at | Remark |
|---|---|---|
| Packet filter (stateless) | addresses, ports, flags of each packet | fast, simple |
| Stateful | tracks connections | allows replies automatically |
| Application / next-generation | content, users, applications | deepest inspection |

Best practice: **default deny**, then allow only what is needed.

### Worked example: ACL on a Cisco router

Goal: guests (VLAN 30) may use the Internet but not reach staff or server VLANs.

```
ip access-list extended GUEST-IN
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip 192.168.30.0 0.0.0.255 any

interface gigabitEthernet 0/0.30
 ip access-group GUEST-IN in
```

The router reads the lines **top to bottom** and stops at the first match. There is an invisible `deny any` at the end.

### Worked example: Linux firewall with ufw

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

Example status output:

```
Status: active
To          Action  From
22/tcp      ALLOW   Anywhere
443/tcp     ALLOW   Anywhere
```

## DMZ

The **DMZ** holds servers that must be reachable from the Internet (web, mail). If one of them is compromised, the attacker is still outside the internal LAN. Rules from Figure 11: Internet to DMZ web ports only, LAN to Internet allowed, DMZ to LAN denied.

## VPN

A **VPN** builds an encrypted tunnel across an untrusted network.

| Type | Use |
|---|---|
| Site-to-site | joins two office networks |
| Remote access | one user connects to the company network |

Common protocols: IPsec, OpenVPN, WireGuard.

### Worked example: a WireGuard client file

```ini
[Interface]
PrivateKey = <your-private-key>
Address = 10.8.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server-public-key>
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

`AllowedIPs = 0.0.0.0/0` sends all traffic through the tunnel. Never share your private key.

## Proxies, load balancers and IDS/IPS

| Tool | Job |
|---|---|
| Proxy | clients reach the Internet through it (filtering, caching) |
| Reverse proxy | stands in front of servers (TLS, caching, protection) |
| Load balancer | spreads traffic over several servers |
| IDS | detects attacks and raises alerts |
| IPS | detects and blocks attacks inline |

## Lab

1. In Packet Tracer build Figure 11 (a router/firewall, a DMZ web server and a LAN).
2. Configure NAT overload on the router.
3. Add an ACL so the Internet can reach only the DMZ web server on port 80.
4. Test from the LAN and from the outside and note which pings succeed.

## Review questions

1. What problem does NAT solve?
2. Why is a DMZ safer than putting a web server on the LAN?
3. In an ACL, what happens when a packet matches no line?
4. What is the difference between an IDS and an IPS?

<details markdown="1">
<summary>Show answers</summary>

1. It lets many private hosts share limited public IPv4 addresses.
2. If the server is compromised, the attacker is still separated from the internal LAN by the firewall.
3. It is denied (implicit deny at the end).
4. An IDS only detects and alerts; an IPS can also block the traffic.

</details>
