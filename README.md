# Splunk Security Monitoring Lab

## 📌 Project Overview
This project documents the setup and configuration of a **Splunk Security Monitoring Lab** within my home lab environment. My home lab is hosted on my Dell PowerEdge R730XD which is running the Proxmox hypervisor as its OS. I was able to obtain a Splunk developer license to gain access to additional features as well as increased data capacity for storing logs within the Splunk Enterprise environment. 

I designed this lab with the intention of mimicking real-world scnearios involving security monitoring solutions such as **Splunk, Wazuh, Windows Event Logs, and pfSense**. My primary objective is to gain hands-on experience in **log collection, SIEM operations, dashboard visualization, and alerting**.

---

## 🌐 Home Lab Network Overview

### **Network Architecture**

| VLAN | Network | Purpose |
|------|---------|---------|
| VLAN 10 | `10.10.10.0/24` | Internal Systems (Splunk, Wazuh) |
| VLAN 20 | `10.10.20.0/24` | Active Directory & Windows Systems |
| VLAN 30 | `10.10.30.0/24` | External & Attack Simulation |
| DMZ | `10.10.5.0/24` | Public-Facing Web Services |
| WAN | `192.168.1.0/24` | Home Network Connection |

### **Key Systems & IP Assignments**

| Host | Role | IP Address | VLAN |
|------|------|------------|------|
| Splunk Server | SIEM & Log Analysis | `10.10.10.10` | 10 |
| Wazuh Manager | Security Monitoring | `10.10.10.20` | 10 |
| Windows DC | Active Directory & Event Logs | `10.10.20.1` | 20 |
| pfSense Firewall | Network Security | `192.168.1.201` (WAN) / `10.10.10.254` (LAN) | WAN/10 |
| Kali Linux | Attack Simulation | `10.10.30.50` | 30 |

---

## 🏗️ Splunk Lab Architecture

The **Splunk Lab** consists of:

- **Splunk Enterprise** (`10.10.10.10`) – Acts as the primary SIEM, collecting and analyzing logs.
- **Splunk Universal Forwarder** (`Installed on Wazuh Manager & Windows DC`) – Forwards logs to Splunk.
- **Wazuh Manager** (`10.10.10.20`) – Provides **endpoint security monitoring**.
- **Windows Server (DC1)** (`10.10.20.1`) – Generates **Windows Event Logs** for authentication and security monitoring.
- **pfSense Firewall** – Sends **firewall logs** to Splunk for network security analysis.
- **Kali Linux** (`10.10.30.50`) – Used for **attack simulations** (brute-force, scanning, etc.).

---

## ⚙️ Step-by-Step Configurations

### **1️⃣ Splunk Enterprise Installation & Configuration**
- Installed **Splunk Enterprise** on `10.10.10.10`.
- Configured **indexing** for `wineventlog`, `wazuh-alerts`, and `firewall-logs`.
- Enabled **data forwarding** from Universal Forwarders.

### **2️⃣ Installing & Configuring Splunk Universal Forwarder**
- Installed **Splunk Universal Forwarder** on:
  - **Wazuh Manager (`10.10.10.20`)** → Forwards Wazuh logs to Splunk.
  - **Windows DC (`10.10.20.1`)** → Forwards Security Event Logs.
- Configured `inputs.conf` to collect:
  - Security Events (Event ID 4624, 4625, 4740, etc.).
  - System Logs.
  - Directory Services Logs.

### **3️⃣ Wazuh Manager Setup**
- Installed **Wazuh Manager** on `10.10.10.20`.
- Installed **Wazuh Agent** on Windows DC (`10.10.20.1`).
- Integrated **Wazuh with Splunk**.

### **4️⃣ pfSense Log Forwarding**
- Configured pfSense to send **syslog data** to Splunk.
- Enabled **Suricata IDS alerts** in Splunk.

### **5️⃣ Custom Field Extraction for Source IP in Event Logs**
- Used `rex` in Splunk to extract `Source Network Address` from Windows Event Logs:
  ```spl
  index=wineventlog sourcetype="WinEventLog:Security" EventCode=4625
  | rex field=_raw "Source\s+Network\s+Address:\s+(?<src_ip>\S+)"
  | stats count by src_ip
  | sort -count
  | head 5
  ```

---

## 📊 Custom Dashboards

### **1️⃣ Domain Controller Dashboard**
- **Failed Logon Attempts** (`EventCode=4625`)
- **Account Lockouts** (`EventCode=4740`) - Haven't produced logs for this yet
- **Successful Logins** (`EventCode=4624`)
- **Group Policy Changes** - Haven't produced logs for this yet

![Splunk Server Dashboard](https://imgur.com/2z95g5g.jpg)

![Splunk Endpoint Dashboard](https://imgur.com/4kvPJ7V.jpg)

### **2️⃣ Network Security Dashboard** - Still in Progress
- **Top Denied Firewall Connections**
- **Suricata IDS Alerts**
- **Open Ports & Scans from Kali Linux**

### **3️⃣ Wazuh Security Events Dashboard**
- **Endpoint Overview** from Wazuh.
- **File Integrity Monitoring Recent Events**.

![Wazuh Dashboard](https://imgur.com/CATMYBH.jpg)

---

## ⚠️ Alerts & Monitoring

### **1️⃣ Brute Force Detection (Failed RDP Logins)**
- Query:
  ```spl
  index=wineventlog sourcetype="WinEventLog:Security" EventCode=4625 Logon_Type=10
  | stats count by src_ip
  | where count > 5
  | sort -count
  ```

### **2️⃣ Account Lockout Alert**
- Query:
  ```spl
  index=wineventlog EventCode=4740
  ```

---

## 🚀 Future Plans
- Implement **Splunk Enterprise Security (ES)** for advanced correlation.
- Automate responses using **SOAR**.
- Integrate with **MITRE ATT&CK framework**.

---

## Conclusion

**This project provides hands-on experience in SIEM monitoring, log collection, and security analysis. This lab is a work in progress but I wanted to showcase my setup so far!**
