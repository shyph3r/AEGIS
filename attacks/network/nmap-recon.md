# Network Reconnaissance — OWASP Juice Shop

**Project:** AEGIS v1.0  
**Phase:** Week 2 — Day 8  
**Tool:** Nmap 7.99  
**Target IP:** 192.168.219.20  
**Environment:** Authorized VMware Cybersecurity Lab

## 1. Objective

The objective of this assessment was to identify open TCP ports and exposed network services on the Ubuntu target machine hosting OWASP Juice Shop.

## 2. Lab Environment

- **Attacker:** Kali Linux VM
- **Target:** Ubuntu Linux VM
- **Virtualization:** VMware
- **Target Application:** OWASP Juice Shop
- **Container Platform:** Docker

All tests were performed within an authorized local laboratory environment.

## 3. Methodology

A TCP Connect Scan was performed using Nmap.

**Command:**

`nmap -sT 192.168.219.20`

The scan used Nmap's default selection of common TCP ports.

## 4. Scan Results

The target was reachable from Kali Linux.

| Port | State | Service Label |
|---|---|---|
| 3000/tcp | Open | ppp |

Additional observations:

- The target responded to Nmap's host discovery.
- 999 scanned TCP ports were reported as closed.
- TCP port 3000 was the only open port identified in the default scan.
- The reported MAC address was associated with VMware.

**Note:** The `ppp` service label is based on Nmap's port-to-service mapping. The actual application was not identified by this initial scan.

## 5. Host-Side Verification

The following commands were executed on Ubuntu:

`sudo ss -tlnp`

`sudo docker ps`

The `ss` output showed that `docker-proxy` was listening on:

- `0.0.0.0:3000`
- `[::]:3000`

This indicates that Docker was publishing TCP port 3000 through the Ubuntu host.

The external Nmap results and local listening sockets were consistent.

## 6. Security Analysis

TCP port 3000 was reachable from the Kali Linux VM and represents an exposed network service in the laboratory environment.

An open port does not necessarily indicate a security vulnerability. Further inspection is required to identify the application, verify its configuration, and evaluate potential weaknesses.

No vulnerability was confirmed by this initial network scan.

## 7. Next Steps

- Perform Nmap service version detection using `-sV`.
- Confirm the HTTP service using curl.
- Enumerate web application endpoints using Burp Suite.
- Document potential security findings during subsequent testing.

## 8. Evidence

- `nmap-results.txt` — Original Nmap scan output.
- `nmap-service-results.txt` — Service detection output, pending.
- Ubuntu `ss -tlnp` screenshot — Local listening sockets.