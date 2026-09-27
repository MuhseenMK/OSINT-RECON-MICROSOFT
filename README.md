
# 🔎 OSINT Reconnaissance — Microsoft.com

**Student:** Muhammad Muhsin Khamis  
**Program:** Cybersecurity — Self-Directed Project  
**Target:** microsoft.com  
**Scope:** Passive OSINT only — no active scanning, no exploitation

---

## Why I did this

I previously used theHarvester against `microsoft.com` during an authorised lab at my internship. That lab was guided — I was told exactly which command to run.

I wanted to know what else is possible. So I picked up OSINT as a self-study topic, watched David Bombal's OSINT walkthrough, and started experimenting with tools I hadn't used before.

This repo is the result. No instructor brief. No lab script. Just me, the tools, and a target.

---

## Scope & ethics

- All activities were **passive**.
- No systems were scanned, probed, or accessed.
- All information gathered is **already public**.
- No private individual's data was published — any leaked emails found in metadata are **redacted** in this repo.
- Target chosen because it was already authorised during my earlier internship lab work.

---

## Tools used

| Tool | Purpose | Status |
|---|---|---|
| Whois | Domain registration lookup | ✅ Worked |
| Wayback Machine (browser) | Historical website snapshots | ✅ Worked |
| `waybackurls` + CDX API | Bulk historical URL enumeration | ⚠️ API timed out — documented |
| Have I Been Pwned | Email breach exposure | ✅ Worked |
| Censys | Infrastructure / certificate recon | ⚠️ Free tier hung — documented |
| Shodan | Internet-connected device search | ✅ Worked |
| Maigret | Username enumeration across platforms | ✅ Worked (DNS issues) |
| ExifTool | Document metadata extraction | ✅ Worked — **found a leak** |
| Holehe | Email registration check | ✅ Worked (all rate-limited) |

---

## Task 1 — Whois Reconnaissance

Ran:

