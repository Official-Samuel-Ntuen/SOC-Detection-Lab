# Cloud-Native SIEM & EDR Detection Engineering

## Project Overview
Built a fully functional SOC home lab using Wazuh SIEM with 2 live endpoints (Kali Linux + Windows 11), simulated 6 real-world MITRE ATT&CK techniques using Atomic Red Team, wrote 4 custom detection rules, and reduced false positives by 18.7%.

## Architecture

Kali Linux (Attacker + Linux Endpoint)
│
▼
Wazuh Manager (Ubuntu 22.04)
▲
│
Windows 11 (Victim Endpoint + Sysmon)


## Tools Used
- Wazuh 4.9.2 (SIEM + EDR)
- Sysmon (SwiftOnSecurity config)
- Atomic Red Team
- Hydra (brute force simulation)
- VirtualBox

## MITRE ATT&CK Techniques Detected
| Technique | Name | Rule ID |
|-----------|------|---------|
| T1059.001 | PowerShell Execution | 100001 |
| T1110 | Brute Force | 100002 |
| T1003.001 | LSASS Credential Dumping | 100003 |
| T1053.005 | Scheduled Task | 100004 |
| T1574 | DLL Hijacking | Built-in |
| T1078 | Valid Accounts | Built-in |

## False Positive Tuning
- Suppressed 331 low-fidelity alerts
- 18.7% noise reduction
- Maintained detection coverage across all 6 techniques

## Repository Structure
- `/sigma-rules.yml` - Custom Sigma detection rules
- `/detection-playbook.md` - Full detection playbook
- `/wazuh-rules/local_rules.xml` - Wazuh custom rules

## Key Achievements
- Deployed full SIEM stack from scratch
- Enrolled 2 endpoints with live log ingestion
- Wrote and tested 4 custom detection rules
- Tuned false positives with documented before/after metrics
- Built SOC dashboard with MITRE ATT&CK visualization


---

👤 **Author**

**Samuel M. Ntuen**

LinkedIn: *https://www.linkedin.com/in/samuelntuen/*
