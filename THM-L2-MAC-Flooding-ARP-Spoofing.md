# TryHackMe: L2 MAC Flooding & ARP Spoofing

**Platform:** TryHackMe | **Room:** [Layer2](https://tryhackme.com/room/layer2)  
**Author:** Brown Samita | **Date:** February 26, 2026  

---

## Overview

This write-up documents hands-on completion of the TryHackMe "L2 MAC Flooding & ARP Spoofing" room. The lab explores attack techniques targeting Layer 2 of the OSI model — specifically exploiting weaknesses in the MAC address learning mechanism of network switches (MAC flooding) and the stateless, trust-based nature of ARP (ARP spoofing).

**Key concepts covered:**
- MAC flooding to degrade switch behaviour and enable passive eavesdropping
- ARP spoofing to achieve Man-in-the-Middle (MITM) positioning
- Credential interception over cleartext HTTP
- Defensive controls and mitigations

**Tools used:** `nmap` • `tcpdump` • `macof` • `ettercap` • `curl` • `Wireshark`

---

## Lab Environment

| Host | IP Address | Role |
|------|-----------|------|
| eve  | 192.168.12.66 | Attacker machine |
| alice | 192.168.12.10 / 192.168.12.1 | Victim host |
| bob  | 192.168.12.20 / 192.168.12.2 | Victim host (HTTP service on port 80) |

---

## 1. Network Reconnaissance & Host Discovery

After elevating privileges with `sudo su -`, confirmed the attacker's IP and subnet:

```bash
ip address show eth1
# Result: 192.168.12.66/24
```

`arp-scan` was unavailable, so used Nmap for a ping sweep:

```bash
sudo nmap -sn 192.168.12.0/24
```

**Results:** Three live hosts discovered — alice (192.168.12.1), bob (192.168.12.2), and eve (192.168.12.66).

---

## 2. Passive Network Sniffing

Captured traffic passively on eth1:

```bash
sudo tcpdump -i eth1
```

In a properly functioning switched network, only unicast traffic addressed to the host and broadcast/multicast frames are visible. Observed ICMP echo requests/replies between hosts and ARP messages. Bob was continuously sending ICMP packets to eve with a **data section size of 666 bytes**. Traffic was limited at this stage — the switch was operating normally and selectively forwarding frames.

---

## 3. Sniffing While MAC Flooding

To gain visibility into traffic not destined for the attacker's machine, MAC flooding was performed using ettercap's `rand_flood` plugin. This exhausts the switch's CAM table with fake MAC entries, causing it to fall back to broadcast mode (behaving like a hub) and exposing all traffic.

```bash
ettercap -T -i eth1 -P rand_flood -q -w /tmp/tcpdump3.pcap
```

With flooding active and a parallel packet capture running, traffic between alice and bob became visible. Alice was observed sending ICMP packets to bob with a **data section size of 1337 bytes**.

> **Note:** MAC flooding is a noisy technique that causes network-wide disruption — detectable by network monitoring tools.

---

## 4. Man-in-the-Middle: ARP Spoofing Introduction

ARP spoofing is a more targeted and stealthy MITM technique. Unlike MAC flooding, it surgically poisons the ARP caches of specific hosts, making them believe the attacker's MAC address belongs to the other party's IP.

**Scenario 1 — Hosts with ARP validation enabled:**

```bash
ettercap -T -i eth1 -M arp
```

ettercap could **not** establish a MITM position. The hosts' ARP validation rejected the poisoned replies — reinforcing that OS-level ARP validation can render this attack ineffective.

---

## 5. Man-in-the-Middle: Sniffing

Switched to a new Ubuntu VM with default ARP implementation and no protective controls — a realistic representation of many unprotected enterprise endpoints.

```bash
sudo nmap -sN 192.168.12.10
sudo nmap -sN 192.168.12.20
```

Bob (192.168.12.20) had **port 80/tcp open** (HTTP). Passive sniffing produced no meaningful traffic to port 80 until ARP spoofing was launched:

```bash
sudo ettercap -T -i eth1 -M arp
```

**Intercepted session revealed:**
- Alice making an HTTP GET request to `www.server.bob` for `test.txt`
- Basic Authentication credentials transmitted in **cleartext**
- Credentials captured: `admin:s3cr3t_P4zz`
- File content: `OK`

---

## 6. Man-in-the-Middle: Manipulation & Flag Retrieval

Using the captured credentials to directly access the HTTP service:

```bash
curl -u admin:s3cr3t_P4zz http://192.168.12.20/
```

Server returned a directory listing: `SimpleHTTPAuthServer.py`, `test.txt`, `user.txt`.

```bash
curl -u admin:s3cr3t_P4zz http://192.168.12.20/user.txt
# Flag: THM{wh0s_$n!ff1ng_0ur_cr3ds}
```

Additionally, intercepted traffic revealed alice had a reverse shell running to the server, executing `whoami`, `pwd`, and `ls` in sequence.

When ettercap was stopped (`q`), it gracefully restored original ARP tables by **RE-ARPing the victims**.

---

## 7. Conclusion & Key Takeaways

This lab demonstrated an end-to-end Layer 2 attack chain — from passive reconnaissance through MAC flooding to a full MITM position with credential interception.

| Takeaway | Detail |
|----------|--------|
| Switches are not inherently secure | CAM table exhaustion degrades them to hub behaviour |
| ARP has no built-in authentication | Stateless, trust-based — inherently susceptible to spoofing |
| Cleartext protocols are dangerous | HTTP transmits credentials in the open under any MITM position |
| ARP validation matters | OS-level protection completely neutralised the attack in Scenario 1 |

---

## Defensive Measures

- **Dynamic ARP Inspection (DAI)** on managed switches
- **ARP packet validation** enabled on endpoints
- **Encrypted protocols** (HTTPS, SSH) to protect credentials in transit
- **802.1X port authentication** to prevent unauthorised devices joining the network segment

---

## Room Completion

✅ Room successfully completed: [https://tryhackme.com/room/layer2](https://tryhackme.com/room/layer2)
