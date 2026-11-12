# 🛡️ Cloud-Native SIEM & EDR Detection Engineering Lab

> A fully functional Security Operations Center (SOC) home lab built from scratch using Wazuh SIEM/EDR, custom detection rules mapped to MITRE ATT&CK, and automated attack simulations using Atomic Red Team.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Wazuh](https://img.shields.io/badge/Wazuh-4.9.2-blue.svg)](https://wazuh.com)
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red.svg)](https://attack.mitre.org)

---

## 📌 Executive Summary

This project demonstrates the end-to-end implementation of a cloud-native SOC environment. It goes beyond simply installing a SIEM — it covers threat simulation, custom detection engineering, false positive tuning, and dashboard visualization. The lab was designed to mirror real-world enterprise security operations, making it directly applicable to SOC analyst, detection engineer, and blue team roles.

**Key achievements:**
- Deployed Wazuh 4.9.2 all-in-one SIEM stack from scratch
- Enrolled 2 live endpoints (Kali Linux + Windows 11) with active log ingestion
- Simulated 6 real-world MITRE ATT&CK techniques using Atomic Red Team
- Wrote 4 custom detection rules (Sigma + Wazuh XML format)
- Reduced false positives by **18.7%** through systematic tuning
- Built a SOC dashboard visualizing attack patterns in real time

---

## 🎯 Purpose & What Makes This Different

Most SIEM home lab projects stop at installation. This project goes further by:

| Feature | This Project | Typical Home Lab |
|---------|-------------|-----------------|
| Custom detection rules | ✅ 4 custom rules written from scratch | ❌ Only default rules |
| Attack simulation | ✅ Atomic Red Team + Hydra brute force | ❌ No simulations |
| False positive tuning | ✅ Documented before/after metrics | ❌ Not addressed |
| MITRE ATT&CK mapping | ✅ 6 techniques mapped and detected | ❌ Generic alerts |
| Detection playbook | ✅ Full playbook per technique | ❌ No documentation |
| Sysmon integration | ✅ SwiftOnSecurity config | ❌ Basic Windows logs only |

---

## 🏗️ Architecture
┌─────────────────────────────────────────────────────┐
│ Physical Machine (Kali Linux) │
│ │
│ ┌─────────────────┐ ┌──────────────────────┐ │
│ │ Kali Linux │ │ Windows 11 VM │ │
│ │ (Attacker + │─────▶│ (Victim Endpoint) │ │
│ │ Endpoint 1) │ │ Wazuh Agent + Sysmon│ │
│ │ Wazuh Agent │ └──────────┬───────────┘ │
│ └────────┬────────┘ │ │
│ │ TCP 1514 │ │
│ └──────────────┬───────────┘ │
│ ▼ │
│ ┌──────────────────────────┐ │
│ │ Wazuh Manager │ │
│ │ Ubuntu 22.04 │ │
│ │ 192.168.56.101 │ │
│ │ Indexer + Dashboard │ │
│ │ Wazuh 4.9.2 │ │
│ └──────────────────────────┘ │
└─────────────────────────────────────────────────────┘


**Network:** VirtualBox Host-Only Adapter (192.168.56.0/24) for internal communication + NAT for internet access.

---

## 🛠️ Technologies Used

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| SIEM/EDR | Wazuh | 4.9.2 | Log ingestion, alerting, dashboard |
| OS - Manager | Ubuntu Server | 22.04 LTS | Wazuh manager host |
| OS - Endpoint 1 | Kali Linux | 2026.1 | Attacker + Linux endpoint |
| OS - Endpoint 2 | Windows 11 | Enterprise Eval | Windows victim endpoint |
| EDR Telemetry | Sysmon | Latest | Deep Windows process visibility |
| Log Config | SwiftOnSecurity | Latest | Sysmon configuration baseline |
| Attack Simulation | Atomic Red Team | Latest | MITRE ATT&CK test framework |
| Brute Force | Hydra | 9.7 | Authentication attack simulation |
| Virtualization | VirtualBox | 7.2 | Local lab environment |

---

## 📋 Prerequisites

Before cloning or reproducing this lab, ensure you have:

- A machine with minimum **8GB RAM** and **100GB free disk space**
- VirtualBox 7.x installed
- Ubuntu Server 22.04 LTS ISO
- Windows 11 Enterprise Evaluation ISO (free from Microsoft)
- Basic familiarity with Linux command line
- Internet connection for package downloads

---

## 🚀 Setup Guide (Step by Step)

### Step 1: Create the Wazuh Manager VM

1. Open VirtualBox → **New**
2. Configure:
   - Name: `wazuh-manager`
   - OS: Ubuntu 22.04 LTS (64-bit)
   - RAM: **3072 MB minimum** (Wazuh indexer is RAM-heavy)
   - Disk: **40 GB** (Wazuh + logs need space)
3. Network settings:
   - Adapter 1: NAT (internet access)
   - Adapter 2: Host-only Adapter → `vboxnet0`
4. Install Ubuntu Server 22.04 with OpenSSH enabled

> ⚠️ **Important:** Before installing Wazuh, add swap space to prevent indexer failures:
> ```bash
> sudo fallocate -l 3G /swapfile
> sudo chmod 600 /swapfile
> sudo mkswap /swapfile
> sudo swapon /swapfile
> echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
> ```

### Step 2: Install Wazuh (All-in-One)

```bash
# Download installer
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh

# Run installer (use -i to bypass hardware check on VMs)
sudo bash wazuh-install.sh -a -i
```

> ⏱️ Installation takes 10-15 minutes. At completion, save the generated admin password — it is shown only once.

Access the dashboard at: `https://192.168.56.101`

### Step 3: Enroll Linux Endpoint (Kali)

On the Wazuh dashboard → **Agents** → **Deploy new agent**:
- OS: Linux → DEB amd64
- Server address: `192.168.56.101`
- Agent name: `kali-endpoint`

Copy and run the generated command on Kali, then:
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### Step 4: Enroll Windows Endpoint + Install Sysmon

**Install Wazuh agent** (from dashboard wizard, select Windows → MSI):
```powershell
# Run generated command from dashboard in PowerShell as Administrator
```

**Install Sysmon:**
```powershell
curl.exe -L "https://download.sysinternals.com/files/Sysmon.zip" -o "C:\Sysmon.zip"
Expand-Archive -Path "C:\Sysmon.zip" -DestinationPath "C:\Sysmon" -Force
curl.exe -L "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -o "C:\Sysmon\sysmonconfig.xml"
C:\Sysmon\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig.xml
```

**Configure Wazuh agent to ingest Sysmon logs:**

Add to `C:\Program Files (x86)\ossec-agent\ossec.conf` before `</ossec_config>`:
```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

```powershell
Restart-Service WazuhSvc
```

### Step 5: Deploy Custom Detection Rules

SSH into the Wazuh manager and run:

```bash
sudo tee /var/ossec/etc/rules/local_rules.xml << 'EOF'
<group name="custom_detections,">
  <rule id="100001" level="12">
    <if_group>windows</if_group>
    <field name="win.eventdata.image" type="pcre2">(?i)powershell\.exe</field>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)(encodedcommand|bypass|hidden|downloadstring|iex)</field>
    <description>Suspicious PowerShell execution - T1059.001</description>
    <mitre><id>T1059.001</id></mitre>
    <group>attack,</group>
  </rule>
  <rule id="100002" level="12" frequency="5" timeframe="60">
    <if_matched_sid>60122</if_matched_sid>
    <description>Brute force detected - T1110</description>
    <mitre><id>T1110</id></mitre>
    <group>attack,authentication_failed,</group>
  </rule>
  <rule id="100003" level="15">
    <if_group>sysmon</if_group>
    <field name="win.eventdata.targetImage" type="pcre2">(?i)lsass\.exe</field>
    <field name="win.eventdata.grantedAccess" type="pcre2">0x1010|0x1410</field>
    <description>LSASS credential dumping - T1003.001</description>
    <mitre><id>T1003.001</id></mitre>
    <group>attack,credential_access,</group>
  </rule>
  <rule id="100004" level="10">
    <if_group>windows</if_group>
    <field name="win.eventdata.image" type="pcre2">(?i)schtasks\.exe</field>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)/create</field>
    <description>Scheduled task persistence - T1053.005</description>
    <mitre><id>T1053.005</id></mitre>
    <group>attack,persistence,</group>
  </rule>
