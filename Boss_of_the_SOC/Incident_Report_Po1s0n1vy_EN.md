# Po1s0n1vy APT — Incident Report

**Incident Investigation Report**

*Website Defacement of imreallynotbatman.com — Po1s0n1vy APT Group*

**Analyst:** Alex (SOC Analyst, training project)
**Report Date:** July 2026
**Classification:** Training project / portfolio piece
**Data Source:** Splunk BOTS v1 (Boss of the SOC), Scenario 1

## 1. Executive Summary

On August 10, 2016, the website imreallynotbatman.com, hosted within Wayne Enterprises' IP address space, was defaced. Users reported seeing an image referencing "P01s0n1vy" — a known APT group with a history of targeting Wayne Enterprises.

This investigation, conducted using the Splunk SIEM platform alongside open-source intelligence (OSINT) and VirusTotal, reconstructs the full attack chain according to the Lockheed Martin Cyber Kill Chain model: from vulnerability scanning and reconnaissance through compromise of the Joomla CMS, web shell upload, executable malware installation, and the final defacement.

The attacking infrastructure was attributed to the Po1s0n1vy group based on WHOIS record analysis, related domains, and IP addresses pre-staged for future attacks against Wayne Enterprises infrastructure.

## 2. Scope and Methodology

The investigation was conducted using the Boss of the SOC v1 (BOTSv1) dataset, which includes web server logs (stream:http), Windows Sysmon and Security event logs, and network logs (Suricata/fgt_utm). Splunk Enterprise (self-hosted via Docker) served as the primary analysis platform.

- Splunk Enterprise — log search and correlation (SPL queries)
- VirusTotal — reputation analysis of IPs, domains, and files; malware attribution
- WHOIS / Whoxy — historical domain registration data
- CyberChef — data decoding (Hex → text)

## 3. Attack Reconstruction — Lockheed Martin Cyber Kill Chain

The timeline below is organized by the classic Cyber Kill Chain phases, highlighting the key indicators identified during analysis.

### 3.1 Reconnaissance

The attack began with automated web vulnerability scanning of imreallynotbatman.com. The scan originated from IP address 40.80.148.42, which sent over 17,500 HTTP requests to the web server (192.168.250.70).

Analysis of the User-Agent and request payloads (SQL injection and XSS test strings carrying the signature "acunetix_wvs_security_test") confirmed the tool in use: the commercial vulnerability scanner Acunetix WVS.

URL path structure (/joomla/administrator/, /joomla/templates/protostar/) identified the site's CMS as Joomla.

### 3.2 Weaponization & Delivery

The defacement image, poisonivy-is-coming-for-you-batman.jpeg, was hosted on a separate attacker-controlled staging server and pulled by the victim's web server via an outbound request. The staging server used a dynamic DNS service: prankglassinebracket.jumpingcrab.com, resolving to IP 23.22.63.114 on port 1337.

OSINT analysis revealed that this same IP (23.22.63.114) had been pre-staged against the domain waynecorinc.com — evidence that Po1s0n1vy prepares its infrastructure in advance, indicating further planned attacks against Wayne Enterprises.

Historical WHOIS analysis of waynecorinc.com identified a probable email address associated with the threat actor: lillian.rose@po1s0n1vy.com.

### 3.3 Exploitation

The Joomla administrator panel was compromised via password brute forcing, originating from the same IP address, 23.22.63.114. A total of 413 login attempts (412 unique passwords) were logged within a few minutes, all targeting the "admin" account.

The first password attempted was "123456789"; among the attempted values was "yellow" — a Coldplay song title. The average password length across the brute-force attempt was 6 characters.

The successful password was "batman". Only 92.17 seconds elapsed between the moment this password first appeared in the automated brute-force sequence and the subsequent successful login — suggesting a semi-automated attack with minimal operator involvement at the access-confirmation stage.

### 3.4 Installation

After gaining administrative access, the attacker installed a third-party Joomla extension — the eXtplorer file manager (eXtplorer_2.1.5.zip), a legitimate tool repurposed to gain direct access to the server's file system.

This was used to upload the web shell agent.php, and subsequently — likely through it — the executable 3791.exe (path: C:\inetpub\wwwroot\joomla\3791.exe). The file's hash (MD5: AAE3F5A29935E6ABCC2C2754D12A9AF0) was captured by Sysmon at process creation.

### 3.5 Command & Control

Immediately after 3791.exe was executed, the server established an outbound connection to ec2-23-22-63-114.compute-1.amazonaws.com (the same attributed attacker IP) over port 3791 — the likely C2 channel for the malware.

