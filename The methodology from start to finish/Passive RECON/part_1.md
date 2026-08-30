# Passive Reconnaissance — Notes 
# Based on TCM's Practical Ethical Hacking, Information Gathering section — updated for 
what's actually current in 2026. 
The One Idea Behind All of This 
Passive recon means learning everything you can about a target without ever touching their 
systems. Nothing gets sent to them. Nothing gets knocked on. You're a detective reading 
newspaper clippings and public records before ever visiting anyone's house. 
Every tool below is just a different way of reading public records. That's it. Hold onto that, 
and none of these tools will feel random. 
One important thing this section sets up for you: because everything here is passive — 
just reading what's already public — it's safe and fair to practice on literally any real 
company you want. You're not touching their systems, just reading what's already out there. 
That changes once you get to Scanning & Enumeration next, where you start actually poking 
at live systems and scope/permission become serious. For now, pick any company and go. 
How to practice this properly: don't do all seven pieces on seven different companies. Pick 
ONE real company. Run every technique below against that same one target, back to back. 
By the end you'll have a real, connected recon profile — not seven disconnected exercises. 
1. Identifying the Target 
Before investigating anything, you need to know exactly what's fair game. 
 In real paid work: bug bounty programs (HackerOne, Bugcrowd) publish an exact 
scope — which domains and apps are in-bounds, which are off-limits. You never 
guess; you check the program page. 
 While practicing: pick any public company yourself. Write down just their main 
domain. Nothing else yet. That single line is your anchor for everything below. 
Try it: pick a company right now. Write its domain on a sticky note or a text file. That's step 
one done. 
2. Finding Email Addresses 
Once you know the domain, the next move is figuring out how their email addresses are built 
— because that pattern unlocks searching for real addresses everywhere else, including the 
breach check you already ran. 
Common patterns: firstname.lastname@, firstinitiallastname@, firstname@ 
Tools: 
 Hunter.io — free tier lets you search a domain and see the email pattern(s) they've 
already found in public sources. 
 theHarvester — free, open-source, comes built into Kali Linux by default. It 
automatically pulls emails, names, and subdomains from search engines and other 
public sources in one pass. 
Try it: 
theHarvester -d company.com -b google 
(-d is the domain, -b is which public source to pull from — check --help since exact flags 
shift slightly between versions.) 
3. Breached Credentials (recap — you've already done the 
hands-on part) 
Why it matters: people reuse passwords. If someone's password leaked in an unrelated 
breach years ago and they're still using it today, that's a real weakness — found without 
attacking anything. 
 breach-parse — free, built by Heath Adams (TCM's founder) himself. But it 
searches a locally-downloaded ~40GB breach dataset, not the live internet. Free in 
money, expensive in bandwidth/storage/time — a genuine bad fit for your current 
setup, and that's a hardware mismatch, not a skill gap. 
 DeHashed — confirmed still paid in 2026, roughly $0.02/search plus subscriptions. 
 Have I Been Pwned — free, web-based, no download. The real 2026 answer for you, 
and you've already used it. 
4. Subdomain Hunting 
A company's main website is just the front door. Most companies have dozens of other 
addresses — staging sites, admin panels, forgotten old projects — and those side doors are 
usually far less watched than the front one. 
Tools: 
 sublist3r — searches public sources for known subdomains. 
 amass — more powerful, actively maintained, pulls from many data sources at once. 
 crt.sh — a clever one: every HTTPS certificate a company gets issued becomes 
public record, and that record usually lists every domain name the certificate covers 
— including subdomains nobody ever linked to from anywhere. 
Try it (no install needed, purely web-based, easiest one to start with on your setup): go 
to crt.sh, search your target's domain, and see what other subdomains show up in the 
certificate history. 
5. Website Tech Fingerprinting 
Knowing what software or framework runs a site tells you what known weaknesses might 
apply, since specific software has specific, documented issues. 
Tools: 
 Wappalyzer — free browser extension. Install once, visit any site, it instantly shows 
the tech stack. 
 BuiltWith — same idea, as a website instead of an extension. 
 whatweb — the command-line version, if you're working from terminal. 
Try it: install the Wappalyzer extension, visit your target's site, note what it reports. 
Lightweight, browser-based, works fine even on slower hardware since it's just an extension, 
not a heavy tool. 
6. Google Fu (Google Dorking) 
Google already crawled and indexed far more than most companies realize — including 
things they never meant to expose. Google Fu is learning the search language that surfaces it. 
Key operators: 
 site: — only results from one domain 
 filetype: — only a specific file type (e.g. pdf) 
 intitle: — word must appear in the page title 
 inurl: — word must appear in the URL itself 
Example: 
site:company.com filetype:pdf 
This finds every PDF Google indexed on that domain — sometimes internal documents that 
were never meant to be public but got crawled anyway. 
Resource: the Google Hacking Database (GHDB) on exploit-db.com — a long-running, 
community-maintained list of useful dork combinations. 
Try it: run that exact search on your target's domain and see what comes back. Free, instant, 
just a search bar. 
7. Social Media OSINT 
People post more than they realize. Employee LinkedIn profiles alone often reveal job titles, 
internal tool names, and team structure — all voluntarily public. 
Techniques: 
 LinkedIn employee search — see who works there, what roles exist 
 Company blog posts / conference talks — sometimes reveal internal tools or tech 
choices in passing 
Tool: 
 Sherlock — free, open-source, searches for a given username across many social 
platforms at once. Most useful once you already have a suspected username to check 
breadth on. 
Try it: search your target company's name on LinkedIn, see how much role and team
structure info is visible from public profiles alone. 
Putting It Together 
Run all seven steps against the same one company, in order, and you'll end up with a real 
recon profile: a domain, a guessed email format, any breach history, a list of subdomains, the 
tech stack behind them, whatever Google Fu surfaces, and a sense of who works there. That's 
the actual deliverable passive recon produces — not seven separate party tricks, one 
connected picture, built entirely from public information. 
When you're ready, Scanning & Enumeration is next — and that's where scope and 
permission start to really matter, since you're no longer just reading, you're touching. 