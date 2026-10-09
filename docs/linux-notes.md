# Linux Fundamentals

## Day 2 — Processes, Services, and Ports

### 1. Processes

A process is a running instance of a program.

I used:

```bash
ps aux
```

to view running processes and:

```bash
ps aux | grep sleep
```

to search for a specific process.

For testing, I created a temporary process:

```bash
sleep 300
```

Each process has a Process ID (PID), which can be used to identify and interact with that specific process.

For example:

```bash
kill <PID>
```

sends a termination signal to the specified process.

---

## 2. Services

A service is typically a long-running background process managed by the operating system.

On Ubuntu, I used:

```bash
systemctl --type=service --state=running
```

to view currently running services.

Examples observed in the lab included:

- `cron.service`
- `avahi-daemon.service`
- `accounts-daemon.service`

A service can be inspected with:

```bash
systemctl status <service>
```

Not every service provides a network-accessible service or listens on a network port.

---

## 3. Ports and Listening Sockets

I used:

```bash
sudo ss -tulpn
```

to inspect TCP and UDP sockets on the Ubuntu target.

Important options:

- `-t` — TCP
- `-u` — UDP
- `-l` — listening sockets
- `-p` — associated processes
- `-n` — numeric addresses and ports

I observed services listening on addresses such as:

```text
127.0.0.1:631
127.0.0.54:53
0.0.0.0:5353
```

An important distinction is:

```text
127.0.0.1:<port>  → accessible only locally

0.0.0.0:<port>    → listening on all IPv4 interfaces
```

---

## 4. Creating a Test HTTP Service

On the Ubuntu target (`192.168.219.20`), I started a temporary HTTP server:

```bash
python3 -m http.server 8000
```

This created a Python process that listened on TCP port `8000`.

I verified the listening socket with:

```bash
sudo ss -tulpn
```

From the Kali attacker (`192.168.219.10`), I accessed:

```text
http://192.168.219.20:8000
```

The request successfully reached the HTTP server running on Ubuntu.

After terminating the Python server with `Ctrl+C`, port `8000` was no longer listening.

---

## 5. Key Concept

The experiment demonstrated the relationship:

```text
Program
   ↓
Process
   ↓
Network Service
   ↓
Listening Socket / Port
   ↓
Network
   ↓
Remote Client
```

In my lab:

```text
Kali Attacker
192.168.219.10
        │
        │ HTTP over TCP
        ▼
Ubuntu Target
192.168.219.20:8000
        │
        ▼
Python HTTP Server
```

This is important for later reconnaissance and port scanning: tools such as Nmap can remotely identify network services exposed by a target.