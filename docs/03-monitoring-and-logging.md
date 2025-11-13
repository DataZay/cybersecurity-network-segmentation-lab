# Monitoring & Logging Architecture

This document describes how logs and telemetry flow from pfSense, servers, workstations, and security tools into the ELK SIEM (Elasticsearch, Logstash, Kibana).

Goal: **full visibility** into LAN, DMZ, and Attacker activities.

---

## 1. Components Overview

The lab uses the following for monitoring:

- **Elasticsearch** – central log store (10.10.0.30)
- **Logstash** – log collection & parsing
- **Kibana** – visualization and dashboards
- **Winlogbeat** – Windows logs (Server + Client)
- **Sysmon** – detailed Windows telemetry
- **Filebeat** – Linux + Apache logs (DMZ, ELK, IDS, Scanner)
- **Suricata** – IDS (either on pfSense or a dedicated VM at 10.10.0.40)
- **pfSense syslog** – firewall / router logs
- **Nessus/OpenVAS** – optional vulnerability scan reports

---

## 2. Logging & Telemetry Diagram

<img width="1024" height="1536" alt="Logging and Telemetry Pfsense" src="https://github.com/user-attachments/assets/5c3d9fe8-9bb4-4d86-b150-57e6e80378fa" />


This diagram illustrates the centralized log ingestion pipeline:

- **Windows Server & Windows Client**
  - Winlogbeat → Logstash

- **Linux Servers (osTicket, IDS, Scanner, etc.)**
  - Filebeat → Logstash

- **pfSense Firewall**
  - Syslog output → Logstash

- **Suricata IDS**
  - eve.json → Logstash

- **Logstash → Elasticsearch**
  - Data parsing, enrichment, indexing

- **Elasticsearch → Kibana**
  - Search, dashboards, SIEM visualizations
 
---

## 3. Windows Monitoring (Server & Client)
### 3.1 Install Sysmon

On each Windows box:

Download Sysmon binaries.

Install with a config (e.g. SwiftOnSecurity or a custom one):

Sysmon64.exe -accepteula -i sysmonconfig.xml


Sysmon generates rich events for:

Process creation (new processes, command lines)

Network connections

File creation/time changes

Registry operations

Driver loads

### 3.2 Install Winlogbeat

Download Winlogbeat MSI from Elastic.

Edit winlogbeat.yml to:

Send to Logstash at 10.10.0.30:5044

Include Application, System, Security, and Sysmon channels.

Example snippet:

output.logstash:
  hosts: ["10.10.0.30:5044"]


Install and start the service:

winlogbeat setup
Start-Service winlogbeat

### 3.3 Windows → SIEM Flow
[Windows Event Logs / Sysmon]
           ↓
        Winlogbeat
           ↓
        Logstash
           ↓
     Elasticsearch
           ↓
        Kibana

---

## 4. Linux & Web Server Monitoring
### 4.1 Filebeat on Linux hosts

Install Filebeat on:

ELK server itself (to monitor system logs)

osTicket web server (10.20.0.10)

IDS VM (10.10.0.40)

Scanner VM (10.10.0.60, optional)

Typical install:

sudo apt update
sudo apt install filebeat -y


In filebeat.yml, enable:

system module (syslog, auth.log)

apache module for osTicket VM

Configure output:

output.logstash:
  hosts: ["10.10.0.30:5044"]


Enable modules and start:

sudo filebeat modules enable system apache
sudo systemctl enable --now filebeat

## 4.2 Linux → SIEM Flow
[Syslog / Auth.log / Apache]
            ↓
          Filebeat
            ↓
          Logstash
            ↓
       Elasticsearch
            ↓
          Kibana

---

## 5. pfSense Firewall Logging
### 5.1 Syslog Output

In pfSense:

Status → System Logs → Settings

Configure a Remote log server: 10.10.0.30

Port: e.g. 514 (UDP)

Facility: local0 or similar

Log types: firewall, system, DHCP, DNS, etc.

### 5.2 Logstash Syslog Input

On ELK VM, a Logstash pipeline (conceptually) listens for pfSense logs:

input {
  udp {
    port => 514
    type => "pfsense"
  }
}

filter {
  # Grok/parse pfSense format (pattern not shown here for brevity)
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "pfsense-%{+YYYY.MM.dd}"
  }
}

### 5.3 What pfSense Logs Show

Allowed / Blocked traffic between LAN/DMZ/Attacker/WAN

NAT translations

DHCP leases

DNS resolver events (if using pfSense DNS)

These allow you to verify segmentation and detect cross-segment access attempts.

---

## 6. Suricata IDS Logging
### 6.1 Suricata Output

Suricata writes events to JSON:

/var/log/suricata/eve.json


Events include:

Port scans (e.g., nmap scans from Kali)

Exploit attempts against Metasploitable or osTicket

Brute-force login behavior

Suspicious HTTP requests

Malware command & control patterns

### 6.2 Logstash File Input (Conceptual)

Example conceptual configuration:

input {
  file {
    path => "/var/log/suricata/eve.json"
    codec => "json"
    type => "suricata"
  }
}

filter {
  # Tag severity, classify signatures, etc.
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "suricata-%{+YYYY.MM.dd}"
  }
}

---

## 7. Kibana Dashboards & Use Cases

Once logs are flowing, create dashboards for:

### 7.1 Firewall & Network

Firewall Denies Over Time

Visualize blocked DMZ → LAN or Attacker → LAN traffic.

Top Talkers

Source/destination IP volume.

Suricata Alerts

Severity heatmaps

Top signatures

Timeline of attack campaigns

### 7.2 Endpoint Analytics

Windows Logon Dashboard

Failed logins (Event ID 4625)

Successful logins by host/user

Sysmon Process Dashboard

New or unusual processes

Command-line analysis

Linux Authentication Dashboard

Failed sudo attempts

SSH activity on DMZ servers

### 7.3 Web & DMZ Visibility

Apache Access Dashboard

Requests over time

4xx and 5xx errors

Suspicious paths or user agents

Attack Traffic

Nmap, Nikto, and brute-force patterns correlated with Suricata and Apache logs.

---

## 8. Validation Steps

To validate your logging pipeline:

From Kali, run:

nmap -sS -Pn 10.20.0.50   # Metasploitable
curl -I http://10.20.0.10 # osTicket


From Metasploitable:

ping 10.20.0.1  # pfSense DMZ


From Nessus/OpenVAS:

Run a scan against 10.20.0.50.

In Kibana:

Confirm new indices for:

winlogbeat-*

filebeat-*

pfsense-*

suricata-*

Confirm recent events corresponding to your test activity.
