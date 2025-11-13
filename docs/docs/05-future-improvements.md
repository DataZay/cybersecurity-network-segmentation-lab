# Future Improvements & Expansion Roadmap

This document outlines recommended enhancements to evolve the Segmentation & Monitoring Lab into a fully mature SOC-grade environment.

---

# 1. Network Enhancements

## 1.1 Migrate Internal Segments to VLANs
Replace VirtualBox internal nets with VLAN-tagged interfaces on pfSense:

- VLAN 10 → LAN  
- VLAN 20 → DMZ  
- VLAN 30 → Attacker  
- VLAN 40 → Security Tools  

## 1.2 Zero-Trust Segmentation
- Enforce identity-based rules
- Implement MFA for admin accounts
- Minimize east-west communication

---

# 2. SIEM & Monitoring Enhancements

## 2.1 Deploy Elastic Agent Fleet
Replaces Filebeat/Winlogbeat:

- Unified telemetry
- EDR-like capabilities
- Easier policy management

## 2.2 Add Sigma Ruleset
Convert Sigma → Elastic detection rules for:

- Privilege escalation
- Persistence mechanisms
- Lateral movement

## 2.3 Threat Intelligence Integration
Feed sources to consider:

- Abuse.ch
- OTX
- Emerging Threats
- Zeek Intel Framework

---

# 3. Automation Enhancements

## 3.1 Terraform for VM Automation
Automate building:

- pfSense
- Windows Server
- Ubuntu hosts
- DMZ servers

## 3.2 Ansible for Configuration Management
Automate:

- ELK deployment
- Beats installation
- pfSense firewall rules
- Suricata sensor provisioning

## 3.3 CI/CD Pipeline for Detection Engineering
GitHub Actions can:

- Validate Suricata signatures
- Test Logstash pipelines
- Deploy ELK dashboards automatically

---

# 4. Additional Tooling

Add:

- **Velociraptor** (DFIR agent)
- **TheHive** (Case management)
- **Cortex** (Automated enrichment)
- **OpenVAS** (Vulnerability scanning)
- **MITRE CALDERA** (Adversary emulation)

---

# 5. SOC Training Integrations

Develop:

- Incident response playbooks
- Alert triage workflows
- Threat hunting scenarios
- Purple-team exercises
