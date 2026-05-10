# Network Security Analysis: UDP Port Scanning, Traffic Analysis, and SSH Brute Force Simulation

## Overview

This project simulates attacker reconnaissance and credential attack techniques in a controlled lab environment, then analyzes the resulting network traffic from a defender's perspective. Using Nmap, Wireshark, and Hydra against a target host at 192.168.2.242, the lab covers full UDP port scanning, packet-level traffic analysis, SSH brute-force simulation, and production of an incident-style report with prioritized defensive recommendations.

---

## Objective

- Perform a full UDP port scan to identify exposed services and map attack surface
- Capture and analyze network traffic in Wireshark to understand scan and attack behavior at the packet level
- Simulate an SSH brute-force attack to demonstrate credential risk on exposed services
- Produce incident-style documentation with security impact assessment and prioritized mitigations

---

## Tools Used

- Nmap 7.94SVN (network scanning and service enumeration)
- Wireshark (packet capture and traffic analysis)
- Hydra (brute-force simulation)
- Kali Linux 2024.4
- Virtualized private lab network (VMware)

---

## Environment

All activity was conducted in an isolated VMware virtualized environment. Target host: 192.168.2.242 (MAC: 00:0C:29:C7:B2:FA). No external or production systems were targeted.

---

## Investigation Walkthrough

### Step 1 — Full UDP Port Scan (Nmap)
Executed a full UDP scan across all 65,535 ports on the lab target to identify exposed services and establish the attack surface.

**Command used:**
```bash
sudo nmap -sU -p- 192.168.2.242
```

**Results:**
- Scan completed in 553.05 seconds
- 65,534 ports shown as open|filtered (no response — expected UDP behavior)
- 1 port confirmed open: **137/udp — netbios-ns**
- Host latency: 0.00052s

**Targeted follow-up scan on specific ports:**
```bash
sudo nmap -sU -p 53,161,500,137 192.168.2.242
```

| Port | State | Service |
|---|---|---|
| 53/udp | closed | domain (DNS) |
| 137/udp | open | netbios-ns |
| 161/udp | closed | snmp |
| 500/udp | open\|filtered | isakmp |

**Defender takeaway:** NetBIOS (137) exposure allows attackers to enumerate hostnames, workgroup names, and MAC addresses without authentication. ISAKMP (500) open|filtered indicates potential VPN service exposure. Both should be restricted by firewall ACL if not required.

### Step 2 — Packet-Level Traffic Analysis (Wireshark)
Captured UDP traffic during the Nmap scan to analyze behavior at the packet level.

**Capture stats:** 506 packets total, 133 displayed (26.3%) after applying UDP filter

**Key observations:**
- Open|filtered UDP ports: no response returned from target — consistent with firewall dropping packets or service not responding
- Closed UDP ports: ICMP Port Unreachable (Type 3, Code 3) returned by target, confirming port is closed
- Traffic observed from source 192.168.2.242 including multicast and broadcast UDP
- High volume of sequential UDP probes visible — a pattern that would trigger port sweep detection rules in a production SOC environment

**SOC application:** A detection rule alerting on sequential UDP probes from a single source IP or high-volume ICMP Port Unreachable responses would catch this reconnaissance pattern. Recommended threshold: >500 ICMP unreachable responses from one source within 60 seconds.

### Step 3 — Service Enumeration
Targeted scan confirmed SSH (port 22) running and accessible on the target host, making it a candidate for credential attack simulation.

### Step 4 — SSH Brute Force Simulation (Hydra)
Simulated a credential brute-force attack against the exposed SSH service on 192.168.2.242.

**Command used:**
```bash
hydra -l <userlist> -P <wordlist> ssh://192.168.2.242
```

**Attack details:**
- 45 total attempts across multiple usernames: guest, user, root, admin, dvwa, kaxpa, sshd, administrator
- Passwords tested: root, admin, password, 123456, p@ssword
- No account lockout triggered — unlimited attempts allowed
- **Result: Valid credentials recovered**
  - **Login: administrator**
  - **Password: password**
