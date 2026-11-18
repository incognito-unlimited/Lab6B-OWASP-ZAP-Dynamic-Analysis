# Lab 6B — OWASP ZAP Dynamic Analysis

**Student:** Aditey Mehra  
**Roll number:** 2301730266  
**Programme:** B.Tech CSE (AI & ML), Section D, Semester 7  
**University:** K.R. Mangalam University  
**Subject:** Secure Coding and Vulnerabilities  

## Target and isolation

- **Target origin:** `http://127.0.0.1:8081`
- **Deployment:** existing Docker container `lab6a-dvwa` (`sagikazarmark/dvwa:latest`), DVWA v1.10 *Development*
- **Published port:** `127.0.0.1:8081 -> 80/tcp` (loopback only)
- **Listener:** `TCP 127.0.0.1:8081 ... LISTENING`
- **Host non-loopback reachability:** `reachable=False`
- **Firewall:** no inbound rule explicitly scoped to port `8081`
- **Limitation:** no separate-device or external-network reachability test was performed. Isolation evidence is in `notes/network-isolation.txt`.

## ZAP configuration

- **Version:** OWASP ZAP 2.17.0 (portable Crossplatform distribution)
- **Dedicated session:** `lab6b-zap-auth.session` (kept outside the repository)
- **Context:** `Lab6B-DVWA` (`contextId=2`), in scope; Default Context out of scope
- **Include:** `^http://127\.0\.0\.1:8081(?:/.*)?$`
- **Exclude:** `logout.php`, `setup.php`, `security.php`, and all non-target origins
- **Scan mode:** authenticated as `lab6b-scan-user` (`userId=0`) with DVWA `security=low`
- **Authentication note:** ZAP re-login reset the DVWA security cookie to `impossible`; the verified low-security session cookie was seeded into the ZAP user and the logged-in indicator set to `.*` to prevent re-login. Actual responses were checked and contained `You have logged in as 'admin'` and `Security Level: low`. This configuration is intentional and documented.

## Actual scan statistics

### Traditional Spider

| Metric | Value |
| --- | --- |
| Scan ID | 4 |
| Time | 2026-11-18 07:50:49 to 07:50:58 IST (9.5 s) |
| Final status | 100 (complete) |
| Discovered URLs | 171 |
| Messages in scan window | 165 |
| Settings | maxDepth 3, maxChildren 20, recurse true |
| Error responses | 8 (6×404, 2×500) |

### Active Scan

| Metric | Value |
| --- | --- |
| Scan ID | 0 |
| Time | 2026-11-18 08:10:35 to 08:23:18 IST (722.9 s) |
| Final status | 26% — stopped at the 12-minute time cap; not a completed 100% scan |
| Scanner-reported requests | 5,788 |
| Scanner-reported alerts | 28 (24 new) |
| Policy | Default Policy; maxScanDuration 10 min; maxRuleDuration 2 min |

Docker Desktop stopped repeatedly during the lab; a local watchdog kept the DVWA container running. No external host was ever scanned.

## Alert summary

| Risk | Count |
| --- | --- |
| High | 25 |
| Medium | 190 |
| Low | 237 |
| Informational | 67 |
| **Total** | **519** |

Confidence: **High 217**, **Medium 249**, **Low 53**.

### Top three actual findings

1. **Cross Site Scripting (Reflected)** — High / Medium, 18 instances; parameter `name` at `/vulnerabilities/xss_r/`; evidence `</pre><scrIpt>alert(1);</scRipt><pre>`.
2. **Cross Site Scripting (Persistent)** — High / Medium, 4 instances; parameter `txtName` at `/vulnerabilities/xss_s/`.
3. **Path Traversal** — High / Medium; parameter `page` at `/vulnerabilities/fi/?page=%2Fetc%2Fpasswd`; evidence `root:x:0:0`.

`SQL Injection` and `Remote File Inclusion` alerts also appear in the original export but were not ranked above these three without additional manual validation.

## Repository contents

- `report/` — final DOCX and PDF lab report.
- `reports/` — original unedited ZAP HTML, JSON and PDF exports plus raw alert JSON.
- `evidence/screenshots/` — genuine screenshots and API-evidence renderings.
- `notes/` — isolation evidence, scan statistics and summary notes.

No credentials, session cookies, API keys, VM snapshots, DB dumps, browser profiles, certificates or private keys are included.
