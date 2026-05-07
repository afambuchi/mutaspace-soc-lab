# MutaSpace SOC Lab

## Project Purpose

The MutaSpace SOC Lab is a hands-on virtual cybersecurity lab built on Proxmox. It supports two goals at the same time:

1. **MutaSpace learner development**: Give learners access to a realistic SOC environment where they can practice monitoring, detection, investigation, alert tuning, and incident response.
2. **Research and PhD preparation**: Build a reproducible research platform for studying detection engineering, phishing detection, SIEM telemetry, trust boundaries, and affordable SOC cyber ranges.

This repository documents the lab from the ground up so another person can recreate the environment, understand the design decisions, and learn from each step.

A certification can show what someone knows. This lab is designed to show what someone can do.

---

## Current Status

The Proxmox host currently has the network foundation created, but no virtual machines have been deployed yet.

Current bridge layout:

| Bridge | Current Role | Notes |
|---|---|---|
| `vmbr2` | External / management / WAN bridge | Physical Ethernet interface `enp1s0` is attached here |
| `vmbr0` | Internal SOC / Blue network bridge | No physical NIC attached |
| `vmbr1` | Internal Red / untrusted network bridge | No physical NIC attached |

Current physical and host interfaces:

| Interface | Purpose |
|---|---|
| `enp1s0` | Wired physical network adapter, attached to `vmbr2` |
| `wlp2s0` | Wireless adapter, currently not used for the lab |
| `tailscale0` | Remote administration interface, not part of the lab network design yet |

The next build milestone is to verify the Proxmox host IP settings and then deploy the first VM: the firewall/router VM.

---

## Lab Design Philosophy

This lab follows a simple rule:

> Build the environment in a way that teaches the concept, documents the reason, and produces evidence that the learner can perform the task.

Every major part of the lab should answer four questions:

1. **What are we building?**
2. **Why does this matter in a real SOC?**
3. **How do we validate that it works?**
4. **What artifact proves the learner or researcher did the work?**

This project is not only a home lab. It is intended to become:

- A MutaSpace training environment
- A detection engineering portfolio
- A reproducible SOC lab guide
- A research platform
- A future foundation for a physical SOC lab in Houston

---

## High-Level Architecture

The lab uses Proxmox bridges as virtual switches. VMs are attached to those bridges based on their purpose.

```text
                         Existing Physical Network
                                  |
                                vmbr2
                                  |
                          [ fw-01 Firewall VM ]
                         /                       \
                    vmbr0                         vmbr1
             Blue / SOC Network             Red / Untrusted Network
                10.20.0.0/24                    10.30.0.0/24
                    |                               |
        Wazuh, Windows, Linux,              Kali, attacker systems,
        Suricata, learner systems           untrusted research VM
```

Planned bridge roles:

| Bridge | Role | Planned Network | Purpose |
|---|---|---|---|
| `vmbr2` | External / WAN / management bridge | Existing LAN | Proxmox access, firewall WAN, controlled internet access |
| `vmbr0` | Blue / SOC / enterprise bridge | `10.20.0.0/24` | Wazuh, Windows endpoint, Linux endpoint, SOC tools |
| `vmbr1` | Red / untrusted bridge | `10.30.0.0/24` | Kali, adversary simulation, untrusted VM |

---

## Key Concept: What Is a Proxmox Bridge?

A Proxmox bridge is a virtual network switch.

A physical network card connects the Proxmox host to the outside network. A bridge connects virtual machines to each other or to that physical network.

In this lab:

```text
enp1s0 is the physical Ethernet port.
vmbr2 is the bridge connected to that physical Ethernet port.
vmbr0 and vmbr1 are isolated virtual switches for lab traffic.
```

Simple mental model:

```text
Physical NIC = door to the outside network
Proxmox bridge = virtual switch
VM network adapter = cable plugged into a virtual switch
Firewall VM = controlled gate between networks
```

This is the foundation of the entire lab.

---

## Planned VM Inventory

