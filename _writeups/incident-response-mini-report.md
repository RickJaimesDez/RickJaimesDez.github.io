---
title: "Incident Response Mini Report – NetSupport RAT (PCAP Analysis)"
layout: default
---

# Incident Response Mini Report – NetSupport RAT (PCAP Analysis)

---

## 1. Executive Summary

On 26 November 2024, internal host **10.11.26.183** was observed initiating suspicious outbound communications consistent with malware command-and-control (C2) activity.

Network analysis identified:

- DNS resolution of `modandcrackedapk.com`
- TLS connections containing SNI for the same domain
- Repeated HTTP POST requests over TCP/443 to **194.180.191.64**
- NetSupport-specific application-layer indicators (`User-Agent: NetSupport Manager/1.3`)

TCP stream inspection confirmed active command polling behavior (`CMD=POLL`) and structured server responses (`NetSupport Gateway/1.8`), validating the presence of **NetSupport RAT** communication.

Based on traffic patterns, protocol analysis, and IDS alert correlation, host **10.11.26.183** is assessed as compromised and actively beaconing to external command infrastructure at the time of capture.

No additional internal hosts were observed communicating with the identified infrastructure within the scope of this dataset.

Immediate containment and forensic triage are recommended.

---

## 2. Incident Overview

**Date Observed:** 26 November 2024  
**Data Source:** 2024-11-26-traffic-analysis-exercise.pcap  
**Detection Method:** Network traffic analysis with IDS alert correlation  
**Affected Host:** 10.11.26.183  
**Suspected Malware:** NetSupport RAT  

The investigation began by pivoting on the suspected internal host to scope all related network activity.

---

## 3. Technical Analysis

---

### 3.1 Host Pivot – Traffic Overview

Initial scoping was performed using the filter:

```
ip.addr == 10.11.26.183
```

This provided a consolidated view of all DNS, TLS, and HTTP communications initiated by the host.

![Host Traffic Overview](/assets/images/incident-response-netsupport/ir-host-traffic-overview.png)

This pivot identified suspicious external communications requiring deeper inspection.

---

### 3.2 DNS Resolution

The host queried and resolved the domain:

- `modandcrackedapk.com`

The DNS response returned:

- **193.42.38.139**

This was the first observable indicator of potentially malicious infrastructure.

**Filter Used:**

```
dns contains "modandcrackedapk"
```

![DNS Resolution Screenshot](/assets/images/incident-response-netsupport/ir-dns-resolution.png)

---

### 3.3 TLS Handshake & SNI Validation

TLS Client Hello packets revealed the Server Name Indication (SNI):

- `modandcrackedapk.com`

Although TLS payload contents are encrypted, the SNI field remains visible in plaintext and confirms that the host intentionally initiated encrypted sessions to the suspicious domain.

**Filter Used:**

```
tls.handshake.extensions_server_name contains "modandcrackedapk"
```

![TLS SNI Screenshot](/assets/images/incident-response-netsupport/ir-tls-sni.png)

---

### 3.4 HTTP POST Beaconing to 194.180.191.64

Further inspection revealed repeated HTTP POST requests over TCP/443 to:

- **194.180.191.64**

All requests targeted:

- `/fakeurl.htm`

**Filter Used:**

```
http.request.method == "POST" && ip.addr == 194.180.191.64
```

![C2 POST Traffic Screenshot](/assets/images/incident-response-netsupport/ir-c2-http-post.png)

The repeated outbound POST requests indicate automated polling behavior consistent with command-and-control (C2) beaconing.

Notably, HTTP traffic was observed over port 443 (typically reserved for HTTPS), suggesting an attempt to blend malicious traffic with legitimate encrypted web activity.

---

### 3.5 TCP Stream Validation (Application-Layer Evidence)

Following the TCP stream between:

- 10.11.26.183  
- 194.180.191.64  

revealed explicit NetSupport RAT communication.

Key observations:

- `User-Agent: NetSupport Manager/1.3`
- `Server: NetSupport Gateway/1.8 (Windows NT)`
- Repeated `CMD=POLL` requests
- Encoded response data (`CMD=ENCD`, `DATA=...`)

![TCP Stream Analysis](/assets/images/incident-response-netsupport/ir-c2-tcp-stream-netsupport.png)

The presence of the NetSupport-specific user-agent string and command polling parameters confirms active malware command-and-control behavior.

---

### 3.6 IPv4 Conversation Analysis

Conversation statistics were reviewed using:

**Wireshark → Statistics → Conversations → IPv4**

Host **10.11.26.183** showed sustained traffic volume with **194.180.191.64**, significantly higher than other external communications within the capture.

![IPv4 Conversations Screenshot](/assets/images/incident-response-netsupport/ir-ipv4-conversations.png)

This reinforces the persistent C2 communication pattern observed in earlier analysis.

---

### 3.7 IDS Alert Correlation (Provided Dataset)

An IDS alert file was included with the traffic analysis dataset and used for correlation purposes.

Notable alerts included:

- ETPRO TROJAN NetSupport RAT CnC Activity  
- ETPRO TROJAN Malicious NetSupport RAT CnC Checkin  
- ET POLICY HTTP POST on unusual port (443)

![IDS Alert Screenshot](/assets/images/incident-response-netsupport/ir-ids-netsupport-alerts.png)

Manual packet inspection validated these detections by confirming:

- POST beaconing behavior
- NetSupport user-agent strings
- Command polling activity

This demonstrates alignment between signature-based detection and packet-level validation.

---

## 4. Impact Assessment

- Confirmed malicious outbound C2 communication
- Active command polling behavior observed
- Evidence of remote access capability
- No additional infected hosts observed within capture scope

If uncontained, the infected host could allow an attacker to execute remote commands, exfiltrate data, or establish persistent access.

---

## 5. Containment & Recommendations

1. Immediately isolate host 10.11.26.183 from the network.
2. Block outbound communication to:
   - 193.42.38.139
   - 194.180.191.64
   - modandcrackedapk.com
3. Perform full forensic acquisition and malware eradication.
4. Review DNS, proxy, and firewall logs for additional systems resolving or communicating with the identified infrastructure.
5. Reset credentials associated with the affected host.

---

## 6. Indicators of Compromise (IOCs)

**Internal Host**
- 10.11.26.183

**Domain**
- modandcrackedapk.com

**External IP Addresses**
- 193.42.38.139
- 194.180.191.64

**Behavioral Indicators**
- TLS Client Hello with malicious SNI
- Repeated HTTP POST beaconing over TCP/443
- User-Agent: NetSupport Manager/1.3
- Command polling behavior (`CMD=POLL`)

---

## Key Takeaway

This investigation demonstrates the importance of correlating DNS activity, encrypted session metadata, application-layer inspection, and IDS alerts to confidently identify command-and-control behavior.

Manual packet validation remains critical in confirming malware activity and accurately assessing organizational impact.