</group>
EOF

sudo systemctl restart wazuh-manager
```

---

## 🔥 Proof of Concept — Attack Simulations

### PoC 1: PowerShell Execution (T1059.001)

**Tool:** Atomic Red Team on Windows endpoint  
**Command:**
```powershell
Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force
Invoke-AtomicTest T1059.001 -TestNumbers 1
```
**Result:** Custom rule 100001 fired at level 12 within 30 seconds  
**Evidence:** See `/screenshots/T1059_alert.png`

---

### PoC 2: Brute Force Attack (T1110)

**Tool:** Hydra from Kali Linux  
**Command:**
```bash
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.56.102 -t 4
```
**Result:** 331 failed logon alerts generated, user account locked out, rule 100002 fired at level 12  
**Evidence:** See `/screenshots/T1110_bruteforce.png`

---

### PoC 3: Scheduled Task Persistence (T1053.005)

**Tool:** Atomic Red Team  
**Command:**
```powershell
Invoke-AtomicTest T1053.005 -TestNumbers 1
```
**Result:** Rule 100004 fired at level 10, schtasks.exe process creation detected via Sysmon  
**Evidence:** See `/screenshots/T1053_persistence.png`

---

### PoC 4: LSASS Credential Dumping (T1003.001)

**Tool:** Atomic Red Team  
**Command:**
```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1
```
**Result:** Rule 100003 fired at level 15 (critical) — highest severity in the lab  
**Evidence:** See `/screenshots/T1003_lsass.png`

---

## 📊 False Positive Tuning Results

| Rule ID | Description | Alerts Before | Action | Alerts After |
|---------|-------------|--------------|--------|-------------|
| 92219 | Windows Update DLL | 116 | Excluded | 0 |
| 60106 | Service Logon Success | 88 | Excluded | 0 |
| 92307 | Svchost Service Creation | 69 | Excluded | 0 |
| 60642 | SW Protection Service | 24 | Excluded | 0 |
| **Total** | | **1,769** | | **1,438** |

**Result: 18.7% false positive reduction** while maintaining full detection coverage across all 6 MITRE ATT&CK techniques.

---

## 📁 Repository Structure
SOC-Detection-Lab/
├── wazuh-rules/
├── README.md # This file
├── detection-playbook.md # Full SOC detection playbook per technique
├── sigma-rules.yml # Custom Sigma detection rules (vendor-neutral)
│ └── local_rules.xml # Wazuh XML custom rules (deploy-ready)
└── screenshots/ # Evidence screenshots from lab


---

## 🤝 How to Contribute

Contributions are welcome! Here's how:

1. **Fork** this repository
2. **Clone** your fork:
   ```bash
   git clone https://github.com/YOUR-USERNAME/SOC-Detection-Lab.git
   ```
3. **Create a branch:**
   ```bash
   git checkout -b feature/new-detection-rule
   ```
4. **Add your contribution** — new Sigma rules, improved playbook entries, additional attack simulations
5. **Test your rule** against Atomic Red Team before submitting
6. **Submit a Pull Request** with:
   - Description of the ATT&CK technique covered
   - Evidence the rule fires correctly
   - False positive analysis

**Ideas for contributions:**
- New Sigma rules for uncovered ATT&CK techniques
- Linux-specific detection rules (auditd)
- Additional false positive exclusions
- Improved dashboard configurations

---

## 📚 Lessons Learned

1. **Swap space is critical** — Wazuh indexer (OpenSearch) needs at least 4GB RAM or equivalent swap to start reliably on VMs
2. **Sysmon dramatically improves Windows visibility** — without it, many ATT&CK techniques are invisible to Wazuh
3. **False positive tuning is a continuous process** — never a one-time task
4. **MITRE ATT&CK mapping helps prioritize** — focus on high-frequency attacker techniques first
5. **Custom rules beat default rules** — narrow, targeted rules produce fewer false positives than broad default ones

---


👤 **Author**

**Samuel M. Ntuen**

Cybersecurity | Blue Team | SOC Engineering  
[LinkedIn](https://www.linkedin.com/in/samuelntuen/) | [GitHub](https://github.com/Official-Samuel-Ntuen)

---

## 📄 License

This project is licensed under the MIT License — feel free to use, modify and distribute with attribution.
EOF
---
