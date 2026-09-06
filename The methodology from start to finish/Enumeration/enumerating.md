# Enumeration — A Deep Dive

Enumeration is the phase between "I found something" and "I know how to use it." This document covers the core framework behind it, then walks through it across web applications, file shares, document metadata, and OSINT — the actual techniques, not just the theory.

## Table of Contents

- [1. The Core Framework](#1-the-core-framework)
- [2. Web Application Enumeration](#2-web-application-enumeration)
- [3. File Share (SMB) Enumeration](#3-file-share-smb-enumeration)
- [4. Metadata & Document-Based Enumeration](#4-metadata--document-based-enumeration)
- [5. OSINT as Enumeration's Foundation](#5-osint-as-enumerations-foundation)
- [6. The Pivot — Connecting What You Find](#6-the-pivot--connecting-what-you-find)

---

## 1. The Core Framework

Every single thing enumeration turns up — a file, an open port, a stray comment in a page's source — answers one of two questions, and nothing else really matters:

**Does this get me in? Or does this help me once I'm already in?**

> **Learning Narrative:** Enumeration without this framework becomes aimless clicking. With it, every finding has an immediate purpose — you're never just "looking around," you're always sorting what you see into one of these two buckets.

---

## 2. Web Application Enumeration

### Fingerprinting First

Before digging for hidden content, identify what's actually running.

```bash
# Full version, script, and OS enumeration
nmap -sV -sC -O -A target_ip

# Deep web-specific fingerprinting
whatweb http://target_ip
```

Browser-based alternative: the **Wappalyzer** extension gives an instant tech-stack read while browsing normally — lighter weight, good for a quick first pass.

### Directory & Content Discovery

Once the target is fingerprinted, the next question is what's actually sitting on the server that isn't linked from the visible page.

```bash
# DirBuster / Gobuster style directory brute-force
gobuster dir -u http://target_ip -w /usr/share/wordlists/dirb/common.txt -x php
```

Key settings that matter:
- **Wordlist size** — a small default list finds less than a larger one (SecLists' `common.txt` typically outperforms a tool's tiny built-in default)
- **File extension** — match the target's actual language (`.php`, `.asp`, etc.)
- **Thread count** — higher speeds up the scan, but can crash lightweight targets; lower it if the scan pauses on repeated errors

### What's Actually Worth Investigating

Not every result matters equally.

| Signal | Why it matters |
|---|---|
| **403 response** | Something real exists — you're just not currently allowed to see it |
| **401 response** | A login-protected resource exists, often an admin area |
| Paths containing `admin`, `backup`, `old`, `test`, `dev`, `config`, `api` | Some of the strongest signals in any result list |
| File extensions `.zip`, `.bak`, `.sql`, `.old`, `.git` | Often an accidental leftover — sometimes real source code or credentials |
| `robots.txt` | Ironic but genuinely useful — built to tell search engines what *not* to index, which often reveals exactly what the owner considers sensitive |

### Investigating a Result Properly

Once something interesting is found, don't just note it — open it and actually look:

- Check the **Server header** in the response — often names the exact software and version running
- Read the **actual page source**, not just the rendered page — developers leave comments, sometimes commented-out code
- Check **linked JavaScript files** — client-side code sometimes reveals API endpoints that appear nowhere else on the visible site
- Ask whether the thing you found **actually does something**, or is just a static dead page — functional beats static for what's worth pursuing further

### Intercepting Traffic — Burp Suite

Burp sits between your browser and the target as a proxy, showing every request and response, not just the final rendered page.

- Set your browser's proxy to `127.0.0.1:8080` (Burp's default listening address)
- Every request now routes through Burp first, gets logged, then forwarded to the real site
- Use this to catch background calls a page makes that never show up in the rendered UI — a common pattern is a page quietly calling something like `/api/user/profile` behind the scenes

> **Learning Narrative:** Directory busting tells you an address exists. Fingerprinting and traffic interception tell you what's actually behind it. Neither alone is enumeration — the combination is.

---

## 3. File Share (SMB) Enumeration

### Checking Access

```bash
# List available shares, attempting anonymous access
smbclient -L //target_ip/ -N
```

`-L` lists shares; `-N` attempts with no password. A returned list means anonymous access is allowed — a real finding on its own.

### The Automated Sweep

```bash
# Comprehensive automated SMB enumeration
enum4linux -a target_ip
```

One command pulls shares, usernames, OS details, and password policy in a single pass — `-a` requests everything available.

### Once Inside a Share

```bash
smbclient //target_ip/ShareName -N
```

- `ls` — list contents
- `get filename` — pull a copy to your own machine

Apply the same categories from the web section above: config files, credential files, stray notes — a plaintext file sitting in a share is exactly as worth investigating as a suspicious web path.

> **Learning Narrative:** SMB enumeration follows the identical logic as web enumeration — find what's reachable, then actually open and read it. The protocol changes; the discipline doesn't.

---

## 4. Metadata & Document-Based Enumeration

Files carry information beyond their visible content.

### Extracting Hidden Metadata

```bash
exiftool document.pdf
```

Metadata can reveal an author name, the software used to generate the file, sometimes a version number — which then becomes searchable in its own right.

```bash
searchsploit "wkhtmltopdf 0.12.6"
```

A version string found buried in a document's metadata is just as valid a lead as one found via a service banner — the source differs, the value doesn't.

### Extracting Emails from GitHub Commits

A specific, reliable technique: adding `.patch` to the end of any GitHub commit URL returns the raw commit output — which includes the committer's real email address, even when it's hidden everywhere else on their public profile.

> **Learning Narrative:** Enumeration isn't limited to services and ports. Any file a target has published — a PDF, a public commit — can carry information never meant to be extracted, if you know to check.

---

## 5. OSINT as Enumeration's Foundation

Passive, public-information gathering that directly feeds every stage above.

| Technique | What it feeds |
|---|---|
| Email format discovery | Turns a guessed username pattern into a checkable, specific target |
| Breach-checking (Have I Been Pwned) | Confirms whether a found or guessed credential has ever leaked |
| Subdomain hunting (`crt.sh`, certificate transparency logs) | Surfaces forgotten assets — old, unpatched, less-watched than the main site |
| Google dorking (`site:`, `filetype:`, `inurl:`) | Surfaces indexed files never meant to be found this way |
| Social media OSINT | Reveals who holds relevant access — useful once already inside, hunting for the next credential worth chasing |

> **Learning Narrative:** OSINT isn't a separate phase that happens before enumeration — it's the same activity, aimed outward before you have any access, instead of inward once you do.

---

## 6. The Pivot — Connecting What You Find

The actual payoff of enumeration is rarely a single, isolated finding. It's connecting one piece of intelligence to a completely different door.

**The pattern:** credentials, a version number, or a hint found via one service or technique get tested against an entirely unrelated one — a password found in a file share tried against a web login; a version found in a document's metadata searched against an exploit database; an email extracted from a commit checked against breach data.

> **Learning Narrative:** This is the real skill enumeration is building toward — not finding any one thing, but recognizing when something found in one place is actually the key to a door somewhere else entirely.