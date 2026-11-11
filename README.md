# Penetration Testing Report (Week 2)

**Footprinting & Network Scanning Phases**

**W2-PM-FINAL | Cybersecurity | Networkwalks**

---

| Field | Detail |
| --- | --- |
| Pentester Name | Kolawole Asmot Omowumi |
| Program/Batch | [B084] - Networkwalks |
| Date | 10 November 2026 |
| Modules completed | W2-PM1 (Multiple Kali Tools), W2-PM5 (Zenmap Scanning) |
| Client/Target | 1. Networkwalks (secured written permission already) 2. My own local LAN network |
| Phases covered | Phase 1: Reconnaissance & Footprinting. Phase 2: Scanning & Network Discovery |

---

## 1. Liability Disclaimer

I performed these activities only on systems and devices where I had secured written permission, or on devices and systems that I own myself. All materials are for education and research only. Do not use anything from here to break the law. Unauthorised access is a crime in most countries, even when nothing is damaged.

---

## 2. Introduction

This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools (W2-PM1) and scanning my own local network with Zenmap (W2-PM5). Together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my internship at Networkwalks.

Footprinting was done in Kali Linux. Scanning was done on a Windows PC with Zenmap installed. No exploitation was attempted.

---

## 3. Tools Used

| Tool | Purpose |
| --- | --- |
| Kali Linux & Windows | Platforms used for the activities |
| WHOIS | Retrieve public domain registration information |
| WhatWeb | Identify website technologies |
| nslookup | Resolve domain names and read DNS records |
| curl -I | Inspect HTTP response headers |
| wafw00f | Detect a Web Application Firewall |
| dnsrecon | Enumerate DNS records (NS, MX, SPF, TXT, SRV) |
| ipconfig | Find local IP address, subnet and gateway on Windows |
| Zenmap (Nmap) | Discover live hosts and generate a topology |

---

## 4. Activities Performed

### 4.1 Footprinting and Reconnaissance (W2-PM1)

All six tools were used against **networkwalks.com** only.

**WHOIS** (`whois networkwalks.com`): The domain is registered with GoDaddy.com, LLC, created 6 November 2019, last updated 12 November 2025 and expires 6 November 2027. The name servers are NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM, and DNSSEC is shown as unsigned. Client lock statuses (delete, renew, transfer, update prohibited) are enabled.
<img width="1920" height="890" alt="whois networkwals" src="https://github.com/user-attachments/assets/f9dcb387-077f-486b-8acb-23df942fe655" />

**nslookup** (`nslookup -type=any networkwalks.com`): Queried through Google's resolver (8.8.8.8). The domain resolves to **192.232.216.135**. Results show the HostGator name servers, the mail server `mail.networkwalks.com`, a Google site-verification TXT record and an SPF record ending in `~all`.

<img width="1920" height="890" alt="nslookup networkwalks" src="https://github.com/user-attachments/assets/1ad784b3-6913-469c-a1da-5c3ba0b446e4" />

**WhatWeb** (`whatweb networkwalks.com`): HTTP redirects to HTTPS (301). The site runs on Apache and WordPress 7.1.4 with the WordPress Download Manager 3.3.76 plugin, plus jQuery 3.7.1, Bootstrap and Google Tag Manager. The email `info@networkwalks.com` appears on the page.

<img width="1920" height="890" alt="whatweb" src="https://github.com/user-attachments/assets/6d2b6797-47ca-43be-832d-e89c756e50cd" />

**curl** (`curl -I https://networkwalks.com`): Response HTTP/2 200 with `Server: Apache`. Headers also expose the `/wp-json/` REST API link and cache headers (`x-nginx-cache`, `x-endurance-cache-level`). A Permissions-Policy header is present and the session cookie uses the Secure and HttpOnly flags. I did not observe Strict-Transport-Security, X-Frame-Options, X-Content-Type-Options or Content-Security-Policy headers.

<img width="1920" height="890" alt="curl" src="https://github.com/user-attachments/assets/72655108-09ef-4d69-b345-5d3e5902be0b" />

**wafw00f** (`wafw00f https://networkwalks.com`): The site is behind a **ModSecurity (SpiderLabs)** WAF, detected with 2 requests.

<img width="1920" height="890" alt="wafwoof" src="https://github.com/user-attachments/assets/9b9668bf-ac8d-4749-82c5-73813432aa7e" />


**DNSRecon** (`dnsrecon -d networkwalks.com`): Reported that **DNSSEC is not configured**. It returned the two name servers (`ns6136.hostgator.com` at 192.232.216.131 and `ns6135.hostgator.com` at 50.87.144.87), both reporting **BIND version 9.16.23-RH**. It also returned the mail server (`mail.networkwalks.com`, 192.232.216.135), the A record (192.232.216.135), the SPF and Google site-verification TXT records, and eight `_autodiscover._tcp` SRV records pointing to `cpanelemaildiscovery.cpanel.net` on port 443. One query to Google's DNS timed out while resolving the SOA record, but the remaining results completed normally.

