# 🔎 OSINT Reconnaissance — Microsoft.com

**Student:** Muhammad Muhsin Khamis
**Program:** Cybersecurity — Self-Directed Project
**Target:** `microsoft.com`
**Scope:** Passive OSINT only — no active scanning, no exploitation

---

## Why I did this

I previously used theHarvester against `microsoft.com` during an authorised lab at my internship. That lab was guided — I was told exactly which command to run.

I wanted to understand what else was possible. So I picked up OSINT as a self-study topic, watched David Bombal's OSINT walkthrough, and started experimenting with tools I had not used before.

This repository is the result.

No instructor brief. No lab script. Just me, the tools, and a target.

---

## Scope & Ethics

* All activities were **passive**.
* No systems were intentionally scanned, probed, or accessed.
* Information collected came from publicly available sources.
* No private individual's data is intentionally published.
* Email addresses discovered in document metadata have been **redacted**.
* The target was selected because `microsoft.com` had already been used as an authorised lab target during my internship.

> **Note:** The results documented here represent observations from the tools and sources used during this project. They should not be interpreted as a complete representation of Microsoft's infrastructure or security posture.

---

## Tools Used

| Tool                    | Purpose                                     | Status                        |
| ----------------------- | ------------------------------------------- | ----------------------------- |
| Whois                   | Domain registration lookup                  | ✅ Worked                      |
| Wayback Machine         | Historical website snapshots                | ✅ Worked                      |
| `waybackurls` + CDX API | Historical URL enumeration                  | ⚠️ API timed out              |
| Have I Been Pwned       | Public email breach exposure                | ✅ Worked                      |
| Censys                  | Infrastructure / certificate reconnaissance | ⚠️ Search did not load        |
| Shodan                  | Internet-facing infrastructure search       | ✅ Worked                      |
| Maigret                 | Username enumeration across platforms       | ✅ Worked with DNS limitations |
| ExifTool                | Document metadata extraction                | ✅ Worked                      |
| Holehe                  | Email registration checks                   | ⚠️ Rate-limited               |

---

# Task 1 — Whois Reconnaissance

### Command

```bash
whois microsoft.com
```

The output revealed several interesting details about the domain.

### What I found

* **Registrar:** MarkMonitor Inc.
* **Created:** 1991-05-02
* **Expires:** 2027-05-03
* Multiple domain locks were present.
* **Name servers:** Microsoft Azure DNS name servers.
* **DNSSEC:** The lookup showed the domain as unsigned at the time of testing.

The domain-registration information provides useful background for reconnaissance, although WHOIS data alone does not provide enough information to determine the security of the domain or its infrastructure.

![Whois output — top](whois.png)

![Whois output — continued](whois2.png)

---

# Task 2 — Wayback Machine

## Part A — Browser Method

I searched the Wayback Machine for `microsoft.com` and selected a snapshot from **17 February 2015**.

The website looked significantly different from the current Microsoft homepage.

### Observations

* Navigation included **Store / Explore / Devices / Software & apps / Support**.
* The homepage promoted products and services including **Office 365, Lumia, and Xbox**.
* The site structure and presentation were different from the current Microsoft website.
* Some URLs still used `.aspx` extensions.

Historical snapshots can be useful during OSINT because they can reveal:

* Older website structures
* Previous product pages
* Historical URLs
* Discontinued services
* Changes in branding and technology
* Potentially forgotten resources

![Wayback Machine homepage](wayback.png)

![Wayback calendar — 2015](wayackdate.png)

![Snapshot — Feb 2015 (part 1)](wayback1.png)

![Snapshot — Feb 2015 (part 2)](wayback2.png)

---

## Part B — CLI Method

I attempted to use:

```bash
waybackurls microsoft.com
```

The command returned no useful results during my test.

I then attempted to query the CDX API directly:

```bash
curl -s "http://web.archive.org/cdx/search/cdx?url=microsoft.com*&output=text&fl=original&collapse=urlkey&limit=5000"
```

The request returned a **504 Gateway Time-out**.

Rather than continuing to repeatedly query the service, I documented the failure and continued with the other OSINT tasks.

### Lesson

Tool failure is also part of practical reconnaissance. A failed tool does not necessarily mean that the data does not exist.

---

# Task 3 — Have I Been Pwned

I checked two **public role-based Microsoft addresses**:

| Address               | Reported breaches |
| --------------------- | ----------------: |
| `press@microsoft.com` |                 2 |
| `abuse@microsoft.com` |                23 |

### What this means