| VM Name | Role | Network Connections | Purpose |
|---|---|---|---|
| `fw-01` | Firewall/router | `vmbr2`, `vmbr0`, `vmbr1` | Routes traffic between lab networks and controls access |
| `wazuh-01` | SIEM manager and dashboard | `vmbr0` | Collects and analyzes endpoint, network, and security logs |
| `sensor-01` | Suricata sensor, later Zeek | `vmbr0`, `vmbr1` | Monitors network traffic and generates IDS alerts |
| `win-client-01` | Windows workstation | `vmbr0` | Learner endpoint, Sysmon source, phishing target |
| `linux-client-01` | Linux endpoint | `vmbr0` | Linux logging, auth logs, web service, learner endpoint |
| `dc-01` | Windows Server / Active Directory | `vmbr0` | Domain authentication and Windows enterprise telemetry |
| `kali-01` | Red team simulation VM | `vmbr1` | Controlled adversary simulation inside the lab only |
| `untrusted-01` | Differently configured VM | `vmbr1` | Trust-boundary research with partial or missing telemetry |
| `nlp-01` | Python analysis VM | `vmbr0` | Phishing dataset analysis and SIEM telemetry experiments |

Initial build order:

1. `fw-01`
2. `wazuh-01`
3. `win-client-01`
4. `linux-client-01`
5. `sensor-01`
6. `kali-01`
7. `untrusted-01`
8. `nlp-01`
9. `dc-01`

---

## Planned IP Addressing

| Network | Gateway | DHCP Range | Static Range | Purpose |
|---|---|---|---|---|
| Existing LAN | Existing router | Existing DHCP | Existing LAN | Proxmox management and firewall WAN |
| `10.20.0.0/24` | `10.20.0.1` | `10.20.0.100-10.20.0.199` | `10.20.0.2-10.20.0.99` | Blue SOC network |
| `10.30.0.0/24` | `10.30.0.1` | `10.30.0.100-10.30.0.199` | `10.30.0.2-10.30.0.99` | Red / untrusted network |

Suggested static IPs:

| System | IP Address |
|---|---|
| `fw-01` Blue interface | `10.20.0.1` |
| `fw-01` Red interface | `10.30.0.1` |
| `wazuh-01` | `10.20.0.10` |
| `sensor-01` | `10.20.0.20` |
| `win-client-01` | DHCP or `10.20.0.50` |
| `linux-client-01` | DHCP or `10.20.0.60` |
| `dc-01` | `10.20.0.5` |
| `kali-01` | DHCP or `10.30.0.50` |
| `untrusted-01` | DHCP or `10.30.0.60` |
| `nlp-01` | `10.20.0.70` |

---

## Build Roadmap

### Phase 0: Baseline the Proxmox Host

Goal: Document the current network state before creating VMs.

Tasks:

- Record bridge layout
- Record IP addresses
- Record routes
- Back up `/etc/network/interfaces`
- Confirm how Proxmox is being accessed

Artifacts:

```text
proxmox/bridge-plan.md
proxmox/current-network-state.md
proxmox/interfaces-backup-notes.md
```

Validation commands:

```bash
ip -br addr
ip route
bridge link
cat /etc/network/interfaces
```

Learning checkpoint:

> I can explain which Proxmox bridge connects to the real network and which bridges are isolated for the lab.

---

### Phase 1: Deploy the Firewall VM

Goal: Create controlled routing between the external network, Blue SOC network, and Red untrusted network.

Firewall VM network adapters:

| Adapter | Proxmox Bridge | Firewall Role |
|---|---|---|
| NIC 1 | `vmbr2` | WAN |
| NIC 2 | `vmbr0` | Blue LAN |
| NIC 3 | `vmbr1` | Red LAN |

Tasks:

- Install pfSense or OPNsense
- Assign WAN, Blue LAN, and Red LAN interfaces
- Configure Blue LAN as `10.20.0.1/24`
- Configure Red LAN as `10.30.0.1/24`
- Enable DHCP on Blue and Red networks
- Create firewall rules that allow safe lab operation

Artifacts:

```text
firewall/fw-01-install.md
firewall/interface-assignments.md
firewall/ruleset-v1.md
troubleshooting/firewall-interface-mapping.md
```

Learning checkpoint:

> I can explain why the firewall VM is the gate between isolated lab networks and the outside network.

---

### Phase 2: Deploy Wazuh

Goal: Build the central SIEM for the lab.

Tasks:

- Create `wazuh-01`
- Assign it to `vmbr0`
- Set static IP `10.20.0.10`
- Install Wazuh manager, indexer, and dashboard
- Confirm dashboard access from the Blue network
- Document login method and access path

Artifacts:

```text
wazuh/install.md
wazuh/network-settings.md
wazuh/dashboard-access.md
troubleshooting/wazuh-dashboard-not-loading.md
```

Learning checkpoint:

> I can explain the role of a SIEM and why Wazuh is the central evidence collection point in this lab.

---

### Phase 3: Add Monitored Endpoints

Goal: Generate real endpoint telemetry.

Tasks:

- Create `win-client-01`
- Install Wazuh agent on Windows
- Install Sysmon on Windows
- Create `linux-client-01`
- Install Wazuh agent on Linux
- Confirm both systems report to Wazuh

Artifacts:

```text
endpoints/windows-agent-install.md
endpoints/sysmon-install.md
endpoints/linux-agent-install.md
wazuh/agent-inventory.md
troubleshooting/wazuh-agent-not-reporting.md
```

Learning checkpoint:

> I can explain the difference between endpoint telemetry and network telemetry.

---

### Phase 4: Add Network Detection

Goal: Monitor traffic with Suricata and later Zeek.

Tasks:

- Create `sensor-01`
- Install Suricata
- Configure EVE JSON output
- Forward Suricata alerts into Wazuh
- Generate a safe test alert inside the lab
- Document what Suricata sees that endpoint logs may miss

Artifacts:

```text
suricata/install.md
suricata/eve-json.md
suricata/wazuh-ingestion.md
suricata/test-alerts.md
troubleshooting/suricata-alerts-not-ingesting.md
```

Learning checkpoint:

> I can explain why a SOC needs both host-based and network-based evidence.

---

### Phase 5: Detection Engineering

Goal: Create and tune custom detections.

Initial custom rule families:

| Rule Family | Data Source | Purpose |
|---|---|---|
| Failed authentication | Windows Security logs, Linux auth logs | Detect brute force and password spraying patterns |
| Phishing indicators | Email headers, mail logs, DNS, HTTP logs | Connect phishing analysis to SIEM telemetry |
| Lateral movement | Windows logs, Sysmon, Suricata | Detect movement between systems |
| Suspicious PowerShell | Windows PowerShell and Sysmon logs | Detect suspicious command execution |
| Persistence | Windows service creation, Linux startup changes | Detect persistence attempts |

Artifacts:

```text
wazuh/rules/
wazuh/decoders/
wazuh/test-logs/
wazuh/tuning-journal.md
research/detection-engineering-learning/
```

Learning checkpoint:

> I can explain the difference between a noisy detection and a useful detection.

---

### Phase 6: Trust Boundary Research

Goal: Study what happens when one system does not provide the same quality of telemetry as the rest of the lab.

Tasks:

- Create `untrusted-01` on `vmbr1`
- Start with no Wazuh agent
- Monitor it through network telemetry only
- Later add limited syslog forwarding
- Compare detection confidence against a fully monitored endpoint

Research question:

> How should a SOC assign confidence to alerts when different systems provide different levels of telemetry?

Artifacts:

```text
research/telemetry-confidence/problem-statement.md
research/telemetry-confidence/confidence-model.md
research/telemetry-confidence/evidence-types.md
trust-boundary/untrusted-vm-config.md
trust-boundary/observations.md
```

Learning checkpoint:

> I can explain why an alert from a fully monitored endpoint is not the same as an alert inferred from network traffic only.

---

### Phase 7: Phishing, NLP, and SIEM Telemetry

Goal: Build a phishing research pipeline that compares text-only detection with text plus operational telemetry.

Tasks:

- Set up phishing email dataset workspace
- Parse email headers and body text
- Build a simple TF-IDF baseline
- Add header features
- Add SIEM and network telemetry features
- Compare results

Research question:

> Can SIEM and network telemetry improve phishing detection beyond text-only models?

