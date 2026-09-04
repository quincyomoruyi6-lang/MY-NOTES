Theoretical Network Discovery and Defensive Audit Mechanics


Scope: Educational reference notes for authorized security assessments, defensive audits, and lab environments only. Do not scan or test systems without explicit permission.

Table of Contents

•
1. Network Discovery: Layer 2 Mapping

•
2. TCP Port Scanning

•
3. Service Enumeration and Version Fingerprinting

•
4. Vulnerability Assessment

•
5. Exploitation Concepts: Shell Mechanics and Payloads

•
6. Defensive Remediation and Detection

•
Summary




1. Network Discovery: Layer 2 Mapping

Network mapping is the foundational step of a security audit. It begins at the Data Link Layer (Layer 2), where the primary objective is to identify active hardware within a local network segment.

This process relies on the Address Resolution Protocol (ARP) to resolve IP addresses to physical MAC addresses. An ARP sweep can help an auditor determine which systems are live and communicating before analyzing specific ports or services.

Core Discovery Tools

Tool
Primary mechanism
Auditing goal
arp-scan
Sends ARP packets to hosts in a local range, such as target_subnet/24.
Identifies active hosts and their MAC addresses, including systems configured to ignore ICMP ping requests.
netdiscover
Actively sends ARP requests or passively observes ARP traffic.
Maps MAC-to-IP relationships for live systems within a network segment.




Command Reference

Bash


# Execute an ARP scan on a local subnet using arp-scan
sudo arp-scan --interface=eth0 --localnet

# Perform an active ARP sweep using netdiscover
sudo netdiscover -i eth0 -r 192.168.1.0/24




Learning narrative: Identifying a host’s presence is the prerequisite for analyzing its services. You cannot evaluate the security of a door—a port—until you have first located the building—the host.




2. TCP Port Scanning

Once a host is identified, the auditor must determine which ports are open. This interaction is based on the TCP three-way handshake: SYN, SYN-ACK, and ACK.

A standard connection completes all three steps. Security auditors may instead use a SYN scan, sometimes called a stealth scan, to identify open ports without completing the full connection.

SYN Scan Process

1.
SYN request: The auditor sends a SYN packet to a specific port on the target system.

2.
Response:

•
If the port is open, the target responds with SYN-ACK.

•
If the port is closed, the target responds with RST.



3.
Final reset: The auditor does not send the final ACK. Instead, a RST packet terminates the interaction.

Command Reference

Bash


# Execute a TCP SYN scan against all TCP ports on an authorized target
sudo nmap -sS -p- 192.168.1.100




Observation: SYN scans were historically considered “stealthy” because they avoid establishing a full connection, which older logging systems might not have recorded. Modern SOC environments can usually detect these patterns. During a professional penetration test, auditors may intentionally generate detectable traffic to evaluate whether the Blue Team is monitoring effectively.




3. Service Enumeration and Version Fingerprinting

Knowing that a port is open is only the beginning. Auditors must identify the specific service and version running behind it. This additional detail distinguishes a basic scan from a professional security assessment.

Service Enumeration Methodology

Service category
Standard ports
Common audit tools
Primary information goal
Web services
80, 443
whatweb, Wappalyzer
Identify CMS versions, programming languages, frameworks, and other web technologies.
File sharing
139, 445
smbclient, Nmap
Identify service versions, such as Samba releases, to support vulnerability research.
Remote access
22
Nmap, Netcat
Perform banner grabbing to identify software and version information, such as OpenSSH releases.




Essential Nmap Flags

Flag
Function
Audit value
-sV
Version detection
Probes open ports to determine software and version information. This provides a key for searching vulnerability databases.
-sC
Default script scanning
Runs a collection of default scripts to identify common misconfigurations and exposures.
-O
OS detection
Analyzes TCP/IP stack behavior to estimate the target operating system and narrow the scope of further research.
-A
Aggressive mode
Enables OS detection, version detection, script scanning, and traceroute in one command.




Command Reference

Bash


# Run version, script, and OS enumeration against an authorized target
sudo nmap -sV -sC -O -A target_ip

# Perform web service fingerprinting
whatweb http://target_ip




Learning narrative: Version numbers act as identifiers for looking up vulnerabilities in security databases. Identifying a version does not prove that a system is vulnerable; additional research and validation are required.




4. Vulnerability Assessment

Security professionals synthesize automated scan results with manual research to validate risks and prioritize remediation.

Manual Research Practices