The results demonstrate that public role-based addresses can appear in breach datasets.

This is useful from a defensive OSINT perspective because publicly exposed addresses can become targets for:

* Spam
* Phishing
* Credential attacks
* Social engineering
* Automated abuse

A breach count should not automatically be interpreted as proof that the mailbox itself was compromised. Breach databases can contain addresses that appeared in leaked datasets for different reasons.

![HIBP — press@microsoft.com](press.png)

![HIBP — abuse@microsoft.com](abuse.png)

---

# Task 4 — Censys

I signed up for Censys Free and attempted to search for `microsoft.com`.

The search interface did not successfully return the expected results during my test.

Rather than treating the failed search as a finding, I documented it as a **tool limitation during this investigation**.

Certificate-transparency data can also be investigated through other publicly available sources, but I deferred that part of the investigation to keep the project focused.

---

# Task 5 — Shodan

I searched Shodan using:

```text
hostname:microsoft.com
```

### Results observed during the test

* **43,561 total hosts**
* **Top country:** United States — 11,184
* **Top port:** 443 — 27,249 hosts

These numbers represent what Shodan exposed through the search at the time of my investigation. They should not be treated as Microsoft's complete infrastructure inventory.

### Other observations

The search also showed:

* A significant number of hosts responding on **port 80**
* Various non-standard ports including **8443, 4443, and 8800**
* Multiple technologies and hosting providers associated with the returned results
* Infrastructure appearing to be distributed across different providers

One interesting observation was that Shodan associated a large number of Microsoft-related results with **Amazon.com, Inc.**

This was interesting because Microsoft and Amazon are major competitors in the cloud-computing market, but third-party hosting/CDN/cloud infrastructure can be used by large organisations for many legitimate reasons.

### Important limitation

A Shodan result associated with a hostname does **not automatically mean** that the corresponding system is owned or directly operated by the organisation being investigated.

The results therefore require verification before being treated as confirmed infrastructure ownership.

![Shodan — summary](shodan1.png)

![Shodan — host results](shodan2.png)

---

# Task 6 — Maigret Username Enumeration

### Command

```bash
maigret microsoft --top-sites 50
```

### Findings

The scan identified multiple accounts associated with the username `microsoft`.

Some platforms returned accounts that appeared to correspond to official Microsoft social-media or developer accounts.

The scan also discovered additional usernames during enumeration.

Examples included:

* `OpenAtMicrosoft`
* `Microsoft`
* `LenovoYoga3Pro`

### Interesting results

The enumeration returned official-looking accounts on platforms including:

* GitHub
* X/Twitter
* Facebook
* YouTube
* Telegram
* LinkedIn
* Medium
* WordPress
* Spotify
* Vimeo
* Bitly
* SoundCloud
* Tumblr
* Flickr

One particularly interesting result was:

```text
github.com/microsoft
```

which is Microsoft's official GitHub organisation.

### Tool limitation

Maigret reported significant DNS resolution failures using its default asynchronous DNS resolver.

This affected some of the checks and demonstrated how network conditions can influence OSINT enumeration results.

![Maigret — microsoft results](maigret1.png)

![Maigret — continued](maigret2.png)

![Maigret — additional username discovered](maigret3.png)

---

# Task 7 — ExifTool Metadata Extraction

This was the most interesting part of the investigation.

I downloaded three publicly available documents from Microsoft's Investor Relations resources:

* `2025_AnnualReport.docx`
* `MSFT_FY25q4_10K.docx`
* `2025_Shareholder_Letter.docx`

I then used ExifTool to inspect their metadata.

### Command

```bash
exiftool ~/Downloads/MSFT_FY25q4_10K.docx
```

## What I found

The visible document metadata had been cleaned.

For example:

* `Creator`: empty
* `Last Modified By`: empty
* `Company`: empty
* `Total Edit Time`: reset

However, the investigation showed that embedded hyperlinks could still contain information that was not immediately visible in the standard metadata fields.

The `H Links` field contained `mailto:` hyperlinks associated with Microsoft personnel.

### Why this was interesting

This demonstrated an important OSINT lesson:

> Removing visible document metadata does not necessarily mean that every piece of embedded information has been removed.

Embedded hyperlinks can potentially expose:

* Email addresses
* Organisational relationships
* Internal document references
* Information about the people involved in producing a document

### Redaction

I am **not publishing the discovered personal email addresses** in this repository.

The screenshots containing the finding have been redacted before publication.

The purpose of the exercise is to demonstrate the discovery technique and its defensive implications, not to expose individual employees.

