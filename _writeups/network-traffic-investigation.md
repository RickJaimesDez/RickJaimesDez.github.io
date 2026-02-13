---
title: "Network Traffic Investigation"
---

## Overview
**Goal:** Identify the infected host, confirm malicious payload delivery, analyze encrypted follow-on communications, and extract actionable indicators of compromise from a public malware PCAP.

**Environment:** Windows 10 virtual machine (VirtualBox) for packet analysis, Kali Linux virtual machine for secure file hashing, Wireshark for traffic inspection, and sha256sum for payload verification.

**Data/Evidence:** 2025-06-20-traffic-from-running-the-malware.pcap (Malware-Traffic-Analysis dataset)

---

## Scenario
A packet capture was analyzed to investigate suspected malware activity originating from an internal host. The objective was to determine whether malicious payload delivery occurred, identify potential command-and-control (C2) behavior, and assess the scope of impact within the network. The analysis focused on outbound HTTP and TLS communications to suspicious external infrastructure. Findings were documented using a structured SOC investigation approach.

---

## Investigation Methodology
1. Identified the primary external IP address communicating with the internal network.
2. Filtered HTTP traffic to determine whether binary payload retrieval occurred.
3. Correlated TLS Client Hello messages to identify encrypted outbound communications and SNI values.
4. Determined whether additional internal hosts were communicating with the malicious infrastructure.
5. Extracted and handled the downloaded payload for validation and hashing in a controlled environment.

---

## Evidence

HTTP GET request for /shrk.bin and corresponding 200 OK response confirming successful binary payload delivery to 172.16.1.128.
![HTTP GET](/assets/images/http-get.png)

Repeated TLS Client Hello sessions from 172.16.1.128 to b1.encountergulf.world, indicating automated encrypted outbound communication consistent with beaconing behavior.
![TLS Beaconing](/assets/images/tls-beaconing.png)

Filtered view of all traffic involving 104.21.21.29, confirming that 172.16.1.128 is the only internal system communicating with the malicious infrastructure within the capture scope.
![IP Filter View](/assets/images/ip-filter.png)

---

## Findings

- **Finding 1 – Successful Payload Delivery:**  
At 12:26:14, host 172.16.1.128 initiated an HTTP GET request for `/shrk.bin` to h4.chatterscaled.top (104.21.21.29) over TCP/80. The server responded with `HTTP/1.1 200 OK`, confirming successful delivery of the binary payload to the internal host.

- **Finding 2 – Encrypted Communication Preceded Download:**  
At 12:25:53, 172.16.1.128 initiated a TLS session to 104.21.21.29 with SNI `b1.encountergulf.world`, approximately 21 seconds before the observed binary download. This suggests malicious activity was already in progress prior to the secondary payload retrieval.

- **Finding 3 – Repeated Beaconing Behavior:**  
Thirteen TLS Client Hello sessions were observed to `b1.encountergulf.world`. The repeated encrypted outbound connections are consistent with automated command-and-control communication rather than legitimate user-driven activity.

- **Finding 4 – Single Host Impact:**  
Filtering all traffic involving 104.21.21.29 confirmed that 172.16.1.128 was the only internal system communicating with this infrastructure. No additional internal hosts were observed interacting with the malicious domain or associated IP address within the scope of this capture.

- **Finding 5 – No DNS Visibility:**  
No DNS queries were observed for the malicious domains within the packet capture. Domain resolution likely occurred prior to the start of the capture or through infrastructure not included in the dataset.

- **Finding 6 – Endpoint Detection Triggered:**  
The exported payload triggered Microsoft Defender protections when accessed on the Windows analysis VM. To maintain system integrity, hashing and verification were performed in an isolated Kali Linux virtual machine.

- **Finding 7 – Threat Intelligence Correlation:**  
The SHA256 hash of the retrieved payload was submitted to VirusTotal, where 55 out of 71 security vendors flagged the file as malicious. Several detections referenced trojan/downloader variants (including Zusy/Tepfer family labels). This external validation supports the staged payload delivery and encrypted command-and-control behavior observed in the PCAP.

---

## Indicators of Compromise (IOCs)

**Internal Host:**
- 172.16.1.128

**External IP Address:**
- 104.21.21.29

**Domains:**
- h4.chatterscaled.top
- b1.encountergulf.world

**File Path:**
- /shrk.bin

**Follow-up Payload (shrk.bin):**
- SHA256: 21d0bd2f5870c46cafa2a3ac4771ce0d907e1e03b926ae8820298f639e3b4fb6
- File Size: 8,359,712 bytes
- VirusTotal Detection: 55/71 engines

---

## What I Would Do Next

1. Submit the extracted file hash to VirusTotal and internal threat intelligence platforms to identify malware family and related infrastructure.
2. Deploy detection rules to monitor for outbound TLS connections containing SNI `b1.encountergulf.world` and block associated domains/IP addresses at the perimeter.
3. Perform host-level forensic analysis on 172.16.1.128 to identify persistence mechanisms and additional artifacts.
4. Review proxy and firewall logs for similar traffic patterns across the enterprise.

---

## Key Takeaway
Effective network traffic analysis requires correlating payload delivery, encrypted follow-on communications, and scope validation to confidently assess compromise and provide actionable defensive recommendations.
