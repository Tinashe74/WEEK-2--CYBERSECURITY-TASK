# Pentesting Report

## Cover Page

| Report information | Details |
|---|---|
| **Assessment** | External reconnaissance and local network discovery |
| **Pentester** | `Tinashe Victor Banda` |
| **Primary domain** | `networkwalks.com` |
| **Report date** | 14 November 2026 (date visible in the Zenmap screenshot) |
| **Prepared from** | Supplied screenshots |
| **Activity Completed** | Host Discovery, Port Scanning, Web Technonolgy Fingerprinting and WAF Detection|


> **Scope note:** This report documents the results visible in the supplied screenshots. It is not evidence of a complete penetration test, exploitation, or a full security audit. Testing should only be performed on systems for which explicit authorization has been granted.

---

## 1. Introduction.

This report summarizes reconnaissance activities shown in the Week 2 screenshots. The activities gathered publicly available domain-registration and DNS information, identified web technologies and HTTP response headers for `networkwalks.com`, checked for a web application firewall (WAF), and performed a host and port discovery scan on the local `192.168.66.0/24` subnet.

The purpose is to record the observed outputs, explain their security relevance, and identify reasonable follow-up actions. Findings are limited to what can be verified from the screenshots.

## 2. Objectives

- Review registration and nameserver information for `networkwalks.com`.
- Resolve the domain and enumerate visible DNS records.
- Identify web technologies and inspect the HTTP response.
- Check whether a WAF was fingerprinted.
- Summarize hosts and ports visible in the local subnet scan.
- Provide proportionate defensive recommendations.

## 3. Scope and Limitations

### In scope

- Public domain: `networkwalks.com`
- Local network shown in Zenmap: `192.168.66.0/24`
- Evidence: the seven screenshots in [`Screenshots/`](Screenshots/)

### Limitations

- No authorization details, test plan, or complete command logs were supplied.
- The screenshots do not establish that all tools were run at the same time.
- DNS and web observations can change; results are a point-in-time record.
- The local subnet scan is reported separately from the public-domain checks. The screenshots do not establish that the subnet is associated with the domain.
- No vulnerability exploitation, authenticated testing, application testing, or verification of remediation is shown. A WAF fingerprint does not prove that the application is secure.

## 4. Methodology and Tools

The screenshots show the following commands or tool activities:

| Activity | Tool | Purpose |
|---|---|---|
| Domain registration lookup | `whois` | Retrieve registrar, dates, status, and nameserver information |
| Web technology fingerprinting | WhatWeb | Identify visible web server and application technologies |
| DNS lookup | `nslookup` | Resolve the domain through the configured DNS resolver |
| HTTP header request | `curl -I` | Inspect response status and selected HTTP headers |
| WAF fingerprinting | `wafw00f` | Check for a recognizable WAF |
| DNS enumeration | DNSRecon | Enumerate visible DNS records |
| Local host and port discovery | Zenmap / Nmap | Identify responsive hosts and port states on the local subnet |

## 5. Results

### 5.1 WHOIS — Domain registration

The WHOIS screenshot reports:

- Domain: `NETWORKWALKS.COM`
- Registrar: GoDaddy.com, LLC
- Creation date: 6 November 2019
- Registry expiration date: 6 November 2027
- Nameservers: `NS6135.HOSTGATOR.COM` and `NS6136.HOSTGATOR.COM`
- DNSSEC: unsigned
- Several registrar status protections are listed, including transfer, update, renewal, and deletion restrictions.

**Observation:** The registration output provides useful administrative context. The screenshot reports DNSSEC as unsigned; this is a hardening opportunity to assess with the domain owner and DNS provider.

### 5.2 WhatWeb — Web technology fingerprinting

The screenshot shows the domain redirecting from HTTP to HTTPS and identifies:

- Apache web server
- WordPress `7.1.4`
- Bootstrap `7.1.4`
- jQuery `3.7.1`
- WordPress Download Manager `3.3.76`
- A `__wpdm_client` cookie marked `HttpOnly`
- A `permissions-policy` response header

**Observation:** Technology fingerprinting reveals software and version information that can help administrators prioritize patch and configuration reviews. Fingerprinting alone does not confirm that a listed component is vulnerable or exploitable.

### 5.3 Nslookup — Domain resolution

The screenshot shows `8.8.8.8` as the DNS server and resolves `networkwalks.com` to:

- `192.232.216.135`

### 5.4 Curl — HTTPS response headers

The request to `https://networkwalks.com` returned **HTTP/2 200** in the screenshot. Visible response details include:

- `server: Apache`
- `x-nginx-cache: WordPress`
- `content-type: text/html; charset=UTF-8`
- `referrer-policy: no-referrer-when-downgrade`
- `permissions-policy` with listed token-related directives
- A `__wpdm_client` cookie with `Secure` and `HttpOnly` attributes

The screenshot also shows WordPress REST API links in the response. This is not, by itself, evidence of a vulnerability.

