---
title: "Networking Course 07: Core Application Protocols"
date: 2026-10-10 15:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, dns, dhcp, http]
---
**Goal:** understand the protocols you use every day and how to inspect them.

## DNS: names to addresses

DNS turns `www.example.com` into an IP address.


![DNS resolution through resolver, root, TLD and authoritative servers](/assets/img/posts/networking/fig-09-dns.svg){: w="700" }
_Figure 9: the recursive resolver asks the root, then the TLD, then the authoritative server._

Common record types:

| Record | Meaning | Example |
|---|---|---|
| A | name to IPv4 | `example.com. 3600 IN A 203.0.113.10` |
| AAAA | name to IPv6 | `example.com. IN AAAA 2001:db8::10` |
| CNAME | alias | `www.example.com. IN CNAME example.com.` |
| MX | mail server | `example.com. IN MX 10 mail.example.com.` |
| NS | name server | `example.com. IN NS ns1.example.com.` |
| TXT | free text (SPF, verification) | `example.com. IN TXT "v=spf1 ..."` |
| PTR | IP to name (reverse) | `10.113.0.203.in-addr.arpa. IN PTR example.com.` |

### Worked example: dig

```bash
dig example.com A
```

Example output (shortened):

```
;; ANSWER SECTION:
example.com.   3600  IN  A  203.0.113.10

;; Query time: 24 msec
;; SERVER: 192.168.1.1#53
```

`3600` is the TTL in seconds (how long it can be cached), `SERVER` shows which resolver answered. `nslookup example.com` gives the same answer in a simpler format.

## DHCP: automatic addressing


![DHCP Discover, Offer, Request and Acknowledge exchange](/assets/img/posts/networking/fig-10-dhcp.svg){: w="700" }
_Figure 10: DORA. The first message is a broadcast because the client has no address yet._

A DHCP lease normally gives: IP address, subnet mask, default gateway, DNS servers and the lease time. If no server answers, Windows and Linux fall back to a `169.254.x.x` address.

## HTTP and HTTPS

### Worked example: an HTTP request and response

Request:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: curl/8.4
```

Response (shortened):

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256
```

| Code | Meaning |
|---|---|
| 200 | OK |
| 301 / 302 | redirect |
| 401 / 403 | not authenticated / forbidden |
| 404 | not found |
| 500 | server error |

See the headers of any site:

```bash
curl -I https://example.com
```

**HTTPS** is HTTP inside a **TLS** tunnel. TLS gives encryption, server identity (certificate) and integrity.

## Other protocols

| Protocol | Port | Use | Safer choice |
|---|---|---|---|
| SMTP | 25 / 587 | send email | use TLS (587) |
| POP3 / IMAP | 110 / 143 | read email | IMAPS 993 |
| FTP | 21 | file transfer, clear text | SFTP or FTPS |
| Telnet | 23 | remote shell, clear text | SSH (22) |
| SMB | 445 | Windows file sharing | keep internal, patch |
| SNMP | 161 | device monitoring | SNMPv3 |
| NTP | 123 | time sync | trusted servers |

## Lab

1. Run `dig` and `nslookup` for a domain you own or a public one.
2. In Wireshark, filter `dns`, find the query and its answer, and match the IP with `dig`.
3. Release and renew your IP address (`sudo dhclient -r && sudo dhclient` on Linux, `ipconfig /release` then `/renew` on Windows) while capturing with the filter `bootp` (or `dhcp`) and find the four DORA packets.

## Review questions

1. What does the TTL in a DNS answer control?
2. Why is the first DHCP message a broadcast?
3. What is the difference between a 403 and a 404 response?
4. Why should you use SSH instead of Telnet?

<details markdown="1">
<summary>Show answers</summary>

1. How long resolvers may cache the answer.
2. The client does not have an IP address yet and does not know the server address.
3. 403 means access is forbidden; 404 means the resource does not exist.
4. Telnet sends everything, including passwords, in clear text; SSH encrypts the session.

</details>
