# Cybersecurity Network Segmentation Lab — Overview

This document provides a high-level overview of the entire segmented enterprise lab built using pfSense, Windows Server domain services, Linux servers, ELK SIEM, Suricata IDS, and an attacker VLAN for controlled penetration testing.

The lab is designed to simulate real-world enterprise network segmentation and provide hands-on experience with:
- Firewalling
- Network access control
- DMZ isolation
- Blue-team detection engineering
- Red-team attack simulation
- Log management & SIEM visibility
- Enterprise host configuration

---

# 1. Lab Objectives

## 1.1 Security Architecture Skills
This lab develops core defensive engineering capabilities:
- Implementing segmented network zones  
- Firewall rule design  
- Zero-trust segmentation  
- DMZ isolation principles  
- Logging & SIEM pipeline construction  
- IDS deployment & tuning  

---

# 2. Network Overview

## 2.1 Segmented Networks

| Network | Subnet | Purpose |
|--------|--------|----------|
| **Management/Host** | 10.0.0.0/24 | Host-only access for pfSense admin |
| **LAN** | 10.10.0.0/24 | Domain services, user workstations, SIEM |
| **DMZ** | 10.20.0.0/24 | Public-facing, semi-trusted services |
| **Attacker** | 10.30.0.0/24 | Offensive tools (Kali) |

---

# 3. Core Lab Components

### ✔ pfSense Firewall  
Manages all segmentation, gateway routing, NAT, and access control.

### ✔ Windows Server (AD DS, DNS, DHCP)  
Used for identity, authentication, and enterprise operations.

### ✔ Windows Client  
Domain workstation used for testing workstation logging and policy effects.

### ✔ ELK SIEM Server  
Full log aggregation, parsing, visualization & detection.

### ✔ Suricata IDS  
Network intrusion detection on LAN/DMZ/Attacker zones.

### ✔ DMZ Servers  
- osTicket (web helpdesk)  
- Metasploitable2 (vulnerable machine for red-team testing)

### ✔ Kali Linux  
Attacker role for penetration testing and detection validation.

---

# 4. Documentation Structure

This lab documentation is divided into six sections:

- **01-network-design.md** → topology, segmentation, diagrams  
- **02-vm-build-guide.md** → build instructions for every VM  
- **03-monitoring-and-logging.md** → logging pipeline architecture  
- **04-attack-scenarios-testing.md** → red-team test scenarios  
- **05-future-improvements.md** → roadmap for enhancements  
