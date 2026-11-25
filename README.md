# SOC Analyst L1 Portfolio

**Investigator:** Clinton Chidera  
**Focus:** Security operations, log analysis, incident investigation, and detection engineering  
**Repository:** [GitHub Profile](https://github.com/Govamarket)

## About This Portfolio

This portfolio documents practical Tier 1 SOC exercises performed in an isolated laboratory environment. Each investigation follows a structured process: define scope, collect evidence, analyse activity, assess impact, and provide recommendations.

> **Environment:** Kali Linux 2026.2 and isolated VirtualBox lab  
> **Classification:** Public lab exercises  
> **Data:** No real-world or production information

## Investigations

| Module | Area | Status | Investigation |
|---|---|---|---|
| 01 | Linux investigation | Complete | [SSH Authentication Investigation](Ssh%20authentication%20investigation) |
| 02 | Network analysis | Planned | Protocol and traffic investigation |
| 03 | Windows investigation | Planned | Windows event and timeline investigation |
| 04 | Vulnerability management | Planned | Vulnerability assessment and register |
| 05 | Threat intelligence | Planned | IOC investigation and MITRE ATT&CK mapping |
| 06 | SIEM monitoring | Planned | SIEM setup and alert triage |
| 07 | Incident response | Planned | Incident report and lessons learned |
| 08 | Detection engineering | Planned | Authentication and anomaly rules |
| 09 | SOC capstone | Planned | Shift report, alert register, and incident tickets |

## Featured Investigations

### SSH Authentication Monitoring

This investigation examines authentication visibility on Kali Linux, including journald, rsyslog, failed SSH attempts, audit trails, and the absence of a completed SSH session in `wtmp`.

**Key finding:** The default lab build did not write SSH authentication events to `/var/log/auth.log` until `rsyslog` was installed and enabled.

[Open the SSH authentication report](Ssh%20authentication%20investigation)

### Email Security and Windows Installer Triage

This investigation reviews suspicious-looking email characteristics, SPF, DKIM and DMARC results, message headers, VirusTotal evidence, publisher hashes, signing disclosures and cautious installer-verification steps.

**Key limitation:** The installer was not executed, and local Minisign verification and a recorded Defender scan were not completed.

## Evidence

The [`files`](files) directory contains SSH screenshots and its evidence index. Email-specific screenshots should be grouped separately and renamed descriptively before linking them in the email report.

## Core Analyst Skills

- Linux authentication log investigation
- SSH service configuration and monitoring
- Email authentication and header analysis
- Threat-intelligence evidence review
- Safe executable triage
- Detection validation and false-assurance analysis
- Timeline creation and evidence preservation
- Clear technical documentation

## Framework Alignment

| Framework | Relevant controls |
|---|---|
| ISO/IEC 27001:2022 | A.8.15 Logging; A.8.16 Monitoring activities |
| CIS Controls v8 | Control 8: Audit Log Management |
| NIST CSF 2.0 | DE.CM: Continuous Monitoring |
| MITRE ATT&CK | T1021.004 Remote Services: SSH; T1110 Brute Force |

## Links

- [GitHub profile](https://github.com/Govamarket)
- [SSH authentication report](Ssh%20authentication%20investigation)
- [CTF write-ups](https://github.com/Govamarket/Ctf-writeups)

---

*All investigations were performed in isolated laboratory environments for learning and documentation purposes.*