Example feature groups:

| Feature Group | Examples |
|---|---|
| Email text | Body text, subject, word frequency, TF-IDF |
| Email headers | From, Reply-To, Received path, SPF/DKIM/DMARC results if available |
| URL features | Number of links, domain mismatch, suspicious URL patterns |
| SIEM telemetry | Mail logs, endpoint alert, user action, timestamp correlation |
| Network telemetry | DNS lookup, HTTP request, Suricata alert, Zeek metadata later |

Artifacts:

```text
phishing-nlp/README.md
phishing-nlp/notebooks/01-email-parsing.ipynb
phishing-nlp/notebooks/02-tfidf-baseline.ipynb
phishing-nlp/notebooks/03-text-plus-telemetry.ipynb
phishing-nlp/features.md
phishing-nlp/results-template.md
```

Learning checkpoint:

> I can explain how phishing detection changes when the model has operational context in addition to email text.

---

### Phase 8: Learner SOC Scenarios

Goal: Convert the lab into repeatable MutaSpace training modules.

Initial learner scenarios:

| Scenario | Learner Outcome | Evidence Produced |
|---|---|---|
| Failed login investigation | Identify brute force evidence | Timeline, Wazuh alert, affected user |
| Phishing triage | Analyze suspicious email and network activity | Email header notes, SIEM evidence, recommendation |
| Lateral movement investigation | Reconstruct movement between systems | Timeline, host evidence, network evidence |
| Alert tuning | Reduce false positives without hiding true positives | Rule change, tuning explanation |
| Incident report writing | Communicate findings clearly | Executive summary and technical report |

Artifacts:

```text
scenarios/failed-authentication/student-guide.md
scenarios/phishing-triage/student-guide.md
scenarios/lateral-movement/student-guide.md
scenarios/alert-tuning/student-guide.md
scenarios/incident-reporting/student-guide.md
```

Learning checkpoint:

> I can use logs and alerts to explain what happened, not just say that an alert fired.

---

## Research Agenda

This lab supports two connected research tracks.

### Track A: Independent MutaSpace Research

Main theme:

> Affordable SOC cyber ranges for detection engineering and workforce development.

Primary research question:

> Can a low-cost Proxmox-based SOC lab produce measurable cybersecurity skill development while also generating reproducible detection engineering artifacts?

Supporting questions:

1. Can learners demonstrate SOC readiness through evidence-based investigations?
2. Does writing and tuning detections improve learner understanding?
3. What kinds of alerts create the most confusion for beginner analysts?
4. Can learner progress be measured without over-collecting personal data?
5. Can a small virtual SOC lab generate useful labeled datasets for training and research?

Planned artifacts:

```text
research/research-agenda.md
research/low-cost-soc-range/research-question.md
research/low-cost-soc-range/learner-rubric.md
research/low-cost-soc-range/data-collection-plan.md
research/detection-engineering-learning/hypothesis.md
research/detection-engineering-learning/grading-rubric.md
```

### Track B: PhD-Focused Research Preparation

Main theme:

> SIEM telemetry, phishing detection, NLP, and trust boundaries in virtualized SOC environments.

Primary research questions:

1. Can SIEM telemetry enrich text-based phishing detection models?
2. How does detection confidence change when telemetry quality differs across systems?
3. What signals distinguish malicious behavior from normal lab noise?
4. How can virtual SOC labs support reproducible cybersecurity research?

Planned artifacts:

```text
research/phd-track/phishing-nlp-question.md
research/phd-track/trust-boundary-question.md
research/phd-track/detection-engineering-notes.md
research/phd-track/reading-notes/
```

---

## Detection Engineering Goals

The lab will not rely only on default alerts. Custom detections are required.

Each detection should include:

- Detection name
- Data source
- Rule logic
- Expected true positive behavior
- Expected false positive behavior
- Test log or test scenario
- Tuning notes
- MITRE ATT&CK mapping when appropriate
- Validation screenshot or exported alert

Detection documentation template:

```markdown
# Detection Name

## Purpose

## Data Source

## Rule Logic

## Test Scenario

## Expected Alert

## False Positives Observed

## Tuning Decision

## Validation Evidence

## Remaining Research Questions
```

