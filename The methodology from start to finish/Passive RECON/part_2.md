 
Passive Reconnaissance  
 
# Beginner's Guide to Passive 
# Reconnaissance Tools in Cyber Security 
The internet is full of exposed data — forgotten subdomains, 
unsecured databases, and outdated servers just waiting to be found. 
The best way to defend against this? Learn how to use the same tools 
hackers rely on! 
In this guide, I’ll break down the top passive reconnaissance tools in 
cybersecurity, a quick overview of how they work, and why they 
matter. 
Let’s dive in. 
# Sidenote
A word of warning! 
Before you dive into passive reconnaissance, there’s something you 
need to know. Just because data is public doesn’t mean you can use it 
however you want. 
Yes, DNS records, SSL certificates, and certain metadata are meant to 
be visible. But poking around misconfigured databases, scraping 
restricted content, or accessing sensitive files? That’s where you can 
cross legal boundaries. 
In the U.S. and some other, the Computer Fraud and Abuse Act (CFAA) makes it clear: 
if you access data without authorization — even if it’s exposed — you 
could be breaking the law. Other countries have similar regulations, 
and companies often have strict rules about what information can be 
accessed, even passively. 
If you’re using these tools for ethical hacking or security research, 
always get permission first. Otherwise, what starts as curiosity could 
quickly turn into unauthorized access and legal trouble. 
Now that you know the risks, let’s look at the first set of tools. 
What’s already exposed? More than you might think 
Before you can assess a company’s security, you need to know what’s 
already out there. Many organizations lose track of old subdomains, 
forgotten servers, and public-facing services — without realizing 
they’re still accessible. 
For example 
A company might have set up a staging site (staging.example.com) for 
testing a few years ago. It was never meant to be public, but if it’s still 
live and running outdated software, attackers can find it and use it as 
a weak entry point. 
That’s where passive reconnaissance tools come in. These tools let you 
map out everything tied to a domain without ever touching the target 
directly. You can uncover: 
 Hidden subdomains that weren’t meant to be public 
 Forgotten servers still hosting outdated applications 
 Misconfigured services that could be exploited 
 SSL certificates and DNS records that reveal infrastructure 
details 
The goal? Find these weak spots before an attacker does. 
One of the easiest ways to uncover forgotten subdomains? SSL 
certificates. Even if a subdomain isn’t listed in DNS records or indexed 
by search engines, it still needs a certificate to enable HTTPS. And 
certificates are public. That’s where crt.sh comes in. 
### 1. crt.sh 
Most companies assume that if a subdomain isn’t listed anywhere, it 
stays private. 
However this isn’t true. 
Every time a company sets up HTTPS for a site — even a private, 
internal one — they issue an SSL certificate. These certificates get 
logged in public Certificate Transparency (CT) databases to prevent 
fraud. 
And that’s exactly what crt.sh tracks. 
 
# Why does this matter? 
Let’s say a company is developing a new payments portal 
at payments.example.com. They haven’t launched it yet, it’s not linked 
anywhere, and search engines don’t know about it. But the second they 
request an SSL certificate, crt.sh logs it. 
Attackers will actively monitor Certificate Transparency logs to find 
weak points before companies even realize they exist. And if an 
attacker finds a subdomain before it’s secured, they can: 
 Find login pages before access controls are set up 
 Scan for vulnerabilities in outdated test environments 
 Spot mergers, acquisitions, or new product launches — and 
launch targeted phishing attacks 
# For example 
Running a search for example.com on crt.sh might return: 
vpn.example.com    
admin.example.com    
secure.example.com    
payments.example.com    
So even if the company thought these were private, they’re now visible 
to anyone who knows where to look. That’s why crt.sh is one of the 
first tools used in passive reconnaissance. 
Once you’ve checked for SSL leaks, the next step is mapping what’s 
officially registered in DNS records. 
### DNSDumpster 
Finding leaked subdomains is just the start. Your DNS records can 
reveal even more. 
That’s where DNSDumpster comes in. 
 
# Why care? 
Old test sites, outdated mail servers, third-party services etc. They all 
leave a trail in DNS records, and once something is in DNS, it doesn’t 
just disappear. 
# For example 
A company might have once used vpn.example.com for remote access. 
Years later, they switch to a new system but no one deletes the old 
DNS record. Now, that subdomain still exists, even if the server behind 
it is long gone. 
This happens all the time, and companies will often leave behind: 
 Mail servers that still accept connections (mail.example.com) 
 Abandoned test environments (files.example.com) 
 Third-party analytics services (tracking.example.com) 
 Outdated name servers that could expose the entire domain 
Individually, these might not seem like a big deal. But to an attacker, 
they’re a list of possible entry points. 
For example 
Let’s go back to that old VPN subdomain. If an attacker finds it, they 
could: 
 Claim the abandoned IP address and set up a fake login page, and 