**Observation:** Review response headers and cookie settings against the site’s intended functionality and current security baseline. The screenshot alone does not show the complete header set or the behavior of the application.

### 5.5 WAFW00F — WAF fingerprinting

WAFW00F reports that the site is behind **ModSecurity (SpiderLabs)**. The screenshot shows two requests.

**Observation:** This indicates that the tool recognized a WAF fingerprint. It does not establish that the WAF is correctly configured, that all routes are protected, or that the application is secure.

### 5.6 DNSRecon — DNS enumeration

The screenshot reports that DNSSEC is not configured and displays these records or record details:

- SOA: `ns6135.hostgator.com`
- Nameservers: `ns6135.hostgator.com` and `ns6136.hostgator.com`
- A record: `networkwalks.com` → `192.232.216.135`
- MX record: `mail.networkwalks.com`
- TXT records including an SPF policy and a Google site-verification value
- SRV records for `_autodiscover._tcp`, pointing to CPanel autodiscovery hosts
- The DNS server version strings shown by the tool

**Observation:** DNS enumeration exposes infrastructure and service-routing details. The domain owner should confirm that every published record is current and intended. The displayed SPF policy should be reviewed against authorized mail senders.

### 5.7 Zenmap / Nmap — Local subnet discovery

The Zenmap screenshot shows a regular scan of `192.168.66.0/24`. Nmap reports **256 IP addresses scanned** and **2 hosts up**:

- `192.168.66.1`: 995 TCP ports reported closed; the following ports are shown open:
  - `135/tcp` — `msrpc`
  - `139/tcp` — `netbios-ssn`
  - `445/tcp` — `microsoft-ds`
  - `902/tcp` — `iss-realsecure`
  - `912/tcp` — `apex-mesh`
- `192.168.66.254`: host up; all 1,000 scanned TCP ports are reported as filtered/no response.

The screenshot shows a VMware MAC address for `192.168.66.254`. Port labels are Nmap service-name associations and do not, on their own, prove the exact application or configuration running on a port.

**Observation:** The open ports on `192.168.66.1` should be confirmed as necessary and limited to trusted network segments. This is a local network result and is not attributed to the public domain.

## 6. Recommendations

1. **Confirm scope and ownership:** Keep public-domain and local-network testing within documented authorization and agreed scope.
2. **Review and patch web components:** Verify the WordPress core, themes, plugins, and libraries against vendor-supported releases; update only after compatibility checks and backups.
3. **Review HTTP security configuration:** Evaluate the complete response headers, cookie attributes, and HTTPS behavior. Consider a `Strict-Transport-Security` policy where HTTPS is enforced site-wide; validate deployment before enabling it.
4. **Review DNS configuration:** Confirm DNSSEC availability with the DNS provider and enable it if appropriate. Remove stale records and verify the SPF policy and mail-routing records.
5. **Validate WAF coverage:** Review ModSecurity rules, logging, exclusions, and protected routes. Treat WAF as an additional control, not a substitute for application security.
6. **Reduce local network exposure:** Confirm that the services on `192.168.66.1` are required, restrict access with host or network firewalls, and disable unused services.
7. **Retest and retain evidence:** After approved changes, repeat the relevant checks and compare results with this point-in-time report.

## 7. Conclusion

The screenshots document a useful baseline of domain, DNS, web-server, WAF, and local-network reconnaissance. They show the domain resolving to `192.232.216.135`, Apache and WordPress-related technologies, a ModSecurity fingerprint, DNS records with DNSSEC reported as unsigned, and five open TCP ports on the separate local host `192.168.66.1`.

These observations support configuration review and defensive follow-up, but they do not establish that a vulnerability exists or that the systems have passed a comprehensive penetration test. The next steps are to confirm authorization, validate the findings with the system owner, prioritize necessary hardening, and retest after changes.

## 8. Evidence

| Evidence | Screenshot |
|---|---|
| Zenmap scan of `192.168.66.0/24` | [Screenshot (356).png](Screenshots/Screenshot%20%28356%29.png) |
| WHOIS output | [W2-PM1_Task1_WHOIS_Output.png](Screenshots/W2-PM1_Task1_WHOIS_Output.png) |
| WhatWeb output | [W2-PM1_Task2_WhatWeb_Output.png](Screenshots/W2-PM1_Task2_WhatWeb_Output.png) |
| Nslookup output | [W2-PM1_Task3_Nslookup_Output.png](Screenshots/W2-PM1_Task3_Nslookup_Output.png) |
| Curl output | [W2-PM1_Task4_Curl_Output.png](Screenshots/W2-PM1_Task4_Curl_Output.png) |
| WAFW00F output | [W2-PM1_Task5_Wafw00f_Output..png](Screenshots/W2-PM1_Task5_Wafw00f_Output..png) |
| DNSRecon output | [W2-PM1_Task6_DNSRecon_Output..png](Screenshots/W2-PM1_Task6_DNSRecon_Output..png) |