<!-- drag the DNSRecon screenshot here -->

**Problem encountered:** DNSRecon first crashed with a Python `TypeError` caused by an outdated DNS library on my older Kali image. An attempt to upgrade the system packages failed part-way, so I restored my clean VM snapshot and upgraded only the Python `dnspython` library for my own user, after which DNSRecon ran successfully.

### 4.2 Network Scanning with Zenmap (W2-PM5)

I used zenmap to perform network discovery on my local network. The practical required me to identify my local IP address and subnet, discover life host, identify their IP and MAC addresses, and generate a network topology.

In Zenmap I used the **Ping scan** profile. It covered 256 IP addresses in 3.53 seconds and found **2 live hosts**:




I then opened the **Topology** tab in Zenmap, enabled the legend and saved the topology as a PDF.

<img width="720" height="388" alt="WhatsApp Image 2026-11-11 at 11 39 37 AM" src="https://github.com/user-attachments/assets/89a7c31a-3b01-4e6f-a513-5271e05c7ce7" />

---

## 5. Risk Analysis / Impact

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
| --- | --- | --- | --- | --- |
| 1 | Web technology and version information exposed | WhatWeb identified WordPress 7.1.4, WordPress Download Manager 3.3.76, jQuery 3.7.1 and a contact email on the page | Version information may help an attacker look for known weaknesses; the email may attract phishing or spam | 🟠 Medium |
| 2 | Server IP address and hosting identifiable | nslookup and WhatWeb resolved the domain to 192.232.216.135; name servers hosted at hostgator.com | Reveals the network location and hosting provider of the web service | 🟢 Low |
| 3 | HTTP technical information exposed; security headers not observed | curl showed `Server: Apache`, the `/wp-json/` link and cache headers; no HSTS, X-Frame-Options, X-Content-Type-Options or CSP observed | May assist fingerprinting; missing headers leave browsers without some extra protections | 🟢 Low |
| 4 | WAF technology identifiable | wafw00f identified ModSecurity (SpiderLabs) after two requests | Reveals part of the security architecture; the WAF is also a protective control | 🟢 Low |
| 5 | DNS and email configuration observations | DNSRecon and nslookup showed an SPF record ending in `~all` (soft fail); DNSRecon and WHOIS showed DNSSEC is not configured; both name servers reported BIND version 9.16.23-RH | Soft-fail SPF and unsigned DNS can make email spoofing and DNS tampering easier; exposed DNS software versions help fingerprinting | 🟠 Medium |
| 6 | Live hosts visible on the local network | Zenmap found 2 live hosts on 192.168.237.0/24: the gateway and my own PC | Only expected devices were found; unknown devices should always be investigated | 🟢 Low |

**Important note:** These exercises were information gathering and host discovery only. No exploitation or vulnerability validation was performed. The presence of a software version, IP address or DNS record does not by itself mean a system is vulnerable. Further authorized testing would be needed to confirm any vulnerability.

---

## 6. Recommendations

1. **Reduce technology and version exposure.** Limit publicly visible WordPress, plugin and REST API information such as generator tags and the `/wp-json/` link header.
2. **Keep software updated.** Update WordPress, plugins (including WordPress Download Manager) and JavaScript libraries regularly.
3. **Add missing HTTP security headers.** Consider Strict-Transport-Security, X-Frame-Options, X-Content-Type-Options and a Content-Security-Policy, and review the Referrer-Policy.
4. **Strengthen DNS and email security.** Once all legitimate mail sources are confirmed, tighten SPF from `~all` to `-all`, add a DMARC record and consider DNSSEC if the DNS provider supports it.
5. **Keep the WAF enabled and tuned.** Keep ModSecurity rules updated.
6. **Perform regular internal network discovery.** Know which devices are connected to your network.
7. **Investigate unknown devices.** Verify any unexpected device, including ones with an unidentified MAC vendor.
8. **Maintain network documentation.** Keep topology and device information up to date.
9. **Test only with authorization.** Reconnaissance and scanning should only be done where proper authorization exists.

---

## 7. Conclusion

In the footprinting activity, I used six tools in Kali Linux to collect information about the target domain. I learned how WHOIS provides domain information, WhatWeb identifies web technologies, nslookup resolves domain names, curl inspects HTTP headers, wafw00f identifies a WAF, and DNSRecon enumerates DNS records.

In the scanning activity, I used Zenmap to identify my local network configuration, discover active hosts, collect IP and MAC address information and create a network topology.

Information gathering is an important part of cybersecurity: even before trying to exploit anything, a professional can learn a lot from public information and network responses. I also learned that findings should be documented clearly, and that reconnaissance and scanning must always stay within an authorized scope.

---

## 👤 Author

**Kolawole Asmot Omowumi**

Cybersecurity Intern, [B084]

LinkedIn: <https://www.linkedin.com/in/kolawole-omolola-8490033a7>

**Program:** Cybersecurity program at Networkwalks | **Week:** 02