![ExifTool — redacted metadata finding](email1.png)

![ExifTool — 10-K part 1](MSFT1.png)

![ExifTool — 10-K part 2](MSFT2.png)

![ExifTool — 10-K part 3](MSFT3.png)

![ExifTool — Annual Report](annual_report11.png)

![ExifTool — Shareholder Letter](Shareholders1.png)

---

## Document Timeline Observations

ExifTool also exposed document creation timestamps.

The documents contained timestamps including:

* **10-K:** 2025-07-29 18:48 UTC
* **Annual Report:** 2025-10-16 16:47 UTC
* **Shareholder Letter:** 2025-10-15 18:45 UTC

These timestamps provide information about when the document files were created.

They should not automatically be interpreted as exact dates for Microsoft's internal business processes because document timestamps can be affected by document-generation and editing workflows.

---

# Task 8 — Holehe Email Registration Check

### Commands

```bash
holehe press@microsoft.com
holehe abuse@microsoft.com
```

Holehe attempted to check the addresses against numerous online services.

### Result

The checks returned rate-limit responses rather than successful account-enumeration results.

In other words, I did **not** obtain reliable evidence that these addresses were registered on the tested platforms.

### What I learned

This was useful because it demonstrated a practical limitation of automated email-enumeration tools:

* Rate limiting can prevent enumeration.
* Public or frequently tested addresses may trigger defensive controls.
* A failed or rate-limited check should not be interpreted as proof that an account does or does not exist.

---

# Risk Analysis

The following table represents **observations from this specific OSINT exercise**, not a complete security assessment of Microsoft.

| # | Finding                                                           | Potential Security Relevance                                    |
| - | ----------------------------------------------------------------- | --------------------------------------------------------------- |
| 1 | Public Microsoft addresses appeared in breach datasets            | Increased phishing and spam exposure                            |
| 2 | Email addresses were discovered inside document hyperlinks        | Potential social-engineering exposure                           |
| 3 | Document timestamps were visible in metadata                      | Can reveal information about document creation                  |
| 4 | Hosts responding on port 80 appeared in Shodan results            | Indicates HTTP exposure that may require verification           |
| 5 | Non-standard ports appeared in Shodan results                     | May warrant further verification by an authorised security team |
| 6 | Multiple third-party infrastructure providers appeared in results | Demonstrates a distributed external attack surface              |
| 7 | Holehe checks were rate-limited                                   | Demonstrates defensive controls against automated enumeration   |

> **Important:** The findings above are not proof of vulnerabilities. They are OSINT observations that would require authorised validation before security conclusions could be made.

---

# What a Defender Could Take From This

This project showed me several defensive lessons:

* Metadata-cleaning processes should consider **embedded hyperlinks**, not only visible metadata fields.
* Public role-based email addresses should be treated as publicly exposed information.
* Organisations can use passive OSINT against their own domains to understand what information is externally visible.
* Search-engine and OSINT results should be verified before being treated as confirmed infrastructure.
* Automated enumeration tools can be affected by DNS failures, rate limiting, API restrictions, and network conditions.
* Redacting sensitive information before publishing research is an important part of responsible OSINT practice.

I am still learning, so these are observations from the perspective of a cybersecurity student rather than professional security recommendations for an organisation of Microsoft's scale.

---

# What I Learned

### 1. Passive OSINT goes deeper than I expected

Whois, Wayback Machine, Shodan, Maigret, HIBP, and ExifTool provided different pieces of information.

No single tool gave the complete picture.

### 2. Metadata can contain useful information

The ExifTool investigation was the biggest learning point for me.

Even when visible metadata is cleaned, embedded document information can still deserve investigation.

### 3. Tool failures are normal

Some tools did not work as expected because of:

* API timeouts
* DNS resolution problems
* Rate limiting
* Search-interface issues

Instead of hiding these failures, I documented them.

### 4. OSINT results require interpretation

Finding an IP address, email address, username, or historical URL does not automatically mean that it represents a vulnerability.

The result needs context and, where appropriate, authorised verification.

### 5. Ethics matters in OSINT

Finding publicly accessible information does not mean that everything should be republished.

In this project, personal email addresses discovered in metadata were redacted.

---

# Repository Structure

```text
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
```

---

# Author

**Muhammad Muhsin Khamis**

Cybersecurity Student — Bayero University Kano
Self-directed learner

LinkedIn: [Muhammad Muhsin Khamis](https://www.linkedin.com/in/muhammad-muhsin-khamis-9860b3311/)

---

**End of Report**
