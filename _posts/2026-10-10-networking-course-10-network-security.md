---
title: "Networking Course 10: Network Security Fundamentals"
date: 2026-10-10 18:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, security, cia-triad, cryptography]
---
**Goal:** connect everything you learned to defense.

## The CIA triad

| Goal | Meaning | Example attack | Example control |
|---|---|---|---|
| Confidentiality | only authorized people read the data | eavesdropping | encryption (TLS, VPN) |
| Integrity | data is not changed secretly | man-in-the-middle tampering | hashes, signatures |
| Availability | the service works when needed | DDoS | redundancy, rate limits |

## Defense in depth

No single control is enough. Use **several layers**, so if one fails the next one still protects you.


![Concentric layers of defense from policies to data](/assets/img/posts/networking/fig-12-defense-in-depth.svg){: w="700" }
_Figure 12: defense in depth. The data you want to protect sits in the center._

## Threats and defenses by layer

| Threat | Layer | Main defense |
|---|---|---|
| ARP spoofing | 2 | Dynamic ARP Inspection |
| MAC flooding | 2 | port security |
| IP spoofing | 3 | ingress and egress filtering |
| ICMP abuse | 3 | rate limit or filter |
| SYN flood / DDoS | 4 | SYN cookies, rate limits, CDN |
| DNS poisoning | 7 | DNSSEC, trusted resolvers |
| Man-in-the-middle | 2 to 7 | TLS, certificate validation, VPN |
| Phishing | 7 | training, email filtering, MFA |

## Worked scenario: ARP spoofing on a LAN

1. **Normal:** PC A asks "who has 192.168.1.1?" and the real router answers.
2. **Attack:** a malicious host sends fake ARP replies: "192.168.1.1 is at my MAC".
3. **Result:** A sends its traffic to the attacker, who can read or modify it (man-in-the-middle).
4. **Signs:** two IP addresses share one MAC in `arp -a`; Wireshark shows many unsolicited ARP replies.
5. **Defense:** Dynamic ARP Inspection and DHCP snooping on the switch, HTTPS/VPN so captured traffic is useless.

Example of a suspicious `arp -a` result:

```
192.168.1.1   at 66:77:88:aa:bb:cc
192.168.1.99  at 66:77:88:aa:bb:cc     <- same MAC for two addresses
```

## Cryptography basics

| Type | Idea | Examples |
|---|---|---|
| Symmetric | one shared key encrypts and decrypts | AES |
| Asymmetric | public key encrypts, private key decrypts | RSA, ECC |
| Hash | one-way fingerprint | SHA-256 |
| Signature | proves who created data and that it is unchanged | RSA / ECDSA signatures |

A hash changes completely if the input changes by one character:

```bash
echo -n "hello" | sha256sum
2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
```

**TLS in short:** the server shows a certificate (signed by a trusted authority), both sides agree on keys using asymmetric cryptography, then the data is encrypted with a fast symmetric key.

## Good habits

- Patch systems and firmware regularly.
- Use unique passwords and **MFA** everywhere you can.
- Apply **least privilege**: give only the access needed.
- Segment the network (VLANs, DMZ) and filter between segments.
- Encrypt in transit (HTTPS, SSH, VPN) and keep **logs** you actually review.
- Make **backups** and test restoring them.


> Learn attacks only to defend. Practice in your own lab, on CTF platforms, or with written permission.
{: .prompt-warning }

## Lab

1. Open Wireshark and capture while you browse an HTTP-only test site and an HTTPS site. Compare what you can read.
2. Compute the SHA-256 of a text file, change one letter, and compute it again.
3. Pick your home or lab network and write which control sits in each ring of Figure 12.

## Review questions

1. Which part of the CIA triad does a DDoS attack target?
2. Why does TLS use both asymmetric and symmetric encryption?
3. How can you spot ARP spoofing in an `arp -a` listing?
4. What is the idea behind least privilege?

<details markdown="1">
<summary>Show answers</summary>

1. Availability.
2. Asymmetric cryptography safely agrees on keys; symmetric encryption is much faster for the data itself.
3. Several IP addresses (especially the gateway) share the same MAC address.
4. Users and programs get only the permissions they need, which limits the damage of a mistake or compromise.

</details>
