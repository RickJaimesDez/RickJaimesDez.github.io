---
layout: page
title: Writeups
permalink: /writeups/
weight: 15
---

# Writeups

These writeups document structured SOC-style investigations using real-world datasets.  
Each entry emphasizes packet-level validation, indicator extraction, and incident-focused reporting rather than step-by-step lab walkthroughs.

---

## Featured Writeups

<div class="row">

  <div class="col-md-6">
    <div class="card border-0 shadow-sm mb-4">
      <div class="card-body">
        <h5 class="card-title">SOC Network Traffic Investigation</h5>
        <p class="card-text">
          Structured malware PCAP investigation identifying infected host, confirming payload delivery, and validating encrypted follow-on communication.
        </p>
        <p class="card-text">
          <b>Focus:</b> Wireshark analysis, TLS inspection, IOC extraction, single-host impact validation
        </p>
        <a class="btn btn-outline-primary btn-sm" href="/writeups/network-traffic-investigation/">
          View Writeup
        </a>
      </div>
    </div>
  </div>

  <div class="col-md-6">
    <div class="card border-0 shadow-sm mb-4">
      <div class="card-body">
        <h5 class="card-title">Incident Response – NetSupport RAT</h5>
        <p class="card-text">
          PCAP-based incident investigation confirming command-and-control (C2) activity through DNS resolution, TLS SNI validation, HTTP POST beaconing, and TCP stream inspection.
        </p>
        <p class="card-text">
          <b>Focus:</b> IDS correlation, C2 validation, application-layer analysis, structured incident reporting
        </p>
        <a class="btn btn-outline-primary btn-sm" href="/writeups/incident-response-mini-report/">
          View Writeup
        </a>
      </div>
    </div>
  </div>

</div>

---

## Investigation Structure

Each writeup follows a consistent investigation workflow:

- Defined investigation scope
- Environment and dataset identification
- Host pivot and traffic scoping
- Protocol-level and application-layer validation
- Indicator extraction
- Impact assessment and containment recommendations

The objective is to demonstrate practical SOC investigative thinking aligned with real-world alert escalation and incident response processes.
