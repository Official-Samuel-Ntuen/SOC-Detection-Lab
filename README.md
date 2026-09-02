# Cloud-Native SIEM & EDR Detection Engineering


## 1. Executive Summary
This project demonstrates the design and implementation of a cloud-native Security Operations Center (SOC) using Wazuh SIEM/EDR. Two live endpoints (Kali Linux and Windows 11) were enrolled, and 6 real-world MITRE ATT&CK techniques were simulated using Atomic Red Team. Four custom detection rules were written and tested, achieving an 18.7% false positive reduction through targeted rule tuning. This project proves practical capability in threat detection, log ingestion, and SOC engineering relevant to enterprise security operations.

---

## 2. Architecture

Kali Linux (Attacker + Linux Endpoint)
│
▼
Wazuh Manager (Ubuntu 22.04)
▲
│
Windows 11 (Victim Endpoint + Sysmon)

---

## 3. Technologies Used

**Infrastructure:**
- VirtualBox (local virtualization)
- Host-only + NAT network adapters

**Operating Systems:**
- Ubuntu Server 22.04 LTS (Wazuh Manager)
- Windows 11 Enterprise Evaluation (Victim Endpoint)
- Kali Linux 2026.1 (Attacker + Linux Endpoint)

**Tooling:**
- Wazuh 4.9.2 (SIEM + EDR)
- Sysmon with SwiftOnSecurity config
- Atomic Red Team (attack simulation)
- Hydra (brute force simulation)
- Custom Sigma rules

---

## 4. Execution & Methodology

### Step 1: Deploy Wazuh Manager
- Installed Wazuh all-in-one on Ubuntu 22.04
- Configured indexer, manager, and dashboard
- Accessible via browser at `https://192.168.xx.xxx`

### Step 2: Enroll Endpoints
- Deployed Wazuh agents on Kali Linux and Windows 11
- Installed Sysmon on Windows with SwiftOnSecurity config
- Configured ossec.conf to ingest Sysmon event logs

### Step 3: Simulate MITRE ATT&CK Techniques
| Technique | Name | Tool Used |
|-----------|------|-----------|
| T1059.001 | PowerShell Execution | Atomic Red Team |
| T1110 | Brute Force | Hydra |
| T1003.001 | LSASS Credential Dumping | Atomic Red Team |
| T1053.005 | Scheduled Task Persistence | Atomic Red Team |
| T1574 | DLL Hijacking | Atomic Red Team |
| T1078 | Valid Accounts Abuse | Atomic Red Team |

### Step 4: Write Custom Detection Rules
Four custom Wazuh rules written and tested:
- Rule 100001 — Suspicious PowerShell (Level 12)
- Rule 100002 — Brute Force Detection (Level 12)
- Rule 100003 — LSASS Access (Level 15)
- Rule 100004 — Scheduled Task Creation (Level 10)

### Step 5: False Positive Tuning
- Baseline: 1,769 alerts
- Suppressed: 331 low-fidelity alerts
- Result: **18.7% false positive reduction**
- Maintained detection coverage across all 6 techniques

---

## 5. Defensive Remediation / Lessons Learned

### Hardening Applied
- Locked all management ports (443, 22, 1514) to specific IPs only
- Configured Sysmon to log process creation, network connections, and file changes
- Tuned rules to suppress Windows Update and service logon noise

### Lessons Learned
- **Swap space is critical** for Wazuh indexer on low-RAM systems
- **Sysmon dramatically improves** Windows visibility vs native event logs alone
- **False positive tuning** requires baseline measurement before and after
- **MITRE ATT&CK mapping** helps prioritize which detections matter most

### Detection Coverage Summary
| ATT&CK Tactic | Techniques Covered |
|---------------|-------------------|
| Execution | T1059.001 |
| Credential Access | T1003.001, T1110 |
| Persistence | T1053.005, T1547 |
| Defense Evasion | T1574, T1078 |

---

### Key Achievements
- Deployed full SIEM stack from scratch
- Enrolled 2 endpoints with live log ingestion
- Wrote and tested 4 custom detection rules
- Tuned false positives with documented before/after metrics
- Built SOC dashboard with MITRE ATT&CK visualization


---

👤 **Author**

**Samuel M. Ntuen**

Cybersecurity | Blue Team | SOC Engineering  
[LinkedIn](https://www.linkedin.com/in/samuelntuen/) | [GitHub](https://github.com/Official-Samuel-Ntuen)
