---
layout: page
title: Projects
permalink: /projects/
weight: 10
---

# Projects

These projects demonstrate hands-on SOC-style investigations, intrusion detection development, and structured incident response documentation using real-world datasets.

---

## Featured Projects

<div class="row">

  <div class="col-md-4">
    <div class="card border-0 shadow-sm mb-4">
      <div class="card-body">
        <h5 class="card-title">Real-Time NIDS Dashboard</h5>
        <p class="card-text"><b>Focus:</b> Detection and alerting</p>
        <p class="card-text">Flask dashboard that displays real-time intrusion alerts with timestamps and severity.</p>
        <p class="card-text"><b>What I learned:</b> Turning raw alerts into clear, actionable notes.</p>
        <span class="btn btn-outline-secondary btn-sm disabled">Coming Soon</span>
        <span class="btn btn-outline-secondary btn-sm disabled">Writeup – Coming Soon</span>
      </div>
    </div>
  </div>

  <div class="col-md-4">
    <div class="card border-0 shadow-sm mb-4">
      <div class="card-body">
        <h5 class="card-title">SOC Network Traffic Investigation</h5>
        <p class="card-text"><b>Focus:</b> Malware PCAP Investigation</p>
        <p class="card-text">
          Conducted structured analysis of malware-related packet capture data to identify infected host, validate payload delivery, and extract actionable indicators of compromise (IOCs).
        </p>
        <p class="card-text">
          Included TLS inspection, HTTP artifact validation, and offline payload hashing in an isolated environment.
        </p>
        <a class="btn btn-outline-primary btn-sm" href="https://github.com/RickJaimesDez/soc-network-traffic-investigation">Repo</a>
        <a class="btn btn-outline-primary btn-sm" href="/writeups/network-traffic-investigation/">Writeup</a>
      </div>
    </div>
  </div>

  <div class="col-md-4">
    <div class="card border-0 shadow-sm mb-4">
      <div class="card-body">
        <h5 class="card-title">Incident Response – NetSupport RAT</h5>
        <p class="card-text"><b>Focus:</b> Command-and-Control Validation</p>
        <p class="card-text">
          Investigated NetSupport RAT activity using DNS analysis, TLS SNI validation, HTTP POST beaconing detection, and TCP stream inspection.
        </p>
        <p class="card-text">
          Correlated IDS alerts with packet-level evidence to confirm active C2 communication and assess impact.
        </p>
        <a class="btn btn-outline-primary btn-sm" href="https://github.com/RickJaimesDez/incident-response-mini-report">Repo</a>
        <a class="btn btn-outline-primary btn-sm" href="/writeups/incident-response-mini-report/">Writeup</a>
      </div>
    </div>
  </div>

</div>

---

## Project Structure

Each project includes:

- Problem or investigation scenario
- Tools and environment used
- Evidence (screenshots, logs, packet captures)
- Technical findings
- Indicators of compromise (IOCs)
- Containment or detection recommendations

The goal is to demonstrate structured investigative thinking aligned with SOC workflows.
