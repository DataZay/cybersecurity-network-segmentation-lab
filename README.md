# Cybersecurity Network Segmentation Lab (pfSense + DMZ + ELK + SIEM)

This repository documents a full enterprise-style cybersecurity lab using network segmentation, pfSense firewalling, DMZ isolation, threat monitoring, and SIEM-based visibility.

## 🚀 Lab Components

- pfSense firewall (WAN, LAN, DMZ, Attacker)
- Windows Server (AD DS, DNS, DHCP)
- Windows 11 Enterprise client
- Ubuntu Server (DMZ services: osTicket, Metasploitable2, custom apps)
- Linux server running ELK stack (Elasticsearch, Logstash, Kibana)
- Suricata IDS/IPS
- Kali attacker network (10.30.0.0/24)

## 🔥 Features & Learning Objectives

- Network segmentation & enterprise firewalling
- DMZ security architecture
- Domain controller deployment
- Endpoint monitoring (Winlogbeat + Filebeat + Sysmon)
- Network monitoring (Suricata → ELK)
- SOC visibility workflows
- Attack simulations (port scanning, brute force, SMB enumeration)

## 📁 Documentation

All detailed documentation lives in the `/docs` folder.

## 📊 Diagrams

<img width="1024" height="1536" alt="Pfsense Diagram" src="https://github.com/user-attachments/assets/4c0ac13c-47fe-46e3-8aee-d824f8d25531" />