```bash
whois microsoft.com

The output revealed more than I expected.

What I found
Registrar: MarkMonitor Inc. — enterprise-grade registrar used by large corporations. Not a consumer registrar.

Created: 1991-05-02 — one of the oldest domains on the internet.

Expires: 2027-05-03 — auto-renewed.

Six domain locks are in place (client + server). Three prevent deletion, transfer, and update. Hijacking this domain is effectively impossible.

Name servers: NS1-39.AZURE-DNS.COM/.NET/.ORG/.INFO — Microsoft hosts its own DNS across four different TLDs. Redundancy by design.

DNSSEC: unsigned — Microsoft doesn't have DNSSEC enabled. Worth noting as a low-risk observation.

https://whois.png/

https://whois2.png/

Task 2 — Wayback Machine
Part A — Browser method
I searched web.archive.org for microsoft.com and picked a snapshot from 17 February 2015.

The page looked completely different from today:

Navigation: Store / Explore / Devices / Software & apps / Support — product-centric structure

Homepage promoted Office 365, Lumia 830, Xbox One — Microsoft was still in the phone business

No mention of Azure, AI, or Copilot

URL structure still used .aspx extensions — the pre-modern-routing era

This is real OSINT. It reveals which products Microsoft has discontinued (Lumia, Windows Phone), old subdomains that may still resolve today, and historical marketing priorities.

https://wayback.png/

https://wayackdate.png/

https://wayback1.png/

https://wayback2.png/

Part B — CLI method (failed)
I tried waybackurls microsoft.com — it silently returned nothing. The tool hasn't been maintained and the CDX API has changed.

I then tried fetching bulk URLs directly from the API:

bash
curl -s "http://web.archive.org/cdx/search/cdx?url=microsoft.com*&output=text&fl=original&collapse=urlkey&limit=5000"
Got a 504 Gateway Time-out. The Wayback Machine's free API can't handle Microsoft-scale queries. I moved on rather than fight it.

Task 3 — Have I Been Pwned
Checked two public role-based Microsoft addresses.

Address	Breaches
press@microsoft.com	2
abuse@microsoft.com	23
What this means
abuse@microsoft.com appearing in 23 known breaches is a real signal:

Role-based public addresses get scraped, published, and leaked constantly

They're viable targets for phishing and spam

Microsoft's security team must handle a huge volume of malicious traffic to these mailboxes

Role-based public emails should be treated as compromised by default.

https://press.png/

https://abuse.png/

Task 4 — Censys (deferred)
I signed up for Censys Free, searched microsoft.com, and the results never loaded. The UI sat on "Searching every corner of the internet…" indefinitely.

This is a known limitation of Censys Free. Rather than waste time, I documented it and moved on. The same certificate-transparency data is available through crt.sh — deferred to focus on the tasks that worked.

Task 5 — Shodan
Searched hostname:microsoft.com on Shodan.

Headline numbers
43,561 total hosts

Top country: United States (11,184)

Top port: 443 (HTTPS, 27,249 hosts)

What surprised me
Amazon.com, Inc. hosts more Microsoft-associated IPs (9,490) than Microsoft Corporation itself (7,175).

Microsoft runs a large portion of its public infrastructure on AWS — its biggest cloud competitor.

Other findings:

9,895 servers still respond on plain HTTP (port 80) — a big legacy surface

Non-standard ports in use: 8443, 4443, 8800 — likely admin panels and legacy services

Only 8.4% of visible servers run Microsoft IIS — the rest are Kerio, nginx, Apache, CloudFront

Kerio Connect webmail appears on 12,254 hosts — a third-party email server, not Microsoft Exchange

https://shodan1.png/

https://shodan2.png/

Task 6 — Maigret Username Enumeration
Ran:

bash
maigret microsoft --top-sites 50
Findings
16 accounts found under the username microsoft

Auto-discovered additional usernames: OpenAtMicrosoft, Microsoft, LenovoYoga3Pro

Confirmed official accounts on: GitHub, Twitter, Facebook, YouTube, Telegram, LinkedIn, Medium, WordPress, Spotify, Vimeo, Bit.ly, SoundCloud, Tumblr, Flickr

Interesting details
GitHub: github.com/microsoft — 8,312 public repositories, 130,567 followers, account created 2013-12-10

Twitter: 13,074,050 followers, account created 2009-09-14

Facebook: 13,287,807 followers

Telegram bio: simply "Microsoft.com"

Tool limitation
Maigret reported 62% DNS resolution failures using its default async DNS resolver (aiodns). On high-latency networks this is common. The documented workaround is --dns-resolver threaded.

https://maigret1.png/

https://maigret2.png/

https://maigret3.png/

Task 7 — ExifTool Metadata Extraction
This is the task that found something real.

I downloaded three public documents from Microsoft's Investor Relations page:

2025_AnnualReport.docx

MSFT_FY25q4_10K.docx (the SEC 10-K filing)

2025_Shareholder_Letter.docx

Then ran ExifTool on each:

bash
exiftool ~/Downloads/MSFT_FY25q4_10K.docx
What I found
Microsoft did clean the visible metadata:

Creator: empty

Last Modified By: empty

Company: empty

Total Edit Time: reset to 0

That's a deliberate cleaning process. Most organisations don't even do this much.

But they missed the hyperlinks.

The H Links field — an obscure metadata field that most cleaning tools don't touch — still contained embedded mailto: hyperlinks to four real Microsoft email addresses of investor relations and communications staff.

Why this matters
1. Confirms Microsoft's email format — [lastname]@microsoft.com and [initial][lastname]@microsoft.com
2. Identifies who handles SEC filings internally — finance + comms teams
3. Creates a social engineering vector — a targeted phishing list against Microsoft's investor relations
4. Shows metadata cleaning is incomplete at the hyperlink layer — most tools check visible fields, not embedded links

Redaction
The leaked email addresses are not published in this repo. I've documented the finding and its significance without reproducing the personal information of real individuals. That's standard OSINT ethics.

Dates leaked
ExifTool also revealed internal document timelines:

. 10-K: created 2025-07-29 18:48 UTC

. Annual Report: created 2025-10-16 16:47 UTC

. Shareholder Letter: created 2025-10-15 18:45 UTC

These map Microsoft's internal financial reporting calendar.

https://email1.png/

https://msft1.png/

https://msft2.png/

https://msft3.png/

https://annual_report11.png/

https://shareholders1.png/


Task 8 — Holehe Email Registration Check
Ran:

bash
holehe press@microsoft.com
holehe abuse@microsoft.com
Holehe checked 121 platforms against each address.

Result
All 121 checks returned [x] — rate-limited.

That means the platforms blocked Holehe's enumeration for these specific addresses. Microsoft's role-based emails are so widely known and abused that platforms defend against them at scale.

This is a finding in itself:

Confirms that press@ and abuse@ are actively targeted and defended

Validates the HIBP result (23 breaches on abuse@)

Shows the security posture around Microsoft's public-facing emails

Screenshot for Task 8 not captured separately — the terminal output is shown in this repo's notes.

Risk Analysis
#	Finding	Impact	Risk Level
1	Microsoft public emails leaked in HIBP (up to 23 breaches on abuse@)	High phishing exposure	High
2	Real Microsoft emails found in 10-K metadata hyperlinks	Spear-phishing vector against investor relations	High
3	Internal document dates leaked in metadata	Reveals internal reporting calendar	Medium
4	9,895 servers respond on plain HTTP (port 80)	Legacy surface, possible downgrade attacks	Medium
5	DNSSEC not enabled on microsoft.com	Theoretical DNS spoofing risk	Low
6	Non-standard ports exposed (8443, 4443, 8800)	Possible admin/legacy interfaces	Medium
7	Almost 60% of Microsoft-associated IPs hosted on AWS	Third-party cloud risk surface	Low
8	Public role-based emails are 100% rate-limited by platforms	Defensive control (positive)	✅ Positive
Risk key: Critical / High / Medium / Low

What I learned
1. Passive OSINT can go deeper than I expected. Whois, Wayback, Shodan, and ExifTool together gave me a much fuller picture than theHarvester alone.

2. Metadata is one of the most under-defended leak vectors. Microsoft cleaned their visible metadata but missed hyperlinks. That's a real, recurring mistake across many organisations.

3. Tool failures are normal. Three of the eight tasks involved tools that didn't work. Documenting the failures was as important as the successes.

4. Platforms defend known targets. Holehe returning 100% rate limits on Microsoft addresses shows that high-profile role emails are protected at scale.

5. Ethics matters in OSINT. Finding leaked data doesn't mean publishing it. Redaction is what separates a professional from an amateur.

Repository Structure
.
├── README.md
├── whois.png
├── whois2.png
├── wayback.png
├── wayackdate.png
├── wayback1.png
├── wayback2.png
├── press.png
├── abuse.png
├── shodan1.png
├── shodan2.png
├── maigret1.png
├── maigret2.png
├── maigret3.png
├── email1.png
├── MSFT1.png
├── MSFT2.png
├── MSFT3.png
├── annual_report11.png
└── Shareholders1.png

Author
Muhammad Muhsin Khamis
Cybersecurity Student — Bayero University Kano
Self-directed learner
LinkedIn: https://www.linkedin.com/in/muhammad-muhsin-khamis-9860b3311/