use that to steal access passwords from employees. (It’s on an 
official subdomain) 
 Scan the old server for vulnerabilities no one’s patched 
 Slip past security controls that still trust the domain 
Companies rarely check for these forgotten records. Their focus is on 
what’s currently running — not what they left behind. 
That’s why tools like DNSDumpster are so useful. They don’t just tell 
you what’s live today — they show you everything still tied to a 
domain, even if no one’s looked at it in years. 
But DNS records only show what’s officially registered. What about 
assets that were never supposed to be public in the first place? 
That’s where Amass comes in. 
### Amass 
 
Unlike the previous tools we’ve covered so far, Amass doesn’t rely on 
existing records. Instead, it combines multiple techniques to map an 
organization’s entire attack surface, including: 
 Cross-referencing public sources (like DNSDumpster and crt.sh) 
 Brute-forcing potential subdomain names (guessing common 
names like dev.example.com or admin.example.com) 
 Checking third-party datasets (registrations, API endpoints, and 
archived domain data) 
And for security teams, that’s both a blessing and a problem. 
# For example 
Let’s say an attacker wants to target the made up website example.com. 
Instead of relying on what’s publicly listed, they run Amass and 
discover: 
 internal.example.com → An internal tool that was never meant 
to be exposed 
 staging.example.com → A test environment running outdated 
software 
 dev.example.com → A developer portal with weak authentication 
 test-api.example.com → An API endpoint with no access 
restrictions 
 legacy-db.example.com → An old database still accessible from 
the internet 
Each one of these is a potential entry point. 
And so instead of attacking the main website, an attacker might: 
 Target a forgotten staging server that lacks proper security 
controls 
 Exploit outdated software running on an old development 
environment 
 Try credential stuffing attacks on an exposed internal login page 
Scary stuff eh? 
Identifying exposed devices and services 
By now, we’ve mapped out a company’s public-facing infrastructure 
from leaked SSL certificates to forgotten subdomains. But knowing a 
domain exists isn’t enough. 
The real question is: What’s running on those domains and IPs? 
Many companies accidentally leave sensitive services exposed without 
realizing it. Things like: 
 Databases that should be private but are accessible from the 
internet 
 Admin panels left open with default passwords 
 Remote access services (RDP, SSH, VNC) that hackers can exploit 
 Security cameras and IoT devices broadcasting live feeds online 
These misconfigurations happen all the time and attackers know 
exactly how to find them. 
The next set of tools scans the internet for publicly accessible services 
and devices. The things that should be locked down but aren’t. 
Let’s start with Shodan, the most well-known search engine for 
exposed infrastructure. 
### Shodan 
Unlike Google, which indexes websites, Shodan scans the entire 
internet for publicly accessible devices, servers, and services. 
 
If something is online and listening for connections, Shodan will find 
it. 
# Anything from: 
 Unsecured databases (MongoDB, Elasticsearch, MySQL) 
 Exposed industrial control systems (power grids, SCADA, 
security cameras) 
 Open remote access services (RDP, SSH, VNC, Telnet) 
 Misconfigured cloud storage (Amazon S3, Google Cloud Buckets) 
# For example 
Let’s say a company sets up a remote desktop server (RDP) for internal 
use but forgets to restrict access. It could be wide open to the 
internet—with no password. 
A simple Shodan query like "Remote Desktop" port:3389 country:US could 
reveal hundreds of exposed RDP servers, some still using default 
credentials. 
This kind of exposure isn’t rare either. In 2020, a misconfigured 
Elasticsearch database leaked 250 million Microsoft customer records 
— including support logs, emails, and IP addresses. 
Anyone running a single Shodan search could have found it before 
Microsoft even knew it was exposed. 
The good news though is that just because something is online doesn’t 
mean it’s automatically vulnerable. 
 Maybe a database is exposed, but it requires authentication 
 Maybe an RDP server is open, but it’s using strong access 
controls 
The bad news? Attackers don’t stop at just using Shodan . 
### Censys 
Unlike Shodan, which focuses on listing open services, Censys goes 
deeper by analyzing security risks. 
 
# For example 
An attacker searching Censys for example.com wouldn’t just see a list 
of running services. They might also see specific weaknesses, like: 
 An OpenVPN server running an outdated version with known 
exploits that could allow an attacker to bypass authentication 
entirely 
 A Microsoft Exchange server missing critical security patches 
which could be exploited for remote code execution 
 A development server leaking sensitive data through 
misconfigured HTTP headers 
 A MongoDB database sitting open on the internet with no 
