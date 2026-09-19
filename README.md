# Networkwalks Cybersecurity Internship — Week 2 PM1
## Footprinting & Reconnaissance with Multiple Kali Tools

**Target:** `networkwalks.com`  
**Module:** W2-PM1 — Week 2 Project Module 1  
**Evidence date:** 19 September 2026  
**Environment:** Kali Linux  
**Purpose:** Authorized reconnaissance / information gathering for the internship assignment.

---

## 1. Objective

This practical exercise demonstrates how several Kali Linux reconnaissance tools can be used to build a basic profile of a web domain.

The six tools are performed in the assignment's order:

1. **WHOIS** — domain registration information
2. **WhatWeb** — web technology fingerprinting
3. **nslookup** — DNS lookup
4. **curl -I** — HTTP response headers
5. **WAFW00F** — Web Application Firewall fingerprinting
6. **DNSRecon** — DNS enumeration

> **Scope and safety:** This report documents reconnaissance only. No exploitation, brute-force/password attacks, denial-of-service, privilege escalation, or unauthorized access was performed.

---

## 2. Tool Sequence

| # | Tool | Command used | Main concept |
|---|---|---|---|
| 1 | WHOIS | `whois networkwalks.com` | Domain registration / ownership metadata |
| 2 | WhatWeb | `whatweb networkwalks.com` | Web technology fingerprinting |
| 3 | nslookup | `nslookup networkwalks.com` | DNS resolution |
| 4 | curl | `curl -I https://networkwalks.com` | HTTP response headers |
| 5 | WAFW00F | `wafw00f networkwalks.com` | WAF detection |
| 6 | DNSRecon | `dnsrecon -d networkwalks.com` | DNS enumeration |

---

# 3. Results and Evidence

## 3.1 WHOIS

**Command**
```bash
whois networkwalks.com
```

**What the screenshot shows**

The WHOIS result identifies the domain as `NETWORKWALKS.COM` and shows:

- Registrar: **GoDaddy.com, LLC**
- Creation date: **2019-11-06**
- Registry expiry date shown: **2027-11-06**
- Name servers:
  - `NS6135.HOSTGATOR.COM`
  - `NS6136.HOSTGATOR.COM`
- Domain status values including client transfer/update/renewal/delete protections
- DNSSEC: **unsigned**

**Concept:** WHOIS provides registration-related information about a domain. It is useful during reconnaissance because it can reveal registrar and name-server information without attempting to access the target.

**Evidence:** [`01-whois.png`](01-whois.png)

---

## 3.2 WhatWeb

**Command**
```bash
whatweb networkwalks.com
```

**What the screenshot shows**

WhatWeb first reports an HTTP **301 Moved Permanently** response and then identifies the HTTPS site with a **200 OK** response.

The fingerprinting output includes indicators such as:

- Apache web server
- WordPress
- WordPress Download Manager
- jQuery 3.7.1
- Bootstrap-related technology
- HTML5
- Open Graph Protocol
- Public email shown by the scanner: `info@networkwalks.com`
- IP address shown by the scanner: `192.232.216.135`

**Concept:** WhatWeb is a web technology fingerprinting tool. It attempts to identify software, frameworks, server technologies, plugins, and other characteristics visible from the web application.

**Evidence:** [`02-whatweb.png`](02-whatweb.png)

---

## 3.3 nslookup

**Command**
```bash
nslookup networkwalks.com
```

**What the screenshot shows**

The DNS lookup reports:

- DNS server used by the VM: `192.168.215.247`
- A non-authoritative answer
- `networkwalks.com` resolves to:
  - `192.232.216.135`

**Concept:** DNS translates human-readable domain names into IP addresses. `nslookup` is a simple way to query DNS and verify the address associated with a domain.

**Evidence:** [`03-nslookup.png`](03-nslookup.png)

---

## 3.4 curl -I

**Command**
```bash
curl -I https://networkwalks.com
```

**What the screenshot shows**

The HTTP response contains:

- Status: **HTTP/2 200**
- `server: Apache`
- `content-type: text/html; charset=UTF-8`
- WordPress-related indicators
- A `link` header exposing a WordPress REST API reference (`/wp-json/`)
- A WordPress Download Manager-related cookie (`_wpdm_client`)
- Additional security/cache/policy headers

**Concept:** `curl -I` requests only the HTTP response headers rather than downloading the full page body. Headers can reveal useful information about server software, content type, cookies, redirects, policies, and application behavior.

**Evidence:** [`04-curl.png`](04-curl.png)

---

## 3.5 WAFW00F

**Command**
```bash
wafw00f networkwalks.com
```

