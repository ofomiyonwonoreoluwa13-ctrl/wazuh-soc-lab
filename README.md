# 🛡️ Wazuh SOC Home Lab — SSH Brute-Force Detection

> A hands-on Security Operations Center (SOC) lab built on **Wazuh SIEM** — deploying the platform, monitoring an endpoint, simulating real attacks from two separate attacker machines, and detecting and investigating them like a SOC analyst.

![Architecture Diagram](screenshots/01-architecture.png)

---

## 🔎 Summary (TL;DR)

I deployed a full **Wazuh SIEM** stack (Indexer + Manager + Dashboard + Filebeat) on Ubuntu, then launched **SSH brute-force attacks from two attacker machines** — a **Windows** host (manual attempts) and a **Kali Linux** box (automated with Hydra). Wazuh detected every attempt, correlated the repeated failures into a **high-severity brute-force alert (level 10)**, and surfaced each attack in the dashboard with the **attacker's source IP, username, port, raw log, and MITRE ATT&CK mapping**.

**Outcome:** a working detection pipeline that proves I can deploy a SIEM, emulate adversary activity, and investigate alerts end-to-end.

---

## 🎯 Objectives

- Stand up a working SIEM / log-analysis platform from scratch
- Monitor a Linux endpoint's authentication activity
- Simulate real-world SSH brute-force attacks from multiple sources
- Detect, correlate, and investigate the attacks in the SIEM
- Map detections to the MITRE ATT&CK framework

---

## 🧱 Architecture

| Role | Machine | OS | IP | Purpose |
|------|---------|-----|-----|---------|
| 🎯 **SOC / Target** | Wazuh Server | Ubuntu 24.04 | `192.168.0.192` | SIEM (Indexer + Manager + Dashboard + Filebeat); the monitored/attacked host |
| 💻 **Attacker 1** | Windows Host | Windows 11 | `192.168.0.10` | Manual SSH brute-force via PowerShell |
| 🐉 **Attacker 2** | Kali Linux | Kali 2026.1 | `192.168.0.238` | Automated brute-force with THC-Hydra |

All machines run in **Oracle VirtualBox** on a **Bridged network**, placing them on the same LAN so the attackers reach the target directly and Wazuh records each attacker's real source IP.

### Detection data flow

```
SSH login attempts  →  /var/log/auth.log  (Ubuntu target)
      →  Wazuh Manager  (analysisd applies detection rules)
      →  alerts.json
      →  Filebeat  →  Wazuh Indexer (OpenSearch)
      →  Wazuh Dashboard  (Threat Hunting / Security Events)
```

---

## 🛠️ Tech Stack

- **SIEM:** Wazuh 4.14 — Indexer, Manager, Dashboard
- **Log shipper:** Filebeat (OSS)
- **Endpoint:** Ubuntu 24.04 (OpenSSH server)
- **Attack tooling:** OpenSSH client (Windows PowerShell), THC-Hydra (Kali)
- **Virtualization:** Oracle VirtualBox (Bridged networking)
- **Framework:** MITRE ATT&CK

---

## ⚙️ Part 1 — Deploying the SIEM

I deployed the Wazuh central components on Ubuntu — Indexer, Manager, and Dashboard — secured with TLS certificates between components, and configured Filebeat to ship alerts into the Indexer for the dashboard to visualize.

**Wazuh dashboard login (TLS-secured web UI):**
![Wazuh login](screenshots/02-wazuh-login.png)

**Core services running — Indexer and Dashboard:**
![Indexer active](screenshots/03-indexer-active.png)
![Dashboard active](screenshots/04-dashboard-active.png)

**TLS certificate/key pair verified for the dashboard, listening on 443:**
![Dashboard cert verify](screenshots/07-dashboard-cert-verify.png)

**Wazuh overview console:**
![Wazuh overview](screenshots/06-wazuh-overview.png)

**All components active and the host on the bridged LAN (`192.168.0.192`):**
![Services active](screenshots/08-services-active.png)

---

## ⚔️ Part 2 — Exposing & attacking the target

OpenSSH was enabled on the target so authentication attempts are logged and monitored.

**SSH server active and listening on port 22:**
![SSH active](screenshots/05-ssh-active.png)

### Attacker 1 — Windows (manual)
Repeated SSH login attempts from Windows PowerShell as a non-existent user:
```powershell
ssh hacker@192.168.0.192
```
![Windows SSH attack](screenshots/10-windows-ssh-attack.png)

### Attacker 2 — Kali Linux (automated with Hydra)
A real automated brute-force using THC-Hydra against SSH:
```bash
hydra -l hacker -P pass.txt ssh://192.168.0.192 -t 4
```
![Kali Hydra attack](screenshots/11-kali-hydra-attack.png)

**Underlying failed-login events captured in the host's auth log:**
![Auth log failed logins](screenshots/09-auth-log-failed-logins.png)

---

## 🚨 Part 3 — Detection & investigation

Wazuh detected **both** attackers and correlated the activity.

| Wazuh Rule | Description | Level | MITRE ATT&CK |
|------------|-------------|:-----:|--------------|
| **5710** | sshd: Attempt to login using a non-existent user | 5 | T1110 – Brute Force |
| **5712** | sshd: Brute force trying to get access to the system | **10** | T1110.001 – Password Guessing |

**Threat Hunting dashboard — alerts and Top MITRE ATT&CK techniques:**
![Threat hunting dashboard](screenshots/12-threat-hunting-dashboard.png)

**Security events — repeated 5710 attempts correlated into the level-10 brute-force rule 5712:**
![Detection events 5710 / 5712](screenshots/13-detection-events-5710-5712.png)

**Investigating the Windows attacker — source IP `192.168.0.10`:**
![Windows attack detail](screenshots/14-windows-attack-detail.png)

**Investigating the Kali attacker — source IP `192.168.0.238`:**
![Kali attack detail](screenshots/15-kali-attack-detail.png)

> The level-10 rule (5712) is the key SOC signal: Wazuh didn't just log individual failures — it recognized the *pattern* of repeated failures as a brute-force attack and raised a high-severity, correlated alert. Each alert carries the attacker's **source IP, username, port, and the raw log line** for investigation.

---

## 🧠 Skills Demonstrated

- SIEM deployment & configuration (Wazuh / OpenSearch)
- Log collection & data-pipeline engineering (Filebeat → Indexer)
- Detection engineering (rule-based alerting & correlation)
- Attack simulation / adversary emulation (manual + automated brute-force)
- Threat hunting & alert investigation in a SIEM
- MITRE ATT&CK mapping
- Linux administration, networking, and virtualization

---

## 🚀 Next Steps

- Deploy Wazuh **agents** on Windows and Kali to monitor the attacker hosts too
- Add **active response** to auto-block brute-force source IPs
- Detect additional techniques (port scans, web attacks, privilege escalation)
- Build custom dashboards and scheduled reporting
- Integrate threat-intelligence feeds

---

*Built as a hands-on cybersecurity home lab to develop and demonstrate SOC analyst skills.*
