# Network Traffic Analysis - HackTheBox

**Platform:** HackTheBox Academy (Module 81)
**Author:** Brown Samita | Nairobi, Kenya
**Achievement:** [academy.hackthebox.com/achievement/2127127/81](https://academy.hackthebox.com/achievement/2127127/81)

## Introduction

This module focuses on the practical application of Network Traffic Analysis (NTA) to observe, filter, and analyze network communications in order to identify abnormal and potentially malicious activity. By examining live and captured traffic, the lab demonstrates how attackers inevitably interact with network infrastructure, and how these interactions can be detected through careful analysis of ports, protocols, and packet behavior.

Traffic was ingested from a network segment and progressively refined using filtering techniques to reduce noise and highlight relevant data. Analysis centered on identifying deviations from normal network behavior, such as non-standard port usage, unusual host communication, protocol anomalies, and indicators of reconnaissance or exploitation attempts.

The module also reinforces core networking concepts, including the TCP/IP stack, common protocols, and packet structure, while providing hands-on experience with traffic analysis tools and filtering syntax. Following a structured NTA workflow (ingestion, filtering, analysis, detection, monitoring) builds the skills required to effectively investigate network-based threats and support incident response and threat hunting.

**Lab objectives:**
- Master TCPdump filtering syntax
- Identify TCP three-way handshake components
- Analyze DNS query patterns
- Extract specific network information from packet captures
- Reconstruct HTTP sessions and extract transferred files with Wireshark
- Decrypt and examine RDP traffic

**Tools used:** TCPdump, Wireshark (Follow TCP Stream, Statistics, Export Objects), TShark

---

## Part 1: TCPdump Fundamentals

TCPdump is one of the most powerful command-line packet analyzers available on Unix-like systems. This section covers reading PCAP files, applying capture and display filters, and interpreting packet flags directly from the terminal.

Key syntax covered:
- `tcpdump -r <file>.pcap` reads from a saved capture
- `-v` / `-vv` increases output verbosity
- `-w` writes captured output to a file
- `-X` shows packet contents in both Hex and ASCII
- Filtering by `host <ip>` shows traffic to or from a specific address
- Logical operators (`and`, `or`) combine filter conditions
- TCPdump resolves IPs to hostnames by default

## Part 2: Interrogating Network Traffic With Capture and Display Filters

This walkthrough uses a pre-captured PCAP file (`TCPDump-lab-2.pcap`) to answer key questions about network behavior, including identifying TCP handshakes and DNS infrastructure. The ability to filter and interpret this data is crucial for network forensics and understanding how applications communicate across networks.

### Question: What are the client and server port numbers used in the first full TCP three-way handshake?

**Step 1 — Read the file**

The capture was loaded with `tcpdump -r TCPDump-lab-2.pcap` to review the raw traffic and locate SYN, SYN-ACK, and ACK flag sequences.

**Step 2 — Filter on the identified port**

Based on the flag sequence observed (SYN, SYN-ACK, ACK), the handshake was isolated by filtering on port 43806:

```bash
tcpdump -r TCPDump-lab-2.pcap port 43806
```

This confirmed the three-way handshake between port 80 (server) and port 43806 (client).

**Answer:** `80 43806`

### Question: Based on the traffic seen in the PCAP file, who is the DNS server in this network segment?

Filtering for DNS traffic on port 53 showed repeated query/response pairs originating from a single internal host:

```bash
tcpdump -r TCPDump-lab-2.pcap port 53
```

**Answer:** `172.16.146.1`

### Conclusion

Through this exercise, TCPdump was used to:

1. **Identify the TCP three-way handshake** — the first complete handshake occurred between port 53 (DNS server) and port 43806 (client), showcasing the fundamental connection establishment process in TCP communications.
2. **Determine the DNS infrastructure** — analysis revealed that `172.16.146.1` serves as the DNS server for this network segment, handling domain name resolution requests from clients.

---

## Part 3: Wireshark — Packet Inception, Dissecting Network Traffic

### Introduction

This section builds hands-on experience analyzing network traffic using Wireshark. The lab involves examining a provided PCAP file containing data from an unencrypted HTTP session, allowing application-layer content to be inspected directly.

Within the captured traffic, an image was transmitted and suspected to contain hidden messages, indicating improper network usage. Filters were applied to locate relevant packets, extract the embedded image, and preserve it as evidence — demonstrating core packet analysis and network forensics skills used in real-world security investigations.

### Methodology

1. Loaded the capture in Wireshark and applied the `http` filter to surface all HTTP request/response traffic.
2. Narrowed the filter to `http && image-jfif` to isolate the packets containing JFIF (JPEG) image data.
3. Used **File > Export Objects > HTTP** to extract the transferred files from the stream.

Three image files were recovered from the capture: `htb.jpeg`, `Rise-Up.jpg`, and `water.jpg`. The file of interest, based on the lab's investigative scenario, was **Rise-Up.jpg**.

### Findings

- **Filename containing the "Transformer Leader" reference:** `Rise-Up.jpg`
- **Employee suspected of performing potentially malicious actions:** `bob`

---

## Part 4: Guided Lab — Traffic Analysis Workflow

### Introduction

The purpose of this analysis was to examine an unencrypted network capture (PCAP) to identify evidence of improper or malicious activity. Using Wireshark's **Follow TCP Stream** feature, the traffic was reconstructed to observe user commands, system information disclosure, and potential unauthorized actions performed during a remote session.

### Methodology

1. Loaded `guided-analysis.pcap` and applied a `tcp` filter to review the full session, including the 3-way handshake and the listening port used for the session.
2. Used **Follow TCP Stream** on the relevant conversation to reconstruct the plaintext session in order.
3. Read through the reconstructed stream to identify reconnaissance commands (`whoami`, `ipconfig`) and directory navigation (`cd c:\`, `dir`) performed by the attacker.
4. Continued through the stream to identify account creation and privilege escalation activity.

### Findings

Reconstructing the stream showed the attacker running standard information-gathering commands (`whoami`, `ipconfig`) before creating a new local user and adding it to the local Administrators group:

```
net user hacker PasswOrd1 /add
net localgroup administrators hacker /add
```

- **New user created on mrb3n's host:** `hacker`
- **Total packets in the Guided-analysis PCAP:** `44`
- **Suspicious port used:** `4444`

### Conclusion

The packet capture provides clear evidence of unauthorized access to a Windows system, where an attacker executed reconnaissance commands, gathered system information, and created a privileged backdoor account. The use of unencrypted communication enabled full visibility into the attack sequence, highlighting the risks of insecure remote access configurations and the importance of encryption, proper authentication controls, and continuous network monitoring to detect and prevent this type of activity.

---

## Part 5: Decrypting RDP Connections

### Introduction

This analysis focused on decrypting and examining an RDP session from a captured PCAP file using Wireshark. The goal was to validate whether sensitive information and authentication artifacts could be extracted from RDP traffic once proper decryption was applied, and to assess the visibility of user activity during remote access sessions.

### Methodology

1. Filtered the capture with `tcp.port==3389` to isolate the RDP session.
2. Observed the TLS handshake (Client Hello, Server Hello/Certificate, Change Cipher Spec, Encrypted Handshake Message) confirming the session was TLS-protected.
3. Applied the appropriate decryption configuration in Wireshark to decrypt the TLS layer and reveal the underlying CredSSP/NTLMSSP authentication exchange.
4. Reviewed the decrypted NTLMSSP_AUTH message to identify the account used to authenticate the RDP session.

### Findings

Once decrypted, the CredSSP negotiation exposed the NTLMSSP authentication flow, revealing the target hostname and the user account used to initiate the RDP connection: `DESKTOP-8BSUEVL\bucky`.

- **User account used to initiate the RDP connection:** `bucky`

### Conclusion

This analysis demonstrates that, with proper keys and configuration, RDP traffic can be decrypted to reveal valuable forensic details such as authentication flows, user context, and session behavior. While payload content remains protected by encryption, exposed metadata can still provide critical insights during incident response and investigations. This highlights the importance of securing RDP access, monitoring encrypted traffic intelligently, and limiting credential exposure through strong authentication and access controls.

---

## Overall Key Takeaways

- TCPdump filtering (host, port, flag-based) is sufficient to isolate handshakes, DNS activity, and specific conversations even without a GUI.
- Wireshark's Export Objects feature can recover files (including images) transferred in the clear over HTTP — useful for identifying data exfiltration or policy violations.
- Follow TCP Stream reconstruction turns raw packets into a readable command-by-command narrative of attacker activity, making it a core technique for incident response.
- Even encrypted protocols like RDP leak useful metadata (usernames, hostnames) during authentication if decryption keys are available, reinforcing the need for strong credential hygiene and TLS configuration review.

## Defensive Recommendations

- Enforce encrypted protocols (HTTPS, RDP over TLS with NLA) across the network and monitor for legacy unencrypted services still in use.
- Monitor for anomalous local account creation and additions to privileged groups (e.g. `net user /add`, `net localgroup administrators /add`) as a high-fidelity detection opportunity.
- Baseline expected DNS servers and alert on DNS queries to unauthorized or unexpected resolvers.
- Restrict and monitor RDP access with network-level authentication, MFA, and jump-host architectures to reduce exposure of remote access services.
- Regularly capture and review internal traffic samples to validate that sensitive data (files, credentials) is not being transmitted in plaintext.
