---
title: "Networking Course 09: Wireless Networking"
date: 2026-10-10 17:00:00 +0100
categories: [Networking, Course]
tags: [networking, course, wifi, wireless, wpa3]
---
**Goal:** understand how Wi-Fi works and how to secure it.

## Wi-Fi basics

| Standard | Name | Bands | Remark |
|---|---|---|---|
| 802.11n | Wi-Fi 4 | 2.4 / 5 GHz | older |
| 802.11ac | Wi-Fi 5 | 5 GHz | common |
| 802.11ax | Wi-Fi 6 / 6E | 2.4 / 5 / 6 GHz | current |

- **SSID:** the network name. **BSSID:** the access point's MAC address.
- **2.4 GHz:** longer range, slower, crowded. Only channels **1, 6 and 11** do not overlap.
- **5 GHz / 6 GHz:** faster, shorter range, many more channels.
- An **access point** bridges Wi-Fi to the wired LAN. A **controller** manages many access points in companies.

## Security evolution

| Protocol | Status | Remark |
|---|---|---|
| WEP | broken | can be cracked in minutes, never use |
| WPA (TKIP) | weak | obsolete |
| **WPA2** (AES-CCMP) | good | Personal (shared passphrase) or Enterprise (802.1X) |
| **WPA3** | best | stronger key exchange, protects against offline guessing |

## Worked example: see nearby networks (your own environment)

```bash
nmcli dev wifi list
```

Example output:

```
IN-USE  SSID        CHAN  SIGNAL  SECURITY
*       HomeNet     6     82      WPA2
        Neighbor    6     60      WPA2
        CafeGuest   11    45      --
```

Reading it: `HomeNet` and `Neighbor` share channel 6, which causes interference (moving to channel 1 or 11 may help). `CafeGuest` has `--` in the security column: **an open network, anyone can read unencrypted traffic**.

## Common wireless risks

| Risk | Meaning | Defense |
|---|---|---|
| Open Wi-Fi sniffing | traffic is readable | VPN, HTTPS only |
| Evil twin | fake access point with a known SSID | check the network, use WPA-Enterprise with certificates |
| Rogue AP | unauthorized AP plugged into your LAN | monitor, port security |
| Deauthentication | forces clients off the network | WPA3 with protected management frames |
| Weak passphrase | guessable password | long random passphrase |

## Home router hardening checklist

1. Use WPA2-AES or WPA3 with a long passphrase (15+ characters).
2. Change the router's admin password and disable remote administration.
3. Disable **WPS**.
4. Keep the firmware updated.
5. Put guests and smart devices on a **guest network**.
6. Do not use hidden SSID as a security measure; it does not help.

## Lab

1. Log in to your own router and write down the Wi-Fi security mode, channel and band.
2. Run `nmcli dev wifi list` (or use a Wi-Fi analyzer app) and check whether you share a channel with neighbors.
3. Apply the checklist and note each change.


> Only test wireless security on networks you own or have written permission to test.
{: .prompt-warning }

## Review questions

1. Which 2.4 GHz channels do not overlap?
2. Why is an open Wi-Fi network risky?
3. Which is stronger, WPA2 or WPA3?
4. Why should IoT devices use a guest network?

<details markdown="1">
<summary>Show answers</summary>

1. 1, 6 and 11.
2. Traffic is not encrypted, so others on the network can capture it, and fake networks are easy to create.
3. WPA3.
4. If one device is compromised, it cannot reach your computers and servers.

</details>
