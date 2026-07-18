# Incident Report: SQL Injection Enumeration Attempt Against DVWA

Framework: CERT Société Générale IRM-3

---

## 1. Time of Activity

July 19, 2026, 01:00:16 – 01:00:48 (UTC+3). Five malicious HTTP GET requests observed within a 32-second window, all originating from the same source IP.

---

## 2. Affected Entities

| Field | Value |
|---|---|
| Target host | Debian victim VM — 192.168.1.196 |
| Affected application | DVWA (Damn Vulnerable Web Application), endpoint `/dvwa/vulnerabilities/sqli/` |
| Source IP | 192.168.1.133 |
| Web server | Apache2, log source: `/var/log/apache2/access.log` |
| Detection platform | Wazuh manager — 192.168.1.153 |
| Wazuh agent | agent.id 001 (debian) |

---

## 3. Summary

A source host (192.168.1.133) submitted a sequence of five HTTP GET requests to the DVWA SQL Injection page, each containing a UNION-based SQL injection payload targeting the `id` parameter. The payload sequence followed a classic database enumeration pattern:

1. `1' UNION SELECT null,null -- -` — baseline column-count probe
2. `1' UNION SELECT user,password FROM users -- -` — credential extraction attempt
3. `1' UNION SELECT table_name,null FROM information_schema.tables -- -` — table enumeration
4. `1' UNION SELECT column_name,null FROM information_schema.columns -- -` — column enumeration
5. `1' UNION SELECT database(),null -- -` — database fingerprinting

All five requests returned HTTP 200 (successful execution, data returned to the attacker), confirming the injection was not blocked or sanitized by the application.

---

## 4. Detection Logic

Detection was implemented as a two-level Wazuh rule chain, consistent with the approach used in the earlier SSH brute-force scenario (atomic signature → frequency-based correlation):

**Level 1 — Atomic signature (built-in Wazuh rules):**
- Rule **31103** ("SQL injection attempt", level 7) matches known SQLi keyword patterns (`union+`, `select+`, `null,null`, etc.) in the decoded Apache URL field.
- Rule **31106** ("A web attack returned code 200 (success)", level 6) is a child of 31103/31104/31105, firing when the same request also returned HTTP 200 — indicating a *successful* (not just attempted) attack.
- MITRE ATT&CK: **T1190** (Exploit Public-Facing Application).

**Level 2 — Custom correlation rule (local_rules.xml):**
```xml
<group name="sqli,web,attack,">
  <rule id="100101" level="12" frequency="5" timeframe="120">
    <if_matched_sid>31106</if_matched_sid>
    <same_source_ip />
    <description>Multiple SQL injection attempts from the same source - possible enumeration</description>
    <mitre>
      <id>T1190</id>
      <id>T1590</id>
    </mitre>
  </rule>
</group>
```
This rule escalates to level 12 when 5 or more instances of rule 31106 fire from the same source IP within a 120-second window — capturing the enumeration behavior rather than a single injection attempt.

MITRE ATT&CK: **T1190** (Exploit Public-Facing Application) for the initial access vector, **T1590** (Gather Victim Network Information) for the enumeration/reconnaissance phase.

**Engineering note:** an initial atomic rule (100100) was written to match `UNION SELECT` directly via a custom regex on the `<url>` field. It failed to fire reliably in production despite passing isolated syntax checks — likely due to interaction with Wazuh's built-in web-access rule chain, which already resolves the event via 31103/31106 before a custom sibling rule gets evaluated as the "final" alert. The correlation rule was therefore rebuilt to chain off the built-in 31106 rather than a custom atomic rule. A separate troubleshooting finding: Wazuh's `if_matched_sid` / frequency tracker appears to register only the *deepest* (most specific) rule that fires in a decoding chain, not every intermediate parent — so correlating against 31103 (the immediate SQLi signature) did not accumulate frequency correctly when 31106 (its more specific child) was also firing; switching the correlation target to 31106 resolved this.

---

## 5. Reason for Classification

Classified as **SQL Injection — Enumeration Phase (High severity)**. Justification:
- Payload structure (UNION-based, progressing from column-count probing to `information_schema` enumeration) is a textbook manual SQLi reconnaissance sequence, not incidental/benign traffic.
- All requests returned HTTP 200 with response bodies of increasing size (1843 → 2010 → 2591 → 6239 → 1850 bytes), consistent with the application returning structured data (table/column names) to the attacker.
- Requests targeted `information_schema`, indicating deliberate schema-mapping ahead of a targeted data-extraction attempt (e.g., pulling the `users` table).

---

## 6. Reason for Escalation

Escalated to level 12 (high) rather than left at the atomic-signature level (6-7) because:
- The correlation rule confirms *sustained, automated-style* behavior from a single source rather than a one-off probe — this is materially more dangerous and indicates an active, ongoing attack rather than background scanner noise.
- The enumeration sequence had already progressed to `information_schema.columns` by the time of the alert, meaning the attacker was one step away from a full credential-extraction query — escalation at this point allows intervention before actual data (e.g., the `users` table password hashes) is exfiltrated.

---

## 7. Remediation

- **Immediate:** Block/rate-limit source IP 192.168.1.133 at the firewall or WAF layer.
- **Application-level:** Replace raw SQL string concatenation in DVWA-equivalent production code with parameterized queries / prepared statements (DVWA is intentionally vulnerable — in a real deployment this is the primary fix).
- **Defense-in-depth:** Deploy a WAF rule set (e.g., OWASP CRS) in front of the application to catch UNION-based injection patterns before they reach the database layer.
- **Monitoring:** Extend the correlation rule's `frequency`/`timeframe` tuning based on further testing to reduce false-negative risk from slower, more evasive enumeration attempts (e.g., an attacker pacing requests beyond the 120s window).

---

## 8. IOCs

| Type | Value |
|---|---|
| Source IP | 192.168.1.133 |
| Target endpoint | `/dvwa/vulnerabilities/sqli/` |
| Payload pattern | `UNION+SELECT` with `information_schema` references |
| User-Agent | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36` |
| Wazuh rule IDs | 31103, 31106 (atomic), 100101 (correlation) |

---

*Part of the [cybersecurity-home-lab](https://github.com/azlobin88/cybersecurity-home-lab) portfolio project.*