### 3.6 Actions on Objectives

The attack's ultimate objective was the defacement of the site's homepage — a public demonstration of compromise intended to cause reputational damage to Wayne Enterprises, consistent with Po1s0n1vy's known modus operandi.

Additionally, as part of the group's standard TTPs, this investigation identified their use of custom malware for spear-phishing: a sample named MirandaTateScreensaver.scr.exe (SHA256: 9709473ab351387aab9e816eff3910b9f28a7a70202e250ed46dba8f820f34a8), attributed by multiple AV vendors to the Zupdax/SanwaiCrypt family.

## 4. Indicators of Compromise (IOC)

The table below summarizes all indicators identified during this investigation.

| Type | Value | Role in Attack |
| --- | --- | --- |
| IPv4 | 40.80.148.42 | Vulnerability scanning (Acunetix) |
| IPv4 | 23.22.63.114 | Brute force, C2, defacement file hosting, pre-staged infrastructure |
| Domain | prankglassinebracket.jumpingcrab.com | Defacement file staging server (Dynamic DNS) |
| Domain | waynecorinc.com | Pre-staged domain for future attack |
| Domain | po1s0n1vy.com | Group attribution infrastructure |
| Email | lillian.rose@po1s0n1vy.com | OSINT attribution |
| File | poisonivy-is-coming-for-you-batman.jpeg | Defacement file |
| File | agent.php | Web shell |
| File (MD5) | 3791.exe — AAE3F5A29935E6ABCC2C2754D12A9AF0 | Malicious executable |
| File (SHA256) | MirandaTateScreensaver.scr.exe — 9709473ab351387aab9e816eff3910b9f28a7a70202e250ed46dba8f820f34a8 | Spear-phishing malware (Zupdax family) |

## 5. Appendix: Full Investigation Question & Answer Log

The appendix lists all 17 checkpoint questions from the BOTS v1 dataset (Scenario 1), along with the answers obtained through hands-on analysis in Splunk.

| # | Question | Answer |
| --- | --- | --- |
| 1 | IPv4 address of the Po1s0n1vy scanner | 40.80.148.42 |
| 2 | Company that created the vulnerability scanner | Acunetix |
| 3 | CMS used by imreallynotbatman.com | Joomla |
| 4 | Name of the defacement file | poisonivy-is-coming-for-you-batman.jpeg |
| 5 | Dynamic DNS FQDN associated with the attack | prankglassinebracket.jumpingcrab.com |
| 6 | IP tied to pre-staged domains | 23.22.63.114 |
| 7 | Brute-force attack IP | 23.22.63.114 |
| 8 | Name of the uploaded executable | 3791.exe |
| 9 | MD5 hash of the uploaded file | AAE3F5A29935E6ABCC2C2754D12A9AF0 |
| 10 | SHA256 of spear-phishing malware | 9709473ab351387aab9e816eff3910b9f28a7a70202e250ed46dba8f820f34a8 |
| 11 | Special hex code tied to the malware | 53 74 65 76 65 20 42 72 61 6e 74 27 73 20 42 65 61 72 64 ... (developer easter egg) |
| 12 | First password used in the brute force | 123456789 |
| 13 | Coldplay song password (6 letters) | yellow |
| 14 | Correct administrator password | batman |
| 15 | Average password length in the brute force | 6 characters |
| 16 | Seconds between password match and login | 92.17 sec. |
| 17 | Unique passwords attempted | 412 |
| OSINT | Email associated with Po1s0n1vy | lillian.rose@po1s0n1vy.com |

## 6. Recommendations

- Implement failed-login rate limiting / account lockout for CMS administrative panels.
- Restrict access to /administrator/ by IP allowlist or VPN, minimizing public internet exposure.
- Keep Joomla core and all installed extensions patched; audit installed plugins (particularly file-manager extensions such as eXtplorer).
- Deploy monitoring for anomalous process creation on web servers (Sysmon + SIEM correlation) to detect web shell and executable uploads early.
- Add the IOCs identified in this investigation to blocklists (IPs, domains, file hashes).
- Alert on outbound connections from web servers to unusual ports (in this case, port 3791).

## 7. Conclusion

This investigation confirmed the compromise of imreallynotbatman.com by the Po1s0n1vy group and fully reconstructed the attack chain from initial reconnaissance through final defacement. The indicators of compromise and tactics identified here can be used to harden the remainder of Wayne Enterprises' infrastructure and to detect future attempts by this group at an earlier stage.
