# VM Build & Configuration Guide

This guide walks through building and configuring every VM in the Segmentation Lab to match the network plan exactly. Follow in order: pfSense → Windows Server → Windows Client → ELK → IDS → Scanner → DMZ servers → Kali.

---

## 1. pfSense Firewall (Core Router & Segmentation)

### 1.1 Create pfSense VM (VirtualBox)

1. **New VM**
   - Name: `pfSense`
   - Type: **BSD**
   - Version: **FreeBSD (64-bit)**
   - Memory: **4096 MB** (min 2048 MB)
   - CPUs: **2+**
   - Disk: **20 GB**, VDI, dynamically allocated

2. **Network Adapters** (very important)

   - **Adapter 1 (WAN)**
     - Attached to: `NAT Network` → `WAN`
     - Promiscuous Mode: **Allow All**
     - Cable connected: ✅

   - **Adapter 2 (LAN)**
     - Attached to: `Internal Network` → `int-lan`
     - Promiscuous Mode: **Allow All**

   - **Adapter 3 (DMZ / OPT1)**
     - Attached to: `Internal Network` → `int-dmz`
     - Promiscuous Mode: **Allow All**

   - **Adapter 4 (Attacker / OPT2)** *(optional)*
     - Attached to: `Internal Network` → `int-attacker`
     - Promiscuous Mode: **Allow All**

> All internal network names must exactly match other VMs’ adapter settings:
> `int-lan`, `int-dmz`, `int-attacker`, and NAT network `WAN`.

### 1.2 Install pfSense

1. Attach the pfSense ISO to the VM.
2. Boot and choose **Install pfSense**.
3. Accept defaults (keyboard, auto UFS partition).
4. Let the installer detect NICs:
   - `em0` → WAN  
   - `em1` → LAN  
   - `em2` → DMZ (OPT1)  
   - `em3` → Attacker (OPT2, optional)

5. Configure interfaces:

   - **WAN (em0)**  
     - DHCP (recommended) or static:
       - IP: `10.0.0.2`
       - Mask: `/24`
       - Gateway: `10.0.0.1` (host NAT)

   - **LAN (em1)**  
     - Static: `10.10.0.1/24`

   - **DMZ (em2)**  
     - Static: `10.20.0.1/24`

   - **Attacker (em3, optional)**  
     - Static: `10.30.0.1/24`

6. Reboot and remove the ISO.

### 1.3 Initial Console Setup

From pfSense console menu:

1. Change admin password (Option 3 or 8 depending on version).
2. Confirm interfaces show the correct IPs.

### 1.4 Access pfSense GUI

1. From any VM on `int-lan`:
   - Open: `https://10.10.0.1/`
   - Accept the self-signed certificate warning.
2. Login:
   - User: `admin`
   - Password: *(your password)*

### 1.5 Configure Interfaces in GUI

1. **Interfaces → Assignments**
2. Ensure:
   - LAN = `10.10.0.1/24`
   - DMZ (OPT1) = `10.20.0.1/24`
   - Attacker (OPT2) = `10.30.0.1/24`
3. Make sure **Enable Interface** is checked for DMZ/Attacker.

### 1.6 Basic Firewall Rules

**LAN (10.10.0.0/24)**

- Allow LAN → any (initially):

  - Firewall → Rules → LAN → Add
  - Action: Pass
  - Source: LAN net
  - Destination: any
  - Description: `Allow LAN → any`

**DMZ (10.20.0.0/24)**

1. Block DMZ → LAN:

   - Action: Block
   - Source: DMZ net
   - Destination: LAN net
   - Place this rule **at the top** of DMZ rules.

2. Allow DMZ → WAN (HTTP/HTTPS):

   - Action: Pass
   - Protocol: TCP
   - Source: DMZ net
   - Destination: any
   - Dest ports: 80, 443

**Attacker (10.30.0.0/24)**

- Default: no rules (isolated).
- When testing, add specific rules like:
  - Allow `10.30.0.10` → `10.20.0.50` (Metasploitable) for pentesting.

### 1.7 NAT Configuration

- Firewall → NAT → Outbound → **Automatic Outbound NAT**
- This allows LAN and DMZ (and Attacker if allowed) to access internet via WAN.

---

## 2. Windows Server (AD/DNS/DHCP) — 10.10.0.10

### 2.1 VM Specs

- Name: `win-server-ad`
- OS: Windows Server 2022/2025
- vCPU: 2–4
- RAM: 8–12 GB
- Disk: 80 GB
- NIC: `int-lan`

### 2.2 Set Static IP

Inside Windows:

- IP address: `10.10.0.10`
- Subnet mask: `255.255.255.0`
- Default gateway: `10.10.0.1`
- Preferred DNS: `10.10.0.10` (self)

Verify: powershell

ipconfig /all
ping 10.10.0.1

### 2.3 Install Roles

Open Server Manager → Add roles and features.

Add roles:

Active Directory Domain Services

DNS Server

DHCP Server (optional)

Web Server (IIS)

Proceed with defaults and install.

### 2.4 Promote to Domain Controller

