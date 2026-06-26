# 🛡️ Suricata IDS Home Lab

> A hands-on Network Intrusion Detection System (IDS) lab built with Suricata, simulating real-world attacks in a virtualized environment and detecting them using custom-written rules.

![Suricata](https://img.shields.io/badge/Suricata-IDS%2FIPS-orange?style=flat-square&logo=suricata)
![Platform](https://img.shields.io/badge/Platform-Ubuntu%2022.04-blue?style=flat-square&logo=ubuntu)
![Attacker](https://img.shields.io/badge/Attacker-Kali%20Linux-black?style=flat-square&logo=kalilinux)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)

---

## 📌 Project Overview

This project demonstrates the deployment and configuration of **Suricata** as a Network IDS in a controlled home lab environment. Four attack scenarios were simulated from a Kali Linux attacker VM against an Ubuntu Server defender VM, with all attacks successfully detected using custom Suricata detection rules.

### 🎯 Objectives
- Deploy and configure Suricata IDS on Ubuntu Server
- Write custom detection rules targeting specific attack signatures
- Simulate real-world network attacks from Kali Linux
- Analyze alert logs and produce a structured incident report

---

## 🖥️ Lab Environment

```
┌─────────────────────────────────────────────────────┐
│                   VirtualBox Host                    │
│                                                      │
│  ┌──────────────────┐    ┌──────────────────────┐   │
│  │   Kali Linux     │    │   Ubuntu Server      │   │
│  │   (Attacker)     │◄──►│   (Defender/IDS)     │   │
│  │  192.168.56.102  │    │   192.168.56.101     │   │
│  └──────────────────┘    └──────────────────────┘   │
│            Host-Only Network: 192.168.56.0/24        │
└─────────────────────────────────────────────────────┘
```

| Component       | Details                        |
|-----------------|-------------------------------|
| Hypervisor      | VirtualBox 7.x                |
| Defender OS     | Ubuntu Server 22.04 LTS       |
| Attacker OS     | Kali Linux 2024.x             |
| IDS Engine      | Suricata 7.x                  |
| Network Mode    | Host-Only Adapter             |
| Defender IP     | 192.168.56.101                |
| Attacker IP     | 192.168.56.102                |

---

## 🛠️ Tools Used

| Tool        | Purpose                              |
|-------------|--------------------------------------|
| Suricata    | IDS engine, rule matching, alerting  |
| Nmap        | Port scanning & ping sweep simulation|
| Hydra       | SSH brute force simulation           |
| Netcat      | Reverse shell traffic simulation     |
| jq          | JSON log parsing                     |
| Wireshark   | Packet-level traffic analysis        |

---

## 📁 Repository Structure

```
suricata-ids-lab/
├── README.md                        ← This file
├── configs/
│   └── suricata.yaml                ← Sanitized Suricata config
├── rules/
│   └── custom.rules                 ← All custom detection rules
├── logs/
│   └── sample_alerts.log            ← Sample alert output (fast.log)
│   └── sample_eve.json              ← Sample JSON event log (eve.json)
├── diagrams/
│   └── lab_topology.png             ← Network topology diagram
└── report/
    └── Suricata_IDS_Lab_Report.pdf  ← Full professional report
```

---

## 🚀 Setup & Installation

### Prerequisites
- VirtualBox or VMware installed
- Ubuntu Server 22.04 ISO
- Kali Linux ISO
- Both VMs connected via Host-Only Network

### Step 1 — Install Suricata

```bash
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update && sudo apt install suricata -y
suricata --build-info
```

### Step 2 — Update Community Rules

```bash
sudo suricata-update
```

### Step 3 — Configure Suricata

Edit `/etc/suricata/suricata.yaml`:

```yaml
vars:
  address-groups:
    HOME_NET: "[192.168.56.0/24]"

af-packet:
  - interface: eth0
```

Add custom rules path:

```yaml
rule-files:
  - suricata.rules
  - /etc/suricata/rules/custom.rules
```

### Step 4 — Deploy Custom Rules

Copy `rules/custom.rules` to `/etc/suricata/rules/custom.rules`, then restart:

```bash
sudo systemctl restart suricata
sudo systemctl status suricata
```

### Step 5 — Monitor Alerts

```bash
# Simple fast log
sudo tail -f /var/log/suricata/fast.log

# Structured JSON log
sudo tail -f /var/log/suricata/eve.json | jq .
```

---

## ⚔️ Attack Scenarios & Detection Results

### Attack 1 — Nmap SYN Port Scan

**Command (Kali):**
```bash
nmap -sS 192.168.56.101
```

**Rule:**
```
alert tcp any any -> $HOME_NET any (msg:"NMAP SYN Scan Detected"; flags:S; threshold: type both, track by_src, count 20, seconds 2; sid:1000001; rev:1;)
```

**Result:** ✅ Detected — Alert fired within 2 seconds of scan initiation

---

### Attack 2 — ICMP Ping Sweep

**Command (Kali):**
```bash
nmap -sn 192.168.56.0/24
```

**Rule:**
```
alert icmp any any -> $HOME_NET any (msg:"ICMP Ping Sweep Detected"; itype:8; threshold: type both, track by_src, count 5, seconds 2; sid:1000002; rev:1;)
```

**Result:** ✅ Detected — 5+ ICMP packets in 2 seconds triggered alert

---

### Attack 3 — SSH Brute Force

**Command (Kali):**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101
```

**Rule:**
```
alert tcp any any -> $HOME_NET 22 (msg:"SSH Brute Force Attempt"; flow:to_server; threshold: type both, track by_src, count 5, seconds 60; sid:1000003; rev:1;)
```

**Result:** ✅ Detected — Multiple rapid SSH connection attempts flagged

---

### Attack 4 — Reverse Shell Simulation

**Command (Kali):**
```bash
echo "/bin/sh" | nc 192.168.56.101 4444
```

**Rule:**
```
alert tcp any any -> $HOME_NET any (msg:"Possible Reverse Shell - Netcat /bin/sh"; content:"/bin/sh"; sid:1000004; rev:1;)
```

**Result:** ✅ Detected — Payload signature matched in packet content

---

## 📊 Detection Summary

| # | Attack Type       | Tool Used | Rule SID | Detected |
|---|-------------------|-----------|----------|----------|
| 1 | SYN Port Scan     | Nmap      | 1000001  | ✅ Yes   |
| 2 | ICMP Ping Sweep   | Nmap      | 1000002  | ✅ Yes   |
| 3 | SSH Brute Force   | Hydra     | 1000003  | ✅ Yes   |
| 4 | Reverse Shell     | Netcat    | 1000004  | ✅ Yes   |

**Detection Rate: 4/4 (100%)**

---

## 📋 Sample Alert Output

```
07/01/2025-14:32:11.456789  [**] [1:1000001:1] NMAP SYN Scan Detected [**] [Classification: Attempted Information Leak] [Priority: 2] {TCP} 192.168.56.102:54231 -> 192.168.56.101:80

07/01/2025-14:33:45.123456  [**] [1:1000003:1] SSH Brute Force Attempt [**] [Classification: Attempted Administrator Privilege Gain] [Priority: 1] {TCP} 192.168.56.102:51234 -> 192.168.56.101:22
```

---

## 🔒 Defensive Recommendations

Based on the findings, the following hardening measures are recommended:

1. **Block aggressive scanners** — Use `drop` rules instead of `alert` for known malicious IPs
2. **Implement fail2ban** — Auto-ban IPs after repeated SSH failures
3. **Disable root SSH login** — Set `PermitRootLogin no` in `/etc/ssh/sshd_config`
4. **Rate-limit ICMP** — Limit ICMP responses via iptables
5. **Network segmentation** — Isolate critical services on separate VLANs
6. **Log forwarding** — Ship Suricata `eve.json` to a SIEM (e.g., Splunk, ELK)

---

## 📄 Full Report

A detailed professional report covering methodology, evidence, and recommendations is available here:
📎 [`report/Suricata_IDS_Lab_Report.pdf`](report/Suricata_IDS_Lab_Report.pdf)

---

## ⚠️ Disclaimer

> All attacks and tests in this project were performed exclusively in a **controlled, isolated virtual lab environment**. No real systems, networks, or third-party infrastructure were targeted. This project is intended solely for educational and portfolio purposes.

---

## 👤 Author

**Your Name**
- 🔗 LinkedIn: www.linkedin.com/in/chetankharatmal
- 🐙 GitHub: https://github.com/chetankharatmal

---

## 📜 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