- 1 of 1 targets successfully completed
- Attack finished: 2025-03-29 02:15:21

**Defender takeaway:** The administrator account was using "password" as its password — recovered in 45 attempts with no lockout triggered. Without rate limiting, an attacker with network access to port 22 can recover weak credentials at machine speed in minutes.

### Step 5 — Security Impact Assessment

| Risk Area | Finding | Severity |
|---|---|---|
| NetBIOS Exposure | 137/udp open — allows unauthenticated host enumeration | High |
| ISAKMP Exposure | 500/udp open\|filtered — potential VPN surface | Medium |
| SSH Credential Weakness | administrator:password recovered in 45 attempts | Critical |
| Lockout Policy | None — unlimited authentication attempts allowed | Critical |
| Network Visibility | No detection triggered during scan or brute force | High |

### Step 6 — Incident-Style Report and Mitigations

**Prioritized recommendations:**

1. **Immediately change administrator password** — "password" is in every wordlist. Enforce a minimum 16-character complex password policy across all accounts.

2. **Disable NetBIOS over TCP/IP** — Port 137 should not be exposed. Disable via network adapter settings or block at firewall. NetBIOS exposes hostname, workgroup, and MAC address to unauthenticated attackers.

3. **Enforce SSH hardening** — Disable password-based SSH authentication entirely and migrate to key-based authentication. Key-based auth eliminates the brute-force vector completely since there is no secret string to guess, making it fundamentally more secure than password + MFA for SSH.

4. **Implement account lockout and rate limiting** — Configure `MaxAuthTries 3` in sshd_config and deploy fail2ban with a threshold of 5 failed attempts triggering a 10-minute IP ban.

5. **Restrict port 22 by IP** — SSH should only be accessible from known management IPs via firewall ACL. Port 22 exposed to the full network is unnecessary attack surface.

6. **Deploy anomaly detection for UDP reconnaissance** — Alert on >500 sequential UDP probes or ICMP Port Unreachable bursts from a single source within 60 seconds.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Network Service Discovery | T1046 | Full UDP port scan across 65,535 ports on 192.168.2.242 |
| Brute Force: Password Guessing | T1110.001 | Hydra SSH brute force — 45 attempts, credentials recovered |
| Valid Accounts | T1078 | administrator:password used for successful authentication |
| Network Sniffing | T1040 | Wireshark capture of 506 packets during reconnaissance |

---

## Key Findings Summary

| Finding | Detail |
|---|---|
| Target Host | 192.168.2.242 |
| Ports Scanned | All 65,535 UDP ports |
| Scan Duration | 553.05 seconds |
| Open Port Found | 137/udp (netbios-ns) |
| Open\|Filtered Port | 500/udp (isakmp) |
| Wireshark Packets Captured | 506 total / 133 displayed |
| Brute Force Attempts | 45 |
| Credentials Recovered | administrator : password |
| Lockout Policy | None |
| MITRE Techniques | T1046, T1110.001, T1078, T1040 |
| Overall Risk Rating | Critical |

---

## Screenshots

### UDP Scan Output (Nmap)
![UDP Scan Output](udp-scan-nmap.png)

### Wireshark UDP Traffic Capture
![Wireshark UDP Capture](wireshark-udp-capture.png)

### Service Enumeration Results
![Service Enumeration Results](service-enumeration-results.png)

### SSH Brute Force Result (Hydra)
![SSH Brute Force Result](ssh-bruteforce-hydra.png)

---

## Full Presentation
[View the full project presentation](Vedant_Security_Presentation.pptx)

---

## Skills Demonstrated

- Network reconnaissance and attack surface mapping
- Full UDP port scanning and service enumeration
- Packet-level traffic analysis (Wireshark)
- Brute-force attack simulation and credential risk assessment
- MITRE ATT&CK technique mapping (offensive and defensive)
- Security impact assessment with severity ratings
- Prioritized remediation recommendations with technical depth
- Incident-style report production
