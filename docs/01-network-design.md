# Network Design & Architecture

This document describes the complete network design for the Segmentation & Monitoring Lab, including networks, IP addressing, pfSense routing, and how each VM fits into the architecture.

The design mirrors a small enterprise with:

- A routed core firewall (pfSense)  
- Segmented LAN, DMZ, and Attacker networks  
- A shared WAN / Management network (10.0.0.0/24)  
- Centralized monitoring (ELK / IDS / Scanner)  
- Windows domain services

---

# 1. High-Level Network Overview

The lab uses **four primary network segments** plus a NAT/WAN network from the hypervisor:

| Segment             | Internal Name | Subnet        | Gateway / Core |
|---------------------|--------------|---------------|----------------|
| **Management / WAN** | `WAN` | **10.0.0.0/24** | 10.0.0.1 (Host NAT) / 10.0.0.2 (pfSense WAN) |
| **LAN (Internal AD)** | `int-lan` | **10.10.0.0/24** | 10.10.0.1 (pfSense LAN) |
| **DMZ (Public Services)** | `int-dmz` | **10.20.0.0/24** | 10.20.0.1 (pfSense DMZ) |
| **Attacker Network** | `int-attacker` | **10.30.0.0/24** | 10.30.0.1 (pfSense Attacker) |

> **Note:** In this lab, the **Management / Host Network and WAN NAT share the subnet 10.0.0.0/24**.  
> - The host OS typically owns **10.0.0.1**  
> - pfSense WAN uses **10.0.0.2**  
> - All other VMs receive internet via NAT through pfSense

---

# 2. Network Design Diagrams

## 2.1 Logical Network Design Diagram

<img width="1024" height="1536" alt="Pfsense Diagram" src="https://github.com/user-attachments/assets/744c55ae-08a7-42a7-a239-67af3de3bfda" />


## 2.2 Detailed Topology Diagram

<img width="1024" height="1536" alt="Pfsense Topology" src="https://github.com/user-attachments/assets/7c6577d8-c6f9-45fc-81a3-14ec7e2d37ed" />


# 3. Static IP & Role Mapping

The following IPs are considered authoritative for this lab:

IP Address	Hostname / Role	Network
10.0.0.1	Host NAT Gateway (VirtualBox)	Management/WAN
10.0.0.2	pfSense WAN	Management/WAN
10.10.0.1	pfSense LAN	LAN
10.10.0.10	Windows Server (AD/DNS/DHCP)	LAN
10.10.0.20	Windows Client	LAN
10.10.0.30	ELK / SIEM	LAN
10.10.0.40	IDS (Suricata VM, optional)	LAN
10.10.0.60	Nessus/OpenVAS Scanner	LAN
10.20.0.1	pfSense DMZ	DMZ
10.20.0.10	Ubuntu Web / osTicket	DMZ
10.20.0.50	Metasploitable2	DMZ
10.30.0.1	pfSense Attacker	Attacker
10.30.0.10	Kali Attacker	Attacker

---

# 4. pfSense Interface Mapping


## 4.1 VirtualBox NIC Mapping

In VirtualBox, the pfSense VM uses four network adapters:

Adapter	Attached to	Internal Name	Purpose
1	NAT Network → WAN	WAN	WAN / Management
2	Internal Network	int-lan	LAN
3	Internal Network	int-dmz	DMZ
4	Internal Network	int-attacker	Attacker (optional)
4.2 pfSense Interface to Subnet Map

Inside pfSense, interfaces are typically:

pfSense Interface	Network	IP / Mask	Notes
em0 (WAN)	10.0.0.0/24	10.0.0.2/24	Gateway 10.0.0.1
em1 (LAN)	10.10.0.0/24	10.10.0.1/24	LAN default gateway
em2 (DMZ / OPT1)	10.20.0.0/24	10.20.0.1/24	DMZ gateway
em3 (Attacker / OPT2)	10.30.0.0/24	10.30.0.1/24	Optional attacker segment gateway

---

# 5. Segmentation & Security Policy

## 5.1 LAN (10.10.0.0/24)

Goal: internal corp network for AD, workstations, scanners, SIEM.

Default Policy: allow outbound traffic (initially), later harden.

Key Asset Flows:

LAN → Internet (updates, browsing)

LAN → ELK (10.10.0.30) for Beats / syslog

LAN → DMZ (only for admin/testing, optional and tightly scoped)

## 5.2 DMZ (10.20.0.0/24)

Goal: host internet-exposed services & intentionally vulnerable targets.

Enforced Constraints:

DMZ → LAN blocked by default

DMZ → WAN allowed only for HTTP/HTTPS (ports 80, 443)

Hosts:

osTicket web server

Metasploitable2

## 5.3 Attacker Network (10.30.0.0/24)

Goal: emulate an external adversary.

Enforced Constraints:

Attacker → DMZ: allowed (to attack DMZ targets)

Attacker → LAN: blocked (unless temporarily allowed during exercises)

Attacker → Internet: optional (for package updates)

## 5.4 WAN / Management (10.0.0.0/24)

Goal: NAT’d network to the outside world and host management path.

Key Addresses:

10.0.0.1 → host OS (NAT gateway)

10.0.0.2 → pfSense WAN

Usage:

Provide internet to pfSense and, via NAT, to LAN/DMZ/Attacker networks.

Optional host ↔ pfSense management.

# 6. DNS, DHCP, and Name Resolution

Primary DNS: Windows Server at 10.10.0.10

Domain Name: e.g., corp.local or similar AD forest

DHCP Options:

LAN DHCP may be provided by pfSense or Windows Server (not both).

DMZ/Attacker DHCP optional; static IPs recommended for servers.

Recommended:

All internal hosts use 10.10.0.10 (Domain Controller) as their DNS.

AD DNS forwards external queries to 8.8.8.8 or other upstream resolvers.

# 7. Logging & Monitoring Placement (High-Level)

pfSense logs firewall decisions and forwards syslog to ELK (10.10.0.30).

Suricata IDS (either on pfSense or dedicated VM at 10.10.0.40) monitors key segments (WAN, DMZ, Attacker).

ELK (10.10.0.30) ingests:

Beats (Winlogbeat, Filebeat)

pfSense syslog (port 514)

Suricata eve.json

Nessus/OpenVAS (10.10.0.60) runs vulnerability scans across LAN/DMZ.

# 8. Design Goals Summary

This network design enables:

Realistic enterprise segmentation (LAN vs DMZ vs Attacker).

Clear enforcement points via pfSense.

Rich monitoring for both network and endpoint activity.

A safe yet realistic environment for practicing:

Attacks from Kali

Blue-team detection in ELK

Vulnerability scanning and hardening

Incident response and troubleshooting