---

## Learner Evidence Model

Every learner lab should produce evidence.

Examples:

| Evidence Type | Purpose |
|---|---|
| Screenshot | Shows the learner reached the correct tool or alert |
| Log excerpt | Shows the learner can cite evidence |
| Timeline | Shows the learner understands sequence |
| Detection rule | Shows the learner can create or modify logic |
| Tuning note | Shows the learner can reduce noise thoughtfully |
| Incident report | Shows the learner can communicate findings |
| Reflection | Shows the learner can explain what they learned |

Learner work should be graded on evidence, not just final answers.

---

## Research Measurement Model

For MutaSpace research, learner performance can be measured with simple, repeatable metrics.

| Metric | What It Measures |
|---|---|
| Time to first correct finding | How quickly the learner finds useful evidence |
| Evidence quality score | Whether the learner cites logs, timestamps, users, IPs, and affected hosts |
| False conclusion rate | Whether the learner makes unsupported claims |
| Detection logic score | Whether the learner understands the rule or alert logic |
| Incident report score | Whether the learner communicates clearly |
| Confidence change | Whether the learner's self-assessment changes after the lab |

Privacy principle:

> Collect only the learner data needed to measure skill growth. Avoid unnecessary personal data collection.

---

## Repository Structure

Planned structure:

```text
mutaspace-soc-lab/
├── README.md
├── CHANGELOG.md
├── diagrams/
│   ├── network-architecture.drawio
│   ├── network-architecture.png
│   └── traffic-flows.md
├── proxmox/
│   ├── bridge-plan.md
│   ├── current-network-state.md
│   ├── vm-inventory.md
│   ├── snapshot-reset-process.md
│   └── firewall-policy.md
├── firewall/
│   ├── fw-01-install.md
│   ├── interface-assignments.md
│   └── ruleset-v1.md
├── wazuh/
│   ├── install.md
│   ├── agent-inventory.md
│   ├── rules/
│   ├── decoders/
│   ├── test-logs/
│   └── tuning-journal.md
├── suricata/
│   ├── install.md
│   ├── eve-json.md
│   ├── wazuh-ingestion.md
│   └── test-alerts.md
├── endpoints/
│   ├── windows-agent-install.md
│   ├── sysmon-install.md
│   └── linux-agent-install.md
├── phishing-nlp/
│   ├── README.md
│   ├── notebooks/
│   ├── features.md
│   └── results-template.md
├── trust-boundary/
│   ├── untrusted-vm-config.md
│   ├── observations.md
│   └── telemetry-comparison.md
├── scenarios/
│   ├── failed-authentication/
│   ├── phishing-triage/
│   ├── lateral-movement/
│   ├── alert-tuning/
│   └── incident-reporting/
├── research/
│   ├── research-agenda.md
│   ├── low-cost-soc-range/
│   ├── detection-engineering-learning/
│   ├── telemetry-confidence/
│   └── phd-track/
└── troubleshooting/
    ├── proxmox-bridge-connectivity.md
    ├── firewall-interface-mapping.md
    ├── wazuh-agent-not-reporting.md
    ├── wazuh-dashboard-not-loading.md
    └── suricata-alerts-not-ingesting.md
```

---

## Documentation Standard

Every build guide should use this structure:

```markdown
# Guide Title

## Purpose

## What This Builds

## Why It Matters

## Prerequisites

## Steps

## Validation

## Troubleshooting

## Screenshots or Evidence

## What I Learned

## Next Step
```

Every troubleshooting guide should use this structure:

```markdown
# Troubleshooting: Problem Name

## Date

## System

## Expected Behavior

## Observed Behavior

## Hypotheses

## Evidence Collected

## Root Cause

## Fix

## Validation

## Lesson Learned
```

---

## Safety and Ethics

This lab is for education, research, and controlled simulation only.

Rules:

1. Attack simulation must stay inside the lab networks.
2. Red network systems should not be used to attack public systems.
3. Malware samples are not required for the first version of this lab.
4. Learner telemetry should be handled carefully and minimized.
5. Credentials, API keys, private IP details, and sensitive screenshots should not be committed to the public repository.
6. All scenarios should be designed so they can be reset safely.

