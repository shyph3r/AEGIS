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

### Section 1 — Lab Infrastructure
### Section 2 — Linux Fundamentals
### Section 3 — Networking
### Section 4 — Docker Fundamentals
### Section 5 — Deploying Target with Docker
### Section 6 — Burp Suite
### Section 7 — Review
### Section 8 — Network Reconnaissance with Nmap
### Section 9 — HTTP Enumeration
### Section 10 — OWASP Vulnerability Testing
### Section 11 — Nuclei Vulnerability Scanning & Custom YAML Templates
### Section 12 — Web Fuzzing & Endpoint Discovery with ffuf
### Section 13 — Vulnerability Assessment & CVSS Fundamentals
### Section 14 — Security Findings, Reporting & Remediation Concepts

## Disclaimer

All security testing in this project is performed only against systems in my own isolated lab environment.
