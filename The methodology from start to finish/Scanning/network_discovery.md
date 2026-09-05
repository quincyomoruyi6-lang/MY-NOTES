# Network Discovery & Defensive Audit Mechanics

Reference notes covering the core theory and commands behind network discovery, port scanning, service enumeration, vulnerability assessment, exploitation shell mechanics, and defensive remediation.

## Table of Contents

- [1. Network Discovery — Layer 2 Mapping](#1-network-discovery--layer-2-mapping)
- [2. TCP Port Scanning](#2-tcp-port-scanning)
- [3. Service Enumeration & Version Fingerprinting](#3-service-enumeration--version-fingerprinting)
- [4. Vulnerability Assessment](#4-vulnerability-assessment)
- [5. Exploitation — Shell Mechanics & Payloads](#5-exploitation--shell-mechanics--payloads)
- [6. Defensive Remediation & Detection](#6-defensive-remediation--detection)

---

## 1. Network Discovery — Layer 2 Mapping

Network mapping is the foundational step of any security audit. It begins at the Data Link Layer (Layer 2), where the goal is identifying active hardware on a local network segment. This relies on ARP (Address Resolution Protocol) to resolve IP addresses to physical MAC addresses — an ARP sweep determines which systems are live before any port or service analysis begins.

### Core Discovery Tools

| Tool | Mechanism & Goal |
|---|---|
| `arp-scan` | Sends ARP packets to every host in the local range. Identifies active hosts and MAC addresses even if they ignore ICMP pings. |
| `netdiscover` | Actively sends ARP requests, or passively sniffs ARP traffic, to map the MAC-to-IP relationship for all live systems in a segment. |

### Commands

```bash
# ARP scan on a local subnet
sudo arp-scan --interface=eth0 --localnet

# Active ARP sweep
sudo netdiscover -i eth0 -r 192.168.1.0/24
```

> **Learning Narrative:** Identifying a host's presence is the essential prerequisite for analyzing its services. You can't evaluate the security of a door (a port) until you've located the building (the host).

---

## 2. TCP Port Scanning

Once a host is identified, the next step is determining which ports are open. This relies on the TCP Three-Way Handshake (SYN, SYN-ACK, ACK). A standard connection completes all three steps — a **stealth scan** deliberately doesn't.

### The SYN (Stealth) Scan Process

1. **SYN Request** — the auditor sends a SYN packet to a specific port.
2. **Response** — open ports reply with SYN-ACK; closed ports reply with RST.
3. **Final Reset** — instead of completing the handshake with an ACK, the auditor sends a RST to terminate immediately.

```bash
# TCP SYN (Stealth) scan
sudo nmap -sS -p- 192.168.1.100
```

> **Observation:** Historically "stealthy" because it avoided triggering basic logging. In a modern SOC environment, this pattern is easily detected — during a professional test, auditors are often *intentionally* loud, specifically to test whether the Blue Team is actively monitoring.

---

## 3. Service Enumeration & Version Fingerprinting

Beyond knowing a port is open, auditors need the specific service and version — this is what separates a generic scan from a professional audit.

### Enumeration by Service Category

| Service | Ports | Key Tools | Goal |
|---|---|---|---|
| Web | 80, 443 | `whatweb`, `wappalyzer` | CMS/language versions (e.g. Drupal 8, PHP 7.3.7). `whatweb` (CLI) gives deeper versioning than the Wappalyzer extension. |
| File Sharing | 139, 445 | `smbclient`, `nmap` | Specific versions (e.g. Samba 2.2.1a) for known-vulnerability matching. |
| Remote Access | 22 | `nmap`, `netcat` | Banner grabbing for exact versions (e.g. OpenSSH 2.9p2). |

### Essential Nmap Flags

| Flag | Function | Why it matters |
|---|---|---|
| `-sV` | Version detection | Gives the exact "primary key" needed to search for known exploits |
| `-sC` | Default script scan | Automates discovery of common misconfigurations — low-hanging fruit |
| `-O` | OS detection | Many exploits are architecture-specific; narrows exploit selection |
| `-A` | Aggressive mode | OS detection + version detection + script scanning + traceroute, in one pass |

```bash
# Full version, script, and OS enumeration
sudo nmap -sV -sC -O -A target_ip

# Deep web service fingerprinting
whatweb http://target_ip
```

> **Learning Narrative:** Version numbers act as primary keys for looking up vulnerabilities. These mechanics identify open doors — validating the actual risk still requires further research.

---

## 4. Vulnerability Assessment

Combining automated scanning with manual research to validate real risk.

### Manual Research

```bash
# Google dork example — unique subdomains
site:tesla.com -www

# Searchsploit — query the local Exploit-DB copy
searchsploit samba 2.2.1a
```

### The Three Pillars of Vulnerability Scanning (Nessus)

1. **Discovery** — scan the network for every active device and open port
2. **Assessment** — compare discovered services against known CVEs (the automated equivalent of manual searchsploit work)
3. **Reporting** — categorize risk by severity (Critical/High/Medium/Low) to prioritize patching

> **Learning Narrative:** This phase connects the "what" (the service version) to the "how" (the exploit) — identifying OpenSSL 0.9.6b, for instance, leads directly to researching the OpenLuck exploit.

---

## 5. Exploitation — Shell Mechanics & Payloads

Gaining access means leveraging a vulnerability to receive a shell — a command-line interface — on the target.

### Reverse Shell vs. Bind Shell

| | Reverse Shell | Bind Shell |
|---|---|---|
| Connection direction | Target connects back to the auditor | Auditor connects to a port opened on the target |
| Firewall implications | Usually bypasses firewalls (looks like outgoing traffic) | Often blocked (incoming traffic is restricted) |
| Common use | ~95% of assessments — auditor listens for the connection | Used when the auditor can't receive incoming traffic due to NAT |

### Payload Architecture

- **Staged** — sent in two parts: a small "stager" pulls a larger "stage" from the listener afterward. Critical for memory-constrained exploits. Metasploit syntax uses a forward slash (`windows/shell/reverse_tcp`).
- **Non-staged** — the entire exploit and shellcode sent as one package. Larger, but often more stable. Metasploit syntax uses an underscore (`windows/shell_reverse_tcp`).

---

## 6. Defensive Remediation & Detection

### Defensive Checklist

**Finding: Outdated service headers** (e.g. Apache 1.3.20)
- Detection: banner grabbing via `nmap -sV` or `whatweb`
- Remediation: apply latest patches, disable version disclosure in configuration

**Finding: Weak or default credentials**
- Detection: brute-force audit to test whether Blue Team alerts trigger

```bash
# Example Hydra SSH brute-force check
hydra -l root -P passwords.txt ssh://target_ip
```

- Remediation: enforce MFA and a robust password policy (avoid predictable patterns like "Fall2019")

### Core Defensive Infrastructure

- **Firewalls** — primary gatekeepers blocking unauthorized bind/reverse shell connections
- **IDS/IPS** — monitor for repeated SYN-RST patterns indicating scanning activity
- **Rate limiting** — prevents loud scanning and brute-force attempts by capping request frequency per source

> **Learning Narrative:** A dual-use mindset — thinking like an auditor, using precise versioning and intentional "loud" detection testing — is what fundamentally improves organizational defense.