Search operators

Advanced search operators can help identify publicly indexed files, subdomains, or other exposure during an authorized assessment.

Plain Text


# Example search operator for identifying indexed subdomains
site:example.com -www




Replace example.com with a domain that you own or are explicitly authorized to assess.

Searchsploit

searchsploit can query a local copy of the Exploit Database for publicly documented research related to a service or version.

Bash


# Search for public research related to a specific service version
searchsploit samba 2.2.1a



Three Pillars of Vulnerability Scanning

Pillar
Description
Discovery
Scan the authorized network to identify active devices and exposed ports.
Assessment
Compare discovered services against known vulnerability information, including CVE records and other trusted databases.
Reporting
Categorize risks by severity—Critical, High, Medium, or Low—to help administrators prioritize remediation.





Learning narrative: This phase connects the “what”—the service version—to the “how”—the documented vulnerability or exploit condition. A version match is an investigative lead, not confirmation of exploitability.




5. Exploitation Concepts: Shell Mechanics and Payloads

In an authorized penetration test, gaining access may involve exploiting a vulnerability to obtain a shell, or command-line interface, on the target system. The following concepts describe payload architecture at a high level.

Reverse Shell vs. Bind Shell

Metric
Reverse shell
Bind shell
Connection direction
The target connects back to the auditor’s listener.
The auditor connects to a port opened by the target.
Firewall implications
May be more likely to pass egress controls because it resembles outbound traffic; modern defenses can still detect it.
May be blocked when inbound traffic is restricted.
Common use case
Useful when the auditor can receive an outbound connection but cannot directly connect to the target.
Useful when the target can expose a reachable listening port.




Payload Architecture

Staged payloads

A staged payload is delivered in multiple parts. A small stager is sent first; it then retrieves the larger stage from a designated location. This architecture can be useful when the initial exploit has limited space for a payload.

Metasploit staged payloads commonly use a forward slash, for example:

Plain Text


windows/shell/reverse_tcp



Non-staged payloads

A non-staged payload includes the complete exploit and shell code in a single package. It is larger, but can be simpler to deploy in some scenarios.

Metasploit non-staged payloads commonly use an underscore, for example:

Plain Text


windows/shell_reverse_tcp






6. Defensive Remediation and Detection

Hardening combines patch management, secure configuration, credential controls, monitoring, and incident response.

Defensive Best-Practices Checklist

Finding
Detection or validation approach
Recommended remediation
Outdated service headers, such as an old Apache release
Review service banners using authorized version-detection tools.
Apply current security patches and disable unnecessary version disclosure in configuration files.
Weak or default credentials
Conduct an approved credential audit using controlled testing and confirm that monitoring and alerting are functioning.
Implement MFA, enforce a strong password policy, remove default credentials, and prevent predictable passwords.
Exposed or unnecessary services
Review asset inventories, firewall rules, and network scan results.
Disable unused services, restrict access by network segment, and document required exceptions.
Repeated scanning activity
Monitor for repeated SYN patterns, unusual port sequences, and high request rates.
Tune IDS/IPS rules, investigate the source, and apply appropriate rate limiting.




Defensive Infrastructure

•
Firewalls: Block unauthorized inbound and outbound connections, including unwanted shell traffic.

•
IDS/IPS: Monitor for repeated SYN/RST patterns, port scans, brute-force attempts, and other suspicious behavior.

•
Rate limiting: Reduce the impact of noisy scanning and authentication attempts by limiting request frequency from individual sources.

•
Centralized logging: Correlate network, endpoint, authentication, and application events for investigation.

•
Patch and configuration management: Track software versions, remediation status, and approved exceptions across the environment.


Learning narrative: A dual-use mindset improves security posture. Understanding how an auditor identifies versions, tests controls, and generates observable activity helps defenders improve detection, response, and remediation.




Summary

A professional defensive audit follows a structured progression:

1.
Discover assets on the authorized network.

2.
Identify exposed ports and understand the TCP behavior involved.

3.
Enumerate services and versions to establish an evidence-based inventory.

4.
Research and validate vulnerabilities using automated findings and manual review.

5.
Assess shell and payload concepts only within an approved testing scope.

6.
Remediate findings and validate detection through monitoring, alerting, and controlled retesting.

The objective is not merely to identify weaknesses, but to produce reliable evidence that supports prioritized remediation and measurable defensive improvement.

References

This document was structured and edited from the user-provided source notes. No external sources were added or independently fact-checked.

