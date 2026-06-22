# Sentinel-Defender-XDR-SOC-Incident-Response-lab

<div align="center">

# 🛡️ Microsoft Sentinel & Defender XDR – SOC Incident Response Lab: Brute Force Attack Detection & Response

![Domain](https://img.shields.io/badge/SIEM-Microsoft%20Sentinel-blue?style=for-the-badge)
![Platform](https://img.shields.io/badge/Portal-Microsoft%20Defender%20XDR-red?style=for-the-badge)
![Type](https://img.shields.io/badge/Type-SOC%20Incident%20Response%20Lab-darkblue?style=for-the-badge)

<img src="https://img.shields.io/badge/Attack-RDP%20Brute%20Force%20(Hydra)-red?style=flat-square" />
<img src="https://img.shields.io/badge/Detection-KQL%20Analytics%20Rule-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Severity-High-critical?style=flat-square" />
<img src="https://img.shields.io/badge/MITRE-T1110%20Brute%20Force-orange?style=flat-square" />
<img src="https://img.shields.io/badge/Incident%20ID-18-purple?style=flat-square" />
<img src="https://img.shields.io/badge/Outcome-Detected%20%26%20Resolved-green?style=flat-square" />

---

**Prepared by:** Bikash Raya
**Project Type:** SOC Lab — Brute Force Attack Simulation, Detection, Analytics Rule Creation & Incident Response

</div>

---

## 📁 Repository Structure

| File | Description |
| --- | --- |
| [Sentinel_Defender_SOC_IR_Lab_Report.pdf](./Sentinel_Defender_SOC_IR_Lab_Report.pdf) | Complete lab report with all screenshots embedded |
| README.md | Project overview |

---

## 📋 Overview

This repository documents a complete **end-to-end SOC Incident Response lab** using **Microsoft Sentinel** and the **Microsoft Defender XDR unified portal**. The lab simulates a real-world attack scenario where an internet-facing Windows Server is targeted with an RDP brute force attack, detected via custom KQL analytics rules, and responded to using the Defender XDR incident management workflow.

The lab covers:

* 🏗️ Deployed a **Resource Group, Windows Server VM (Web01)** and **Ubuntu Attacker VM** in Azure
* 🔓 Configured an intentionally vulnerable **NSG inbound rule (Any/Any/*)** and **disabled Windows Firewall** on Web01
* 📊 Created a **Log Analytics Workspace (Bik-SOC-Lab)** and onboarded **Microsoft Sentinel**
* 🔌 Installed **Windows Security Events via AMA** data connector and created a **Data Collection Rule**
* ⚔️ Launched an **RDP brute force attack** from AttackerVM using **Hydra** and a real-world password list
* 🔍 Verified attack logs in **Sentinel using KQL** (Event IDs 4624/4625 — failed + successful logins)
* 🚨 Created a **Scheduled Analytics Rule** mapped to **MITRE ATT&CK T1110** that generated a High-severity incident
* 🛡️ Investigated **Incident ID 18** in the **Defender XDR unified portal**
* 📋 Performed simulated **incident response**: assigned task, documented investigation, closed incident

---

## 🛠️ Technologies Used

* Microsoft Azure (Azure Portal)
* Microsoft Sentinel
* Microsoft Defender XDR (security.microsoft.com — unified portal)
* Microsoft Defender for Cloud
* Log Analytics Workspace
* Windows Security Events via AMA (Data Connector)
* Data Collection Rule (DCR)
* KQL (Kusto Query Language)
* Hydra v9.5 (THC-Hydra — RDP brute force tool)
* Azure Bastion (secure VM access)
* MITRE ATT&CK Framework
* Ubuntu 24.04 LTS (Attacker VM)
* Windows Server 2022 Datacenter (Victim VM)

---

## 🧪 Lab Environment

| Component | Details |
| --- | --- |
| Resource Group | Security-BikSecOps (Australia East) |
| Victim VM | Web01 -- Windows Server 2022 Datacenter (Standard D2s v3) |
| Victim Public IP | 20.5.48.127 |
| NSG Inbound Rule | Any / Any / * / Allow (Priority 100 -- Named: Dangerous) |
| Windows Firewall | Disabled on Web01 (Public + Private profiles) |
| Attacker VM | AttackerVM -- Ubuntu 24.04 LTS |
| Attack Tool | Hydra v9.5 (THC-Hydra) |
| Log Analytics Workspace | Bik-SOC-Lab (australiaeast) |
| SIEM | Microsoft Sentinel |
| Unified Portal | Microsoft Defender XDR |
| Data Connector | Windows Security Events via AMA |
| DCR Name | SecurityEventsforAzureVM -- All Events |

---

## 🌐 Lab Architecture

```
[AttackerVM -- Ubuntu 24.04]
         │
         │  Hydra RDP Brute Force
         │  rdp://20.5.48.127:3389
         ▼
[Web01 -- Windows Server 2022]
  NSG: Any/Any/* Allow
  Firewall: OFF
  Public IP: 20.5.48.127
         │
         │  Windows Security Events (4624, 4625)
         │  via AMA + DCR (SecurityEventsforAzureVM)
         ▼
[Log Analytics Workspace -- Bik-SOC-Lab]
         │
         │  Microsoft Sentinel ingests logs
         ▼
[Microsoft Sentinel -- Analytics Rule]
  Rule: Successful Brute Force Attempt
  KQL: EventID 4624/4625 | summarize by TargetUserName, IpAddress
  Trigger: FailedAttempts >= 7 AND SuccessAttempts > 0 AND SuccessTime > LastFailed
         │
         │  Alert --> Incident Created
         ▼
[Microsoft Defender XDR -- Incident ID 18]
  Severity: High | Status: Active --> Resolved
  Entity: bikashsoc1 | IP: 20.70.152.193
  Task: User/Host Review | Classification: Expected Activity
```

---

## 🔬 Lab Phases

### Phase 1 — Infrastructure Deployment

- Created Resource Group **Security-BikSecOps** (Australia East)
- Deployed **Web01** (Windows Server 2022) with deliberately insecure NSG:
  - Inbound Rule: Source Any / Destination Any / Port * / Action Allow / Name: **Dangerous**
- RDP'd into Web01 and **disabled Windows Defender Firewall** (Public + Private profiles)
- Deployed **AttackerVM** (Ubuntu 24.04 LTS) in the same resource group

---

### Phase 2 — Sentinel & Log Analytics Setup

- Created Log Analytics Workspace: **Bik-SOC-Lab** (australiaeast)
- Added Bik-SOC-Lab to **Microsoft Sentinel**
- Installed **Windows Security Events** solution from Content Hub
- Created Data Collection Rule **SecurityEventsforAzureVM**:
  - Resource: Web01
  - Events: All Events

---

### Phase 3 — Attack Simulation (Hydra RDP Brute Force)

Connected to AttackerVM via **Azure Bastion**, then:

```bash
sudo apt update
sudo apt install hydra
```

Downloaded common password list and added the correct password to guarantee a successful login:

```bash
wget raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/2025-199_most_used_passwords.txt
nano 2025-199_most_used_passwords.txt   # added: bikashsoc123
```

Launched RDP brute force attack:

```bash
hydra -l bikashsoc1 -P 2025-199_most_used_passwords.txt -t 1 -W 5 rdp://20.5.48.127:3389
```

**Result:** `1 of 1 target successfully completed. 1 valid password found` — 199 attempts, password cracked.

---

### Phase 4 — KQL Threat Hunting & Log Verification

Verified the attack in Sentinel Logs using KQL:

```kql
SecurityEvent
| where TimeGenerated > ago(15m)
| where EventID in (4624, 4625)
| summarize
    FailedAttempts  = countif(EventID == 4625),
    SuccessAttempts = countif(EventID == 4624),
    FirstFailure    = minif(TimeGenerated, EventID == 4625),
    LastSuccess     = maxif(TimeGenerated, EventID == 4624)
  by TargetUserName, IpAddress
| where FailedAttempts >= 7
| where SuccessAttempts > 0
| where LastSuccess > FirstFailure
```

**Query Result:**

| TargetUserName | IpAddress | FailedAttempts | SuccessAttempts | FirstFailure | LastSuccess |
| --- | --- | --- | --- | --- | --- |
| bikashsoc1 | 20.70.152.193 | 7 | 1 | 6/22/2026 1:53:24 AM | 6/22/2026 1:54:00 AM |

✅ Brute force attack confirmed in logs — 7 failed logins followed by 1 successful login from the same IP.

---

### Phase 5 — Analytics Detection Rule

Created a **Scheduled Analytics Rule** in Microsoft Sentinel (via Defender XDR unified portal):

| Property | Value |
| --- | --- |
| Rule Name | Successful Brute Force Attempt |
| Severity | High |
| Status | Enabled |
| MITRE Tactics | Credential Access \| Initial Access |
| MITRE Techniques | T1110 \| T1110.001 \| T1110.003 \| T1078 |
| Run Frequency | Every 5 minutes |
| Lookback Period | Last 5 hours |
| Entity Mapping | Account: TargetUserName \| IP: IpAddress |

**Analytics Rule KQL:**

```kql
SecurityEvent
| where EventID in (4624, 4625)
| where LogonType in (2, 3, 10)
| summarize
    FailedAttempts  = countif(EventID == 4625),
    SuccessAttempts = countif(EventID == 4624),
    LastFailed      = maxif(TimeGenerated, EventID == 4625),
    SuccessTime     = maxif(TimeGenerated, EventID == 4624)
  by TargetUserName, IpAddress
| where FailedAttempts >= 5
| where SuccessAttempts > 0
| where SuccessTime > LastFailed
| project TargetUserName, IpAddress, FailedAttempts, SuccessAttempts, LastFailed, SuccessTime
| order by SuccessTime desc
```

---

### Phase 6 — Incident Detection & Response (Defender XDR)

**Incident ID 18** was automatically generated in Microsoft Defender XDR:

| Property | Value |
| --- | --- |
| Incident Title | Successful Brute Force Attempt involving one user |
| Severity | 🔴 High |
| Status | Active → Resolved |
| Active Alerts | 2/2 |
| Affected User | bikashsoc1 |
| Attack Source IP | 20.70.151.193 |
| First Activity | Jun 22, 2026 7:57 AM |
| Workspace | Bik-SOC-Lab |

**Investigation Task Created:**
- Task Name: User / Host Review
- Affected Account: bikashsoc1 (Privileged)
- Affected Host: Web01 (Production / External Facing)

**Recommended Containment Actions:**
- Disable affected account
- Reset account password
- Revoke active sessions
- Block source IP address
- Isolate affected host if compromise confirmed
- Escalate to DFIR team if malicious activity validated

**Incident Closed:**
> Investigation completed. Activity confirmed as authorized security testing. Incident closed as **Informational: Expected Activity / Confirmed Activity**.

---

## 🎯 Skills Demonstrated

* Microsoft Sentinel Deployment & Configuration
* Azure VM Deployment & NSG Configuration
* Log Analytics Workspace & DCR Setup
* Windows Security Events via AMA Data Connector
* KQL Threat Hunting (SecurityEvent table — Event IDs 4624/4625)
* Custom Scheduled Analytics Rule Creation
* MITRE ATT&CK Framework Mapping (T1110, T1078)
* Microsoft Defender XDR Unified Portal
* Incident Triage, Investigation & Response
* RDP Brute Force Simulation (Hydra)
* Linux CLI & Azure Bastion
* SOC Tier 1/2 Workflow End-to-End

---

## 🎯 Key Takeaway

> This lab demonstrates practical, hands-on SOC experience using Microsoft Sentinel and Microsoft Defender XDR — covering the full incident response lifecycle from infrastructure deployment and attack simulation through log ingestion, KQL-based threat detection, custom analytics rule creation mapped to MITRE ATT&CK T1110, automated incident generation, and structured incident response in the Defender XDR unified portal. The brute force attack using Hydra generated real Windows Security Events (4624/4625) that were detected and escalated as a High-severity incident, then investigated and closed following SOC procedures.

---

## 🔗 Connect With Me

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/bikash-raya/)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=for-the-badge&logo=github)](https://github.com/Bikash-Raya)

</div>

---

<div align="center">

⭐ If you find this project useful, feel free to star the repository ⭐

</div>
