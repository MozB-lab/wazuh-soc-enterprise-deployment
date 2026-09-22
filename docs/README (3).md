# Enterprise Wazuh SOC Deployment

A segmented enterprise network security monitoring lab built end-to-end — firewall, DMZ, internal, and management zones, with a self-hosted Wazuh SIEM/XDR deployment validated against five realistic attack and operational scenarios.

Simulated organization: **Meridian Logistics Co.**

📄 [Full Project Documentation](docs/Meridian_SOC_Wazuh_Deployment_Investigation_Portfolio.docx) — objectives, architecture, methodology, consolidated findings, and recommendations
📁 [Per-Scenario Investigation Reports](docs/) — detailed timelines and evidence for each scenario

---

## Overview

This project simulates a mid-size enterprise network behind a firewall, with segmented DMZ, internal, and management zones, monitored by a centralized Wazuh SIEM/XDR stack. Rather than stopping at "infrastructure deployed, agents reporting," the project was built around five scoped scenarios designed to actually validate detection coverage — and to treat every unexpected result along the way (a missing alert, a silent log-format mismatch, a misbehaving firewall rule) as a finding worth root-causing and documenting, not an obstacle to work around quietly.

That approach surfaced multiple genuine gaps in default vendor detection coverage, each remediated with purpose-built custom detection logic in Wazuh's ruleset.

## Objectives

- Build a realistic, segmented enterprise network topology using virtualization
- Deploy and configure Wazuh as a centralized SIEM/XDR platform across endpoints and a network log source (firewall syslog)
- Validate detection coverage against representative attack and operational scenarios
- Practice the full incident-response workflow where warranted — triage, containment, evidence preservation, recovery
- Identify, diagnose, and remediate real gaps in default detection logic, including authoring custom decoders/rules
- Produce professional, reproducible documentation of the process

## Architecture

| Segment | CIDR | Purpose |
|---|---|---|
| WAN | NAT | Simulated internet-facing uplink |
| DMZ (dmznet) | 10.20.0.0/24 | Externally-reachable services (web/FTP server) |
| Internal (intnet) | 10.10.0.0/24 | Simulated corporate LAN |
| Management (mgmtnet) | 10.30.0.0/24 | Isolated SOC/SIEM infrastructure |

| VM | Role |
|---|---|
| pfSense-FW | Perimeter firewall and router between all segments |
| Wazuh-SOC | Wazuh Manager, Indexer, and Dashboard |
| DMZ-Web01 | Apache + DVWA + vsftpd |
| INT-FileSrv | Samba file share with FIM monitoring |
| INT-Win10 | Internal workstation / browser access point |
| Kali-Attacker | Dual-homed adversary-simulation host |

*(Network diagram: `docs/screenshots/network-topology.png` — placeholder pending upload)*

**Stack:** Oracle VirtualBox · pfSense CE · Wazuh (Manager/Indexer/Dashboard/Agent) · Ubuntu Server · Windows 10 · Kali Linux · Apache2 · MariaDB · DVWA · Samba · vsftpd

## Scenarios

| # | Scenario | MITRE ATT&CK | Status |
|---|---|---|---|
| 1 | Internal Brute-Force Attack (RDP) | T1110 | ✅ Complete |
| 2 | File Integrity Monitoring & IR Exercise | T1565.001 | ✅ Complete |
| 3 | File Exfiltration via Anonymous FTP | T1048, T1078 | ✅ Complete |
| 4 | Vulnerable Server Detection | — | ✅ Complete |
| 5 | Internal-to-DMZ Reconnaissance Scan via pfSense Syslog | T1046 | ✅ Complete (one follow-up item open — see docs) |

Each scenario followed the same structure: define an objective, execute the technique, review detection output, root-cause any gap found, remediate where applicable, and verify. Full details, evidence, and screenshots are in each scenario's individual investigation report under `docs/`.

## Key Findings

- **No default Wazuh rule for successful FTP downloads** on the DMZ web server — closed with a custom rule (Scenario 3)
- **No default Wazuh rule for allowed ("pass") pfSense firewall events** — only blocked traffic is covered out of the box — closed with two custom rules (Scenario 5)
- **Silent FIM inventory/alerting gap**: Wazuh's file-integrity inventory updated on file restoration without generating an alert (Scenario 2)
- **A broad, unlogged pfSense firewall rule** was silently absorbing internal-to-DMZ traffic ahead of the intended logging rule, due to top-down rule evaluation order (Scenario 5)
- **An unpurged legacy kernel package** was found doubling the reported vulnerability count on the DMZ server (Scenario 4)
- One item — a pfSense syslog decoding issue discovered during Scenario 5 verification — is documented as an open follow-up rather than presented as resolved

Full detail, remediation, and MITRE mapping for each finding is in the [project documentation](docs/Meridian_SOC_Wazuh_Deployment_Investigation_Portfolio.docx).

## Custom Detection Rules Authored

| Rule ID | Purpose | MITRE |
|---|---|---|
| 100210 | Detects successful FTP download events | T1048 |
| 100211 | Detects individual allowed ("pass") pfSense firewall events | — |
| 100212 | Correlation rule: multiple allowed events from one source in a short window | T1046 |

## Skills Demonstrated

Network segmentation & firewall administration · SIEM/XDR deployment and tuning · Custom decoder/rule authoring · Log source integration · File Integrity Monitoring · Vulnerability management & CVE triage · Incident response (triage, containment, evidence preservation, recovery) · Linux system administration · Network traffic analysis and multi-layer root-cause diagnosis

## Documentation

- [`docs/Meridian_SOC_Wazuh_Deployment_Investigation_Portfolio.docx`](docs/) — full project documentation: objectives, scope, problem statement, architecture, methodology, consolidated findings, lessons learned, recommendations
- `docs/Scenario1_Investigation_Report.docx` — SCN-01-BRUTEFORCE
- `docs/Scenario2_Investigation_Report.docx` — SCN-02-FILETAMPER
- `docs/Scenario3_Investigation_Report.docx` — SCN-03-FTPEXFIL
- `docs/Scenario4_Investigation_Report.docx` — SCN-04-VULNSERVER
- `docs/Scenario5_Investigation_Report.docx` — SCN-05-PORTSCAN

## Author

**Moses (Mozb)** — Cybersecurity professional building a SOC Analyst / Detection Engineering portfolio.
GitHub: [MozB-lab](https://github.com/MozB-lab)

Related projects: [SSH Honeypot & Threat Detection Lab](https://github.com/MozB-lab/honeypot-ssh-cowrie-lab) · [SOAR Automation Platform](https://github.com/MozB-lab/soar-thehive-cortex-automation) · [Threat Intelligence & OSINT Platform](https://github.com/MozB-lab/threat-intelligence-osint-misp)