authentication wouldn’t even require hacking — anyone could 
access its contents with the right query 
These aren’t just technical flaws — they’re direct opportunities for 
exploitation. An attacker isn’t going to waste time on a locked-down 
system when they can find an exposed database that hands over 
information without resistance. 
That’s what makes Censys powerful. Instead of just listing services, it 
helps security teams (and hackers) identify the ones that matter 
most. 
That being said - the tool does have some flaws in that although it can 
quickly flag vulnerable systems, investigating every exposed service 
manually is still a slow process. 
So when security teams need to gather intelligence across multiple 
targets and automate their workflow, they turn to more advanced 
tools. 
This is where Recon-ng comes in. 
### Recon-ng 
Security teams often need more than just lists of exposed services. 
They need to connect the dots between different data points and 
automate repetitive tasks to work efficiently. 
Recon-ng helps with this, and acts as a centralized framework for 
gathering intelligence, similar to how Metasploit is used for 
exploitation. 
 
# With Recon-ng, you can: 
 Automate OSINT tasks instead of performing them manually 
 Pull data from WHOIS records, search engines, breach databases, 
and other sources 
 Identify linked domains, email addresses, and infrastructure 
details 
 Generate structured reports that make analysis faster and more 
effective 
Instead of juggling scattered search results from multiple tools, Recon
ng consolidates everything into a single database, making it easier to 
analyze patterns and prioritize risks. 
Handy right? 
Let’s not stop here though, because not all reconnaissance is about 
finding servers and infrastructure. Sometimes, the most valuable 
intelligence comes from people. Employees, administrators, and 
developers who may unknowingly expose credentials, emails, and 
usernames online. 
That’s where theHarvester comes in. 
### theHarvester 
 
theHarvester is designed to gather personal identifiers that attackers 
can use for phishing, credential stuffing, or social engineering. 
# With a simple query, it can: 
 Find email addresses linked to a domain from public sources 
 Identify usernames from GitHub, forums, and other platforms 
 Check employee names and leaked credentials in breach 
databases 
 Cross-reference results to uncover patterns and potential attack 
vectors 
This matters because even a single email address can reveal a 
username pattern used across internal systems. A leaked password 
might still be active on company accounts. Even an old forum post can 
expose sensitive details that shouldn’t be public. 
Attackers use this information to: 
 Craft targeted phishing emails using real employee names 
 Check if usernames appear in leaked password databases 
 Guess passwords based on known patterns (e.g., 
"CompanyName2024!") 
However, they never just send an email blast like you might expect 
from random spam emails. Instead, hackers use these to be much more 
targeted… 
Shifting from raw data to intelligence mapping 
So far, we’ve covered tools that help uncover exposed infrastructure, 
find vulnerable services, and collect personal identifiers like emails 
and usernames. But reconnaissance isn’t just about finding individual 
data points. 
The real power comes from connecting the dots and seeing how 
domains, people, and systems relate to one another to map an 
entire attack surface. 
This next set of tools helps security professionals turn scattered 
information into actionable intelligence by identifying relationships, 
uncovering hidden links, and automating large-scale analysis. 
Basically? They can make that fake email not only sound like it came 
from the right person - but they can send it from their email account 
also… 
### Maltego 
 
# With Maltego, you can: 
 Analyze relationships between people, domains, IP addresses, 
and infrastructure 
 Pull data from multiple sources, including WHOIS records, social 
media, and breach databases 
 Visually map connections to uncover hidden links that might 
otherwise go unnoticed 
 Pivot between entities to uncover additional intelligence with 
just a few clicks 
# Why care? 
We’re always told to check who sent the email right? Well, a phishing 
email is far more convincing if it actually comes from a real company 
email account. Likewise, a leaked admin username is far more 
dangerous when linked to an exposed login page. 
That’s what makes Maltego so powerful — it pieces together seemingly 
unrelated data to reveal the bigger picture. 
# For example 
A security team (or hacker) using Maltego might: 
 Start with a leaked email address found using theHarvester 
 Check if that email appears in breach databases 
 Discover that the same username pattern was used on an 
administrator login page 
 Map the IP of the login page back to the company’s main 
infrastructure 
Instead of looking at isolated pieces of information, Maltego connects 
everything into a visual map, making patterns and attack paths 
obvious. 
But when security teams need to analyze thousands of data points at 
once, manually mapping relationships can become overwhelming. 
That’s where SpiderFoot comes in. 
### SpiderFoot 
Maltego is great for mapping connections, but what if you need to 
analyze thousands of data points at once? Investigating each entity 
manually takes time — especially when dealing with multiple targets, 
large organizations, or constantly changing infrastructure. 
SpiderFoot solves this by automating large-scale OSINT collection and 
correlation. Instead of manually running searches across different 
tools, SpiderFoot scans hundreds of sources automatically, pulling 
everything into a structured risk report. 
 
With SpiderFoot, you can: 
 Collect and analyze OSINT data from hundreds of public sources 
 Identify patterns, anomalies, and security risks across multiple 
organizations 
 Detect exposed credentials, infrastructure vulnerabilities, and 
