# AEGIS

AEGIS is a cybersecurity homelab designed to study the complete security lifecycle:

Attack → Telemetry → Detection → Investigation → Response → Remediation → Retest → Measurement

## Current Lab Architecture

| System | Role | Lab IP |
|---|---|---|
| Kali Linux | Attacker | 192.168.219.10 |
| Ubuntu | Target | 192.168.219.20 |

### Networks

- VMware NAT: Internet access
- VMware Host-only (VMnet1): isolated AEGIS lab traffic
- AEGIS subnet: `192.168.219.0/24`

## Project Status

### Day 1 — Lab Infrastructure
### Day 2 - Linux fundamentals
### Day 3 - Networking

## Disclaimer

All security testing in this project is performed only against systems in my own isolated lab environment.
