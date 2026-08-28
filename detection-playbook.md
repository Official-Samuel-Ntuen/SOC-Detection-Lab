# SOC Detection Playbook
**Author:** Samuel M. Ntuen  
**Date:** August 2026  
**Environment:** Wazuh 4.9.2 | Windows 11 + Kali Linux endpoints

---

## Threat 1: Suspicious PowerShell Execution (T1059.001)

### Threat Profile
Attackers use PowerShell to download malware, execute encoded commands, and bypass security controls. Commonly seen in ransomware, APT campaigns, and living-off-the-land attacks.

### Log Sources Required
- Windows Event Log (Event ID 4688)
- Sysmon Event ID 1 (Process Creation)
- Wazuh rule group: windows

### Detection Logic
```xml
<rule id="100001" level="12">
  <field name="win.eventdata.image">powershell.exe</field>
  <field name="win.eventdata.commandLine">encodedcommand|bypass|hidden|iex</field>
  <description>Suspicious PowerShell execution - T1059.001</description>
</rule>
```
Detects PowerShell launched with suspicious flags commonly used by attackers.

### False Positive Mitigation
- Exclude known admin scripts by whitelisting specific parent processes
- Exclude scheduled maintenance scripts by path

### Test Evidence
- Tested using Atomic Red Team: `Invoke-AtomicTest T1059.001 -TestNumbers 1`
- Alert fired at rule level 12 within 30 seconds

---

## Threat 2: Brute Force Authentication (T1110)

### Threat Profile
Attackers attempt repeated logins to guess credentials. Detected via multiple failed logon events from same source.

### Log Sources Required
- Windows Event ID 4625 (Failed Logon)
- Wazuh rule: 60122

### Detection Logic
```xml
<rule id="100002" level="12" frequency="5" timeframe="60">
  <if_matched_sid>60122</if_matched_sid>
  <description>Multiple failed logons - possible brute force T1110</description>
</rule>
```
Fires when 5+ failed logons occur within 60 seconds.

### False Positive Mitigation
- Exclude service accounts with known frequent auth failures
- Whitelist internal monitoring systems

### Test Evidence
- Tested using Hydra: `hydra -l administrator -P rockyou.txt rdp://192.168.56.102`
- Generated 331 failed logon alerts
- User account lockout detected (rule level 9)

---

## Threat 3: Credential Dumping via LSASS (T1003.001)

### Threat Profile
Attackers access LSASS memory to steal credentials. Used by Mimikatz, CobaltStrike, and most post-exploitation frameworks.

### Log Sources Required
- Sysmon Event ID 10 (Process Access)
- Wazuh rule group: sysmon

### Detection Logic
```xml
<rule id="100003" level="15">
  <field name="win.eventdata.targetImage">lsass.exe</field>
  <field name="win.eventdata.grantedAccess">0x1010|0x1410</field>
  <description>Possible LSASS credential dumping - T1003.001</description>
</rule>
```
Detects suspicious memory access to lsass.exe with credential-dumping access masks.

### False Positive Mitigation
- Exclude legitimate security tools (AV, EDR) by process name
- Whitelist known admin tools

### Test Evidence
- Tested using Atomic Red Team: `Invoke-AtomicTest T1003.001 -TestNumbers 1`

---

## Threat 4: Scheduled Task Persistence (T1053.005)

### Threat Profile
Attackers create scheduled tasks to maintain persistence after reboot. Common in malware and APT campaigns.

### Log Sources Required
- Windows Event ID 4698 (Scheduled Task Created)
- Sysmon Event ID 1 (schtasks.exe execution)

### Detection Logic
```xml
<rule id="100004" level="10">
  <field name="win.eventdata.image">schtasks.exe</field>
  <field name="win.eventdata.commandLine">/create</field>
  <description>Scheduled task created - T1053.005</description>
</rule>
```

### False Positive Mitigation
- Exclude known software update schedulers
- Whitelist IT admin accounts

### Test Evidence
- Tested using Atomic Red Team: `Invoke-AtomicTest T1053.005 -TestNumbers 1`
- Alert fired at rule level 10

---

## False Positive Tuning Summary

| Rule | Alerts Before | Suppressed | Result |
|------|--------------|------------|--------|
| 92219 Windows Update DLL | 116 | 116 | Excluded |
| 60106 Service Logon | 88 | 88 | Excluded |
| 92307 Svchost Service | 69 | 69 | Excluded |
| 60642 SW Protection | 24 | 24 | Excluded |
| **Total** | **1,769** | **331** | **18.7% reduction** |

---

## Architecture

```
Kali Linux (Attacker + Linux Endpoint)
         │
         ▼
Wazuh Manager (192.168.56.101)
         ▲
         │
Windows 11 (Victim Endpoint + Sysmon)
```