weak points 
 Automate reconnaissance without manual effort 
A single leaked email or exposed server might not seem like a big deal 
on its own. But when combined with other data points, it can paint a 
much bigger picture — one that attackers can use to breach a system. 
For example 
Let’s say a company’s VPN subdomain is still active but unmonitored. 
On its own, that’s not an immediate red flag. But now, imagine a 
SpiderFoot scan finds: 
 vpn.example.com (discovered via DNS records) 
 192.168.1.50 (the VPN server’s IP, exposed via Shodan, running 
outdated software) 
 admin@example.com (an employee’s email, found on LinkedIn and 
in a breach database) 
 Password123 (previously leaked in a credential dump that’s 
potentially still in use) 
By automatically linking these findings, SpiderFoot doesn’t just list 
data — it exposes security gaps that attackers could exploit. This 
means that what seemed like random bits of information now forms a 
clear attack path: 
A leaked admin email → A known password pattern → An outdated 
VPN server → Direct access to internal systems 
This is why you should always use different passwords! Having a 12 
character password might be hard to crack, but if you use the same 
one on an e-commerce site with poor security, all it takes now is for 
the hacker to find this password and try it elsewhere. 
# So what's next? 
So far, we’ve been focused on finding the kind of information that only 
a hacker would know how to uncover — leaked credentials, forgotten 
subdomains, and exposed infrastructure. 
But sensitive data doesn’t always come from misconfigured servers or 
breached accounts. Sometimes, the biggest security risks are hiding in 
plain sight—inside documents, spreadsheets, and images that were 
never meant to be public. 
That’s where FOCA comes in. 
### FOCA 
Not all sensitive information is locked away behind passwords or 
firewalls. Sometimes, companies unknowingly expose valuable data 
inside everyday files such as Word documents, PDFs, spreadsheets, 
and even images. 
These files may look harmless, but they often contain hidden metadata 
that can reveal internal usernames, software versions, file paths, and 
other details that attackers can use to their advantage. 
FOCA (Fingerprinting Organizations with Collected Archives) is 
designed to extract and analyze metadata from public documents, 
helping security teams uncover leaks before attackers do. 
 
# With FOCA, you can: 
 Analyze documents for hidden metadata, including usernames, 
email addresses, and file paths 
 Identify software versions used to create the document, which 
may reveal outdated or vulnerable software 
 Extract internal network details, such as shared drive locations 
or IP addresses 
 Discover who edited the document and when, which can provide 
insight into an organization’s internal structure 
# For example 
Imagine a company uploads a technical manual for its employees, 
thinking it contains nothing sensitive. But a quick scan with FOCA 
reveals: 
 Author: Jane Smith 
 Company: Acme Corp 
 Software: Adobe InDesign CS5 (outdated) 
 Internal email: jane.smith@acme.com 
 File path: \\acmecorp\secure\engineering\confidential\blueprints\2024_plans.docx 
At first glance, this might not seem like a security risk. But for an 
attacker, it’s a goldmine of information: 
 The internal file path confirms the company’s folder structure, 
revealing potential targets for cyberattacks 
 The email address can be used for phishing or social 
engineering 
 The outdated Adobe software version could indicate a known 
vulnerability that hackers can exploit 
 The document location ("confidential/blueprints") suggests it 
contains proprietary or sensitive information 
If this document were indexed by a search engine, anyone could find 
and analyze it without needing to hack anything. 
Speaking of which… 
### Google 
Most people use Google to find websites. Hackers use it to find easy 
security holes. 
You see, the issue is that Google indexes everything unless you tell it 
not to. And so if a company forgets to restrict access, sensitive data 
can show up in search results and be found by anyone who knows 
what to look for. 
# Things like: 
 Publicly accessible PDFs, spreadsheets, and Word docs 
 Login pages that weren’t meant to be exposed 
 Misconfigured databases leaking customer data 
# For example 
In 2021, security researchers used Google Dorking to find a 
government database containing sensitive personal records — 
completely exposed because it wasn’t properly secured. No hacking 
was needed; it was indexed by Google. 
All it took was a simple query like ‘site:gov.example.com filetype:xls’, 
and just like that, they found confidential information. 
 
It sounds unbelievable, but this happens all the time. 
# Fun fact: I did this same search for my local government and found 
over 100 xls files available… 
Try these out for yourself 
So as you can see, the right tools can make all the difference in passive 
reconnaissance. From mapping exposed infrastructure 
with Amass and DNSDumpster to uncovering leaked credentials 
with theHarvester and SpiderFoot, each tool provides a unique way 
to gather intelligence without direct interaction. 
Want to see what’s publicly exposed? Try these tools on your own 
systems or with explicit permission from the owner. Testing third
party systems without consent — even passively — can cross legal 
boundaries. 
If you find something unexpected, secure it ASAP because if you can 
access it, so can an attacker. 