**What the screenshot shows**

WAFW00F reports:

> The site is behind **ModSecurity (SpiderLabs) WAF**.

It also shows that the tool made **2 requests** during its check.

**Concept:** A WAF (Web Application Firewall) is a security control placed in front of a web application. WAFW00F attempts to identify whether a WAF is present and, when possible, identify its technology.

**Evidence:** [`05-wafw00f.png`](05-wafw00f.png)

---

## 3.6 DNSRecon

**Command**
```bash
dnsrecon -d networkwalks.com
```

**What the screenshot shows**

DNSRecon completed an enumeration and reported **8 records found**.

The screenshot includes:

- SOA record associated with `ns6135.hostgator.com`
- NS records:
  - `ns6135.hostgator.com`
  - `ns6136.hostgator.com`
- MX record:
  - `mail.networkwalks.com`
  - `192.232.216.135`
- A record:
  - `networkwalks.com` → `192.232.216.135`
- TXT records including an SPF-related record
- A Google-site-verification TXT value
- Multiple SRV/autodiscover records pointing to a cPanel email-discovery service
- A message indicating no answer for the DNSSEC query

**Concept:** DNSRecon automates DNS enumeration. It can identify DNS records such as SOA, NS, MX, A, TXT, and SRV records. This helps a security analyst understand the domain's DNS structure.

**Evidence:** [`06-dnsrecon.png`](06-dnsrecon.png)

---

# 4. Combined Reconnaissance Picture

The six tools reveal different layers of information:

```text
                         networkwalks.com
                                |
       +------------------------+------------------------+
       |                        |                        |
     WHOIS                    DNS                    Web App
       |                        |                        |
 Registrar                A / NS / MX / TXT          WhatWeb
 Name servers                  / SRV                   |
 Creation date                    |                WordPress
 Expiry date                  DNSRecon              Apache
                                                        |
                                                   curl -I
                                                        |
                                                  HTTP headers
                                                        |
                                                   WAFW00F
                                                        |
                                                 ModSecurity WAF
```

The main lesson is that reconnaissance is cumulative: one tool provides one piece of information, while several tools together produce a broader technical profile.

---

# 5. Key Findings Summary

| Area | Finding from the screenshots |
|---|---|
| Domain | `networkwalks.com` |
| Registrar | GoDaddy.com, LLC |
| Name servers | `ns6135.hostgator.com`, `ns6136.hostgator.com` |
| Resolved IP | `192.232.216.135` |
| Web server | Apache |
| Application indicators | WordPress / WordPress Download Manager |
| JavaScript/library indicator | jQuery 3.7.1 |
| WAF | ModSecurity (SpiderLabs) |
| DNS records observed | SOA, NS, MX, A, TXT, SRV |
| DNSRecon records found | 8 |
| DNSSEC query | No answer shown |
| HTTP status observed by curl | HTTP/2 200 |

---

# 6. What I Learned

### WHOIS
I learned how to collect domain registration and name-server information.

### WhatWeb
I learned how to fingerprint web technologies from externally observable responses.

### nslookup
I learned how DNS maps a domain name to an IP address.

### curl
I learned how HTTP response headers can expose useful technical information without downloading the entire webpage.

### WAFW00F
I learned how to identify the presence and apparent type of a Web Application Firewall.

### DNSRecon
I learned how to enumerate different DNS record types and understand the DNS structure of a domain.

---

# 7. Security Relevance

Footprinting is normally performed before deeper security testing because it helps an analyst understand the target's exposed attack surface.

For example:

- DNS information can identify infrastructure.
- Technology fingerprinting can identify software that should be kept patched.
- HTTP headers can reveal application/server details.
- WAF detection shows that a web application has an additional security layer.
- DNS records can reveal mail and service infrastructure.

**Important:** A finding from reconnaissance is not automatically a vulnerability. Reconnaissance identifies information and potential areas for further authorized review; additional testing would be required to establish whether a security weakness actually exists.

---

# 8. Evidence Files

The screenshots are intentionally numbered in the same order as the assignment:

1. `01-whois.png`
2. `02-whatweb.png`
3. `03-nslookup.png`
4. `04-curl.png`
5. `05-wafw00f.png`
6. `06-dnsrecon.png`

---

# 9. Conclusion

This W2-PM1 practical demonstrated six reconnaissance techniques against the assigned domain. Together, the results provided information about domain registration, DNS infrastructure, web technologies, HTTP behavior, WAF protection, and DNS records.

The exercise demonstrates an important cybersecurity principle:

> **Before attempting to secure or test a system, first understand what information the system exposes.**

