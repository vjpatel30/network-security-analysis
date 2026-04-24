# Network Security Analysis: UDP Port Scanning and SSH Brute Force

## Overview
This project demonstrates two common network attack techniques in a controlled lab environment: UDP port scanning and SSH brute-force attacks. The goal was to understand how these attacks behave on the network, how they can be identified, and what defensive actions can reduce risk.

## Objective
- Perform a UDP port scan using Nmap
- Capture and analyze traffic in Wireshark
- Simulate an SSH brute-force attack using Hydra
- Review the security implications and identify mitigations

## Tools Used
- Nmap
- Wireshark
- Hydra
- Kali Linux

## Environment
The lab was completed in a virtualized environment using test systems on a private network.

## Steps Performed
1. Ran a full UDP scan against the target using Nmap
2. Captured UDP traffic in Wireshark to review packet behavior
3. Identified the behavior of open and closed UDP ports
4. Verified SSH exposure on port 22
5. Simulated a brute-force attack against SSH using Hydra with test credentials
6. Documented the attack behavior and defensive recommendations

## Key Findings
- UDP scanning can be difficult to interpret because open UDP ports may not respond
- Closed UDP ports often return ICMP port unreachable messages
- Weak SSH credentials can be discovered quickly using brute-force techniques
- Network traffic analysis provides useful visibility into both reconnaissance and authentication attacks

## Security Impact
This project highlights how exposed services and weak passwords can create security risk. It also shows why monitoring, strong credential policies, and service hardening are important in real environments.

## Mitigations / Recommendations
- Disable unused UDP services
- Use firewalls and monitor unusual UDP traffic
- Enforce strong password policies
- Enable MFA where possible
- Prefer SSH key-based authentication over password logins
- Use account lockout or rate limiting to reduce brute-force risk

## Skills Demonstrated
- Network scanning
- Packet analysis
- Security investigation
- Threat identification
- Defensive thinking