In Server Manager, click the AD DS notification → Promote this server to a domain controller.

Select Add a new forest:

Root domain: corp.local (or your choice)

Set DSRM password.

Accept defaults and complete wizard.

Reboot and log in as CORP\Administrator.

### 2.5 DNS Forwarders

Open DNS Manager.

Right-click your domain → Properties → Forwarders tab.

Add 8.8.8.8 as forwarder.

Test:

ping google.com
nslookup corp.local

### 2.6 DHCP Scope (Optional if you want Windows doing DHCP)

Open DHCP Manager.

Right-click IPv4 → New Scope.

Example scope:

Name: corp-scope

Start IP: 10.10.0.100

End IP: 10.10.0.200

Subnet mask: 255.255.255.0

Router: 10.10.0.1

DNS server: 10.10.0.10

Authorize and activate scope.

If using this, disable pfSense DHCP on LAN to avoid conflicts.

---

## 3. Windows Client — 10.10.0.20
### 3.1 VM Specs

OS: Windows 11 Enterprise

vCPU: 2

RAM: 4 GB

NIC: int-lan

### 3.2 IP Configuration

IP: 10.10.0.20

Subnet: 255.255.255.0

Gateway: 10.10.0.1

DNS: 10.10.0.10

### 3.3 Join Domain

Right-click Start → System → About → Domain or workgroup.

Click Change and select Domain.

Enter: corp.local (or your AD domain).

Use domain admin creds.

Reboot and log in as CORP\Username.

---

## 4. ELK SIEM VM — 10.10.0.30
### 4.1 VM Specs

OS: Ubuntu Server LTS

vCPU: 4

RAM: 8 GB

Disk: 100 GB

NIC: int-lan

IP: 10.10.0.30

Gateway: 10.10.0.1

DNS: 10.10.0.10

### 4.2 Install Elasticsearch, Logstash, Kibana
sudo apt update
sudo apt install elasticsearch logstash kibana -y


Enable and start:

sudo systemctl enable --now elasticsearch
sudo systemctl enable --now logstash
sudo systemctl enable --now kibana


Test:

curl http://localhost:9200


Browse Kibana from any LAN host:

http://10.10.0.30:5601


(You may need to edit kibana.yml to listen on 0.0.0.0.)

---

## 5. IDS (Suricata) VM — 10.10.0.40 (Optional)
### 5.1 VM Specs

OS: Ubuntu Server

vCPU: 2–4

RAM: 4–8 GB

IP: 10.10.0.40

Gateway: 10.10.0.1

NICs: at least one NIC connected to a mirrored/monitor port (or the segment you want to inspect).

### 5.2 Install Suricata
sudo apt update
sudo apt install suricata -y


Configure Suricata to monitor the appropriate interface and output to /var/log/suricata/eve.json.
Later, Logstash will parse and ship these events to Elasticsearch.

---

## 6. Scanner VM — Nessus/OpenVAS (10.10.0.60)
### 6.1 VM Specs

OS: Ubuntu Server or dedicated Nessus appliance

IP: 10.10.0.60

Gateway: 10.10.0.1

Install Nessus/OpenVAS following vendor instructions and target:

Metasploitable: 10.20.0.50

Other DMZ/LAN assets as required.

---

## 7. Ubuntu Web / osTicket (DMZ) — 10.20.0.10
### 7.1 VM Specs

OS: Ubuntu Server

NIC: int-dmz

IP: 10.20.0.10

Gateway: 10.20.0.1

DNS: 10.10.0.10 (for internal resolution)

### 7.2 Install Apache + PHP
sudo apt update
sudo apt install apache2 php php-mysql php-imap php-xml php-json -y

### 7.3 Deploy osTicket

Download osTicket and extract into:

sudo rm -rf /var/www/html/*
sudo cp -r /path/to/osticket/* /var/www/html/


Set permissions as required by osTicket documentation.

Access from LAN via http://10.20.0.10/.

---

## 8. Metasploitable2 — 10.20.0.50

Import Metasploitable2 VM.

Attach NIC to int-dmz.

Configure static IP (if not using DHCP):

IP: 10.20.0.50

Gateway: 10.20.0.1

DNS: 10.10.0.10 or 8.8.8.8

No further hardening—this is intentionally vulnerable.

---

## 9. Kali Attacker VM — 10.30.0.10
### 9.1 VM Specs

OS: Kali Linux

NIC: int-attacker

IP: 10.30.0.10

Gateway: 10.30.0.1

DNS: 8.8.8.8 or 10.10.0.10

### 9.2 Install Tools
sudo apt update
sudo apt install -y nmap hydra nikto enum4linux metasploit-framework

---

## 10. Quick End-to-End Validation

From Windows client: ping 10.10.0.1, 10.10.0.10, 10.10.0.30.

From ELK: ping 10.10.0.1 and confirm pfSense reaches internet.

From Kali:

ping 10.20.0.10 (should work)

ping 10.10.0.10 (should fail if rules are correct)

From Nessus: scan 10.20.0.50 and review results.

In Kibana: confirm log indices and dashboards show new events.
