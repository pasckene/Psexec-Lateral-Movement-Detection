
---

# 🔎 Psexec Lateral Movement Detection Case Study

## PsExec Abuse Investigation via SMB & NTLM Analysis

![Status](https://img.shields.io/badge/Status-Completed-success)
![Focus](https://img.shields.io/badge/Focus-Detection%20Engineering-blue)
![Skill](https://img.shields.io/badge/Skill-PCAP%20Analysis-orange)
![Protocol](https://img.shields.io/badge/Protocol-SMB%2FNTLM-lightgrey)
![Tool](https://img.shields.io/badge/Tool-Wireshark-yellowgreen)

---

## 📌 Executive Summary

This project documents a full packet-level investigation of a real-world lateral movement scenario involving **PsExec abuse over SMB**.

An IDS alert flagged suspicious SMB activity indicating potential remote service execution. Through deep PCAP analysis, I reconstructed the attacker’s movement across the network, identified compromised credentials, traced service installation artifacts, and determined the extent of lateral traversal attempts.

The investigation demonstrates:

* Advanced SMB2 and NTLM traffic analysis
* Lateral movement detection via ADMIN$ and IPC$ abuse
* Credential misuse identification
* Service creation artifact discovery (PSEXESVC.exe)
* Attack chain reconstruction using packet forensics

This case study highlights hands-on SOC-level incident analysis skills aligned with enterprise detection engineering workflows.

---

# 🧠 Attack Scenario Overview

An attacker leveraged:

* **SMB (TCP 445)**
* **NTLM Authentication**
* **Administrative Shares (ADMIN$)**
* **IPC$ for inter-process communication**
* **PsExec service installation**

The goal: **Remote command execution and lateral movement.**

---

# 🗺️ Attack Chain Reconstruction

### Initial Access

* Source IP: `10.0.0.130`
* SMB negotiation initiated toward internal host

### First Pivot

* Target: `10.0.0.133`
* Hostname identified via NTLM metadata: **SALES-PC**
* Compromised account: `ssales`
* Service installed: `PSEXESVC.exe`
* Share used: `ADMIN$`
* Communication channel: `IPC$`

### Second Pivot Attempt

* Target: `10.0.0.131`
* Hostname: **MARKETING-PC**
* Result: `STATUS_LOGON_FAILURE`
* Attack unsuccessful

---

# 🔍 Technical Deep Dive

---

## Q1 — Initial Compromise Source

Using **Wireshark → Statistics → Protocol Hierarchy**, SMB2 traffic over TCP 445 was identified.

SMB Negotiate Request originated from:

```
10.0.0.130 → 10.0.0.133
```

This confirms the attacker-controlled machine was:

> **10.0.0.130**

📸 *Screenshot Placeholder – SMB Protocol Hierarchy*
`/screenshots/protocol_hierarchy.png`

---

## Q2 — First Pivot Hostname

Following the TCP stream revealed NTLM authentication exchange.

Extracted NTLM metadata:

* NetBIOS Computer Name: **SALES-PC**
* DNS Name: SALES-PC

This confirms the first lateral movement target.

📸 *Screenshot Placeholder – NTLM Metadata Extraction*
`/screenshots/ntlm_sales_pc.png`

---

## Q3 — Compromised Username

From SMB2 Session Setup Request:

* Account used: **ssales**
* Origin host: HR-PC

This indicates credential compromise and misuse for lateral movement.

📸 *Screenshot Placeholder – SMB2 Session Setup Request*
`/screenshots/ntlm_username.png`

---

## Q4 — Service Installed on Target

SMB2 Create Request revealed creation of:

```
PSEXESVC.exe
```

This is PsExec’s service binary copied to the target system to enable remote execution.

📸 *Screenshot Placeholder – SMB Create Request*
`/screenshots/psexesvc_creation.png`

---

## Q5 — Share Used for Service Installation

Tree Connect Request shows:

```
\\10.0.0.133\ADMIN$
```

ADMIN$ is mapped to:

```
C:\Windows
```

This confirms standard PsExec lateral movement behavior.

📸 *Screenshot Placeholder – ADMIN$ Tree Connect*
`/screenshots/admin_share.png`

---

## Q6 — Communication Share Used

Tree Connect Request shows usage of:

```
\\10.0.0.133\IPC$
```

IPC$ enables:

* Remote procedure calls
* Service control communication
* Inter-process coordination

📸 *Screenshot Placeholder – IPC$ Connection*
`/screenshots/ipc_share.png`

---

## Q7 — Second Pivot Attempt

New SMB session initiated:

```
10.0.0.130 → 10.0.0.131
```

Extracted Hostname:

> **MARKETING-PC**

Authentication result:

```
STATUS_LOGON_FAILURE
```

This indicates attempted but unsuccessful lateral movement.

📸 *Screenshot Placeholder – Failed NTLM Authentication*
`/screenshots/marketing_pc_failure.png`

---

# 🧬 MITRE ATT&CK Mapping

| Technique                    | ID        | Evidence              |
| ---------------------------- | --------- | --------------------- |
| Lateral Movement             | T1021.002 | SMB usage             |
| Remote Service Execution     | T1569.002 | PSEXESVC.exe creation |
| Use of Administrative Shares | T1021     | ADMIN$ abuse          |
| Valid Accounts               | T1078     | ssales authentication |
| Credential Abuse             | T1550     | NTLM session          |

---

# 🛡️ Detection Engineering Insights

### High-Fidelity Detection Opportunities

1. Alert on SMB write to ADMIN$ containing `PSEXESVC.exe`
2. Detect multiple SMB session setups using same account across hosts
3. Monitor NTLM authentication anomalies
4. Alert on IPC$ tree connect immediately after ADMIN$ file creation
5. Flag STATUS_LOGON_FAILURE following successful pivot

---

# 🧰 Tools Used

* Wireshark
* TCP Stream Reconstruction
* SMB2 Packet Analysis
* NTLM Challenge-Response Inspection
* Manual IoC Extraction

---

# 📊 Indicators of Compromise (IoCs)

| Type      | Value        |
| --------- | ------------ |
| Source IP | 10.0.0.130   |
| Target 1  | SALES-PC     |
| Target 2  | MARKETING-PC |
| Username  | ssales       |
| Service   | PSEXESVC.exe |
| Shares    | ADMIN$, IPC$ |
| Protocol  | SMB2         |
| Port      | TCP 445      |

---

# 🧠 Key Takeaways

* PsExec abuse leaves clear SMB artifacts
* ADMIN$ and IPC$ monitoring is critical
* NTLM metadata reveals pivot targets
* Failed authentication attempts reveal expansion intent
* PCAP analysis can fully reconstruct lateral movement chains

---

# 📈 Skills Demonstrated

* Network forensic investigation
* Protocol-level attack reconstruction
* Lateral movement detection
* NTLM authentication analysis
* SMB2 traffic dissection
* Incident reporting & documentation

---

# 🚀 Why This Project Matters

Modern enterprise environments rely heavily on SMB for internal operations. PsExec abuse is a common technique used by ransomware operators and APT groups for lateral movement.

Understanding and detecting:

* ADMIN$ abuse
* NTLM misuse
* Remote service creation

is critical for SOC teams and detection engineers.

---

# 🏁 Final Outcome

The attacker:

✔ Successfully pivoted to SALES-PC
✔ Installed PSEXESVC.exe
✔ Used ADMIN$ and IPC$
✔ Attempted pivot to MARKETING-PC
❌ Failed authentication on second pivot

This investigation successfully reconstructed the attack chain and identified all pivot attempts.

---