The purpose of this lab is to build skill, judgment, and responsible security practice.

---

## Definition of Done for Version 1

Version 1 of the lab is complete when the following are true:

| Requirement | Status |
|---|---|
| Proxmox bridge layout documented | In progress |
| Firewall VM deployed | Not started |
| Blue SOC network working | Not started |
| Red untrusted network working | Not started |
| Wazuh installed and reachable | Not started |
| Windows endpoint reporting to Wazuh | Not started |
| Linux endpoint reporting to Wazuh | Not started |
| Suricata alerts flowing into Wazuh | Not started |
| At least three custom Wazuh rules created | Not started |
| One false-positive tuning cycle documented | Not started |
| One trust-boundary experiment documented | Not started |
| One phishing NLP baseline completed | Not started |
| One text-plus-telemetry phishing experiment completed | Not started |
| Three learner scenarios written | Not started |
| One learner grading rubric written | Not started |
| GitHub documentation clean enough for reproduction | Not started |

---

## First Three Scenarios

### Scenario 1: Failed Authentication Investigation

Learner task:

> Investigate repeated failed logins and determine whether the activity is normal user error or suspicious authentication behavior.

Evidence required:

- Alert ID or event name
- Source IP
- Target host
- Target account
- Timeline
- Final conclusion

Research use:

- Detection engineering
- Alert fatigue
- Learner evidence quality

### Scenario 2: Phishing Triage

Learner task:

> Analyze a suspicious email using headers, body text, URL indicators, endpoint activity, and network telemetry.

Evidence required:

- Suspicious header fields
- URL or domain notes
- Related DNS or HTTP activity
- Wazuh or Suricata evidence
- Recommendation

Research use:

- NLP and phishing detection
- SIEM telemetry enrichment
- Incident report quality

### Scenario 3: Lateral Movement Timeline

Learner task:

> Reconstruct a possible lateral movement attempt using Windows logs, Linux logs, Sysmon events, and network alerts.

Evidence required:

- Initial system
- Target system
- User account involved
- Network evidence
- Endpoint evidence
- Timeline

Research use:

- Timeline reconstruction
- Telemetry confidence scoring
- Multi-source evidence analysis

---

## Research Identity Statement

This project supports the following research identity:

> I study how affordable virtual SOC labs can develop real cybersecurity capability by combining detection engineering, telemetry confidence, reproducible lab design, and measurable learner outcomes.

This identity connects MutaSpace, SOC workforce development, and PhD-level research.

---

## Interview Summary

A short explanation of this project:

> I am building a Proxmox-based SOC lab with segmented virtual networks, a firewall VM, Wazuh SIEM, Windows and Linux endpoints, Suricata network monitoring, attacker simulation, and an intentionally under-instrumented VM for trust-boundary research. The lab supports MutaSpace learners while also serving as a reproducible research platform for detection engineering, phishing detection with SIEM telemetry, telemetry confidence scoring, and affordable SOC cyber range effectiveness.

---

## Immediate Next Step

Before creating VMs, capture the current Proxmox network state:

```bash
mkdir -p ~/mutaspace-soc-lab-notes/network
ip -br addr | tee ~/mutaspace-soc-lab-notes/network/ip-brief-addr.txt
ip route | tee ~/mutaspace-soc-lab-notes/network/ip-route.txt
bridge link | tee ~/mutaspace-soc-lab-notes/network/bridge-link.txt
cat /etc/network/interfaces | tee ~/mutaspace-soc-lab-notes/network/interfaces.txt
```

Then create the first VM:

```text
VM name: fw-01
Role: Firewall/router
NIC 1: vmbr2, WAN
NIC 2: vmbr0, Blue LAN
NIC 3: vmbr1, Red LAN
```

---

## Maintainer Notes

This repository is built as a living document. Every lab change should create documentation, and every documented step should be clear enough for another learner to reproduce.

When in doubt, document:

1. What changed
2. Why it changed
3. How it was tested
4. What broke
5. What was learned
