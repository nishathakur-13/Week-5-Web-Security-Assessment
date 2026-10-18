# Web Application Security Assessment Report

> **Status: Preliminary draft — evidence and application checks remain incomplete.**
>
> This report records DNS and HTTP response-header checks, the supplied Nmap and homepage screenshots, and non-submitting inspection of several application pages. It does not claim that authentication, database technology, or application vulnerabilities were tested. Input forms were inspected but not submitted. See the remaining evidence checklist before submitting.

## Cover page

| Item | Details |
|---|---|
| Assignment | Week 5 — Web Application Security Assessment & Defensive Analysis |
| Student | Nisha Kumari *(confirm or edit)* |
| Track | Cyber Security – 6 Week Batch *(confirm or edit)* |
| Assessment date | October 18, 2026 |
| Target | `testasp.vulnweb.com` |

## 1. Executive summary

A limited assessment of the in-scope hostname resolved it to `44.238.29.244`. The supplied Nmap screenshot shows TCP port 80 open running Microsoft IIS httpd 8.5 and port 443 filtered. An HTTP `HEAD` request to the root URL returned `200 OK` over plain HTTP and exposed IIS/ASP.NET headers. The sanitized response capture shows an ASP session cookie without the `Secure`, `HttpOnly`, or `SameSite` attributes, and no `X-Frame-Options`, `X-Content-Type-Options`, or CSP response header on that response. DevTools separately displayed a CSP-related `eval` issue, but that diagnostic was not correlated to a specific response; do not interpret it as proof that the root document has an effective CSP. Browser mapping identified search, login, registration, about, and forum pages, including their visible form fields, without submitting data. The site warns not to post real names, email addresses, or other sensitive information.

These are observations, not proof that an exploitable vulnerability exists throughout the application. The DevTools CSP and Quirks Mode messages could not be correlated to the top-level document: a current inspection found no CSP meta element and reported `document.compatMode` as `CSS1Compat` (standards mode). The site's disclaimer is not independent confirmation of a specific flaw. Form input handling, database technology, and authentication/access-control behavior were not tested, so the overall security posture cannot yet be rated comprehensively. Prioritize reviewing transport and cookie protections.

## 2. Scope

- **In scope:** `testasp.vulnweb.com` and its resolved address `44.238.29.244`, solely as the authorized assignment target.
- **Out of scope:** All other websites, systems, IP addresses, applications, and production infrastructure.
- **Activity/evidence covered:** DNS A-record lookup; HTTP `HEAD` response-header capture to `http://testasp.vulnweb.com/`; Nmap service/version scan of ports 80 and 443 shown in the supplied screenshot; and non-submitting inspection of the homepage, search, login, registration, about, and forum pages.
- **Not performed:** Form submission, authentication attempts, security payload testing, exploit attempts, or checks against other hosts.

## 3. Methodology

The intended assignment workflow is:

Reconnaissance → Technology Discovery → Application Mapping → Security Investigation → Finding Identification → Risk Assessment → Defensive Recommendations

For this preliminary report, hostname resolution, HTTP response-header inspection, the supplied Nmap scan screenshot, and non-submitting browser mapping of several in-scope pages are documented. No input payloads were submitted. Further investigation must remain within the assignment's authorized scope and non-destructive rules.

## 4. Tools used

- PowerShell `Resolve-DnsName` — A-record lookup.
- `curl.exe` — one HTTP `HEAD` request to the target root.
- Nmap 7.92 — service/version scan of TCP ports 80 and 443, evidenced by the supplied terminal screenshot.
- Web browser and Developer Tools — non-submitting inspection of in-scope pages and form fields; the supplied Issues panel screenshot was reviewed as an uncorrelated diagnostic.

## 5. Reconnaissance results

| Item | Observed result | Status / evidence |
|---|---|---|
| Target URL | `http://testasp.vulnweb.com/` | Requested by `curl.exe`; homepage shown in `Evidence/reconnaissance.png` |
| Resolved IPv4 address | `44.238.29.244` | A-record result from `Resolve-DnsName` |
| Protocol / response | HTTP; `200 OK` to `HEAD` | One response observed; Nmap reports port 443 filtered, which does not establish HTTPS application behavior |
| Web server | Microsoft IIS httpd 8.5 | Nmap screenshot and `Server: Microsoft-IIS/8.5` response header |
| Application technology | ASP, as stated in the visible page notice; `ASP.NET` in `X-Powered-By` response header | Homepage screenshot and HTTP response; underlying implementation not independently verified |
| Content type | `text/html` | `Content-Type` response header |
| Cookie observed | An `ASPSESSIONID...` cookie was set | Cookie value intentionally omitted; attributes are described in Finding 02 |
| Nmap result | TCP 80 open (`http`, Microsoft IIS httpd 8.5); TCP 443 filtered (`https`) | Supplied screenshot `Evidence/nmap.png`; scan shows one host and service detection |
| Nmap OS information | Nmap service info identifies Windows | Banner/fingerprint output only; treat as a scan hint, not definitive OS verification |
| Homepage and visible features | Acuforum forum listing with Acunetix Web Vulnerability Scanner, Weather, and Miscellaneous categories; links include about, forums, search, login, register, SQL scanner, and SQL vulnerability help | Supplied screenshot `Evidence/reconnaissance.png`; homepage only |
| Application notice | States that this is an Acunetix test/demo application intentionally vulnerable to SQL injection, directory traversal and other web attacks; says forum content is erased daily | Visible on the homepage screenshot; statement does not independently verify any specific flaw |
| Browser DevTools diagnostics | Issues panel reports a CSP/eval diagnostic and a Quirks Mode warning; current top-level document inspection found no CSP meta element and reported `document.compatMode` as `CSS1Compat` | `Evidence/devtools-issues.png`; diagnostic could not be tied to a particular document/resource |
| Search page | `/Search.asp`; GET form with text input `tfSearch` and submit control | `Evidence/search-page.png`; form not submitted |
| Login page | `/Login.asp?RetURL=...`; POST form with text field `tfUName`, password field `tfUPass`, and submit control | `Evidence/login-page.png`; credentials not entered and form not submitted |
| Registration page | `/Register.asp?RetURL=...`; POST form with text fields `tfUName`, `tfRName`, `tfEmail`, password field `tfUPass`, and submit control | `Evidence/register-page.png`; no personal data entered and form not submitted |
| About page | `/Templatize.asp?item=html/about.html`; page says not to use site as a forum or post sensitive data, including email addresses or real names | `Evidence/about-page.png` |
| Forum page / parameter | `/showforum.asp?id=0` lists thread IDs; homepage links expose `showforum.asp?id=0`, `id=1`, `id=2` | `Evidence/forum-page.png`; no parameter manipulation performed |
| Authentication behavior | Login and registration forms are exposed, but authentication behavior was not assessed | No credentials entered and no forms submitted |
| Database technology | Not observable from collected evidence | Do not infer it from ASP/IIS headers |
| Other visible parameters | `RetURL` appears on login/register links; `item` on about route; `tfSearch` is the search form field | Observed route/form names only; no security testing of these parameters performed |

The sanitized response capture records the server's response Date as October 18, 2026 (UTC). The Nmap screenshot shows 80/tcp open and 443/tcp filtered for that scan; filtered does not mean closed. Routes and form fields were inspected without submitting forms or parameters. Database technology remains unknown.

## 6. Security observations

The following are observations from captured responses and browser diagnostics. Their security impact is conditional and should not be described as a confirmed exploit.

### Observation 01 — Unencrypted HTTP response

- **Type:** Security observation
- **Location:** `http://testasp.vulnweb.com/`
- **Observed:** The root URL returned `200 OK` over HTTP rather than redirecting this request to HTTPS.
- **Evidence:** `Evidence/http-headers.txt` records `HTTP/1.1 200 OK`; `Evidence/reconnaissance.png` shows the site loaded over HTTP.
- **Why it matters:** HTTP does not protect page content or data in transit from network interception or modification. The risk to credentials or other sensitive data depends on whether such data is sent over this HTTP origin; that was not tested.
- **Assessment:** Confirmed response behavior; impact on sensitive data not established.

### Observation 02 — Session cookie missing common protection attributes

- **Type:** Potential weakness
- **Location:** `Set-Cookie` response header from the HTTP root response
- **Observed:** An `ASPSESSIONID...` cookie was set without visible `Secure`, `HttpOnly`, or `SameSite` attributes in the captured header. The cookie value is not reproduced in this report.
- **Evidence:** Redacted response in `Evidence/http-headers.txt`. The browser screenshot shows the target served over HTTP. The session cookie's purpose was not verified.
- **Why it matters:** Without `Secure`, a cookie may be sent over unencrypted HTTP; without `HttpOnly`, client-side scripts may be able to read it; without an explicit `SameSite` policy, cross-site request protections may be weaker or dependent on browser defaults. Cookie purpose and application behavior were not tested.
- **Assessment:** Attribute omission is visible in the captured response. Exploitability and whether this cookie is an authenticated session token are unverified.

### Observation 03 — Server product/version disclosure

- **Type:** Security observation
- **Location:** HTTP `Server` response header
- **Observed:** The response disclosed `Microsoft-IIS/8.5`.
- **Evidence:** `Evidence/http-headers.txt` and Nmap service/version output in `Evidence/nmap.png`.
- **Why it matters:** Product/version information can help an attacker tailor reconnaissance. Disclosure alone does not demonstrate that the server is vulnerable or unpatched.
- **Assessment:** Confirmed header disclosure; no associated vulnerable component was verified.

### Observation 04 — Framework disclosure

- **Type:** Security observation
- **Location:** HTTP `X-Powered-By` response header
- **Observed:** The response disclosed `ASP.NET`.
- **Evidence:** `Evidence/http-headers.txt`.
- **Why it matters:** This provides additional technology information that may help fingerprint the application. It does not independently prove the full application stack or a vulnerability.
- **Assessment:** Confirmed header disclosure.

### Observation 05 — Several common browser security headers absent from this response

- **Type:** Security observation / potential weakness
- **Location:** HTTP response headers for the root `HEAD` request
- **Observed:** The captured root response did not contain `X-Frame-Options`, `X-Content-Type-Options`, or a CSP response header. DevTools separately showed a CSP-related `eval` diagnostic, but the current root response had no CSP meta element and the diagnostic was not mapped to a resource.
- **Evidence:** `Evidence/http-headers.txt` and `Evidence/devtools-issues.png`.
- **Why it matters:** Where applicable, `X-Frame-Options`/CSP `frame-ancestors` and `X-Content-Type-Options: nosniff` can help mitigate framing and MIME-sniffing risks. Header absence alone does not prove an exploitable condition; CSP delivery/context needs clarification.
- **Assessment:** Header absence confirmed only for the captured root response. The DevTools issue does not resolve whether another document/resource has a policy.

### Observation 06 — Homepage identifies the application as an intentionally vulnerable demo

- **Type:** Security observation; stated vulnerabilities are not confirmed by this assessment
- **Location:** Disclaimer on the Acuforum homepage and notice on its About page
- **Observed:** The site identifies itself as an Acunetix test/demo application and says it is intentionally vulnerable to SQL injection, directory traversal, and other web-based attacks. It says forum content is erased daily; the About page warns users not to post sensitive information, email addresses, or real names.
- **Evidence:** `Evidence/reconnaissance.png` and `Evidence/about-page.png`.
- **Why it matters:** This provides important authorization and handling context. The statements do not prove that a particular endpoint currently contains a specific vulnerability; the warning also means users should not submit real personal information.
- **Assessment:** Confirmed that the page displays this statement; no injection or traversal test was performed.

### Observation 07 — Browser reports CSP blocking `eval` and Quirks Mode

- **Type:** Browser diagnostic; not a confirmed vulnerability
- **Location:** Acuforum homepage, as shown in the browser Developer Tools Issues panel
- **Observed:** The supplied DevTools screenshot reports “Content Security Policy of your site blocks the use of 'eval' in JavaScript” and warns “Page layout may be unexpected due to Quirks Mode.” A separate current inspection of the root document found no CSP meta element and reported `document.compatMode` as `CSS1Compat` (standards mode).
- **Evidence:** `Evidence/devtools-issues.png`; current response details in `Evidence/http-headers.txt`. The supplied Network panel screenshot has no visible request rows and is not used to draw conclusions.
- **Why it matters:** A CSP can block unsafe dynamic code; scripts that depend on `eval` may fail, but the block is not itself evidence of a security weakness. The Quirks warning is not confirmed for the current top-level document; if applicable to a frame/resource, it may affect rendering consistency.
- **Assessment:** The diagnostics are visible in the supplied screenshot but their target resource/context is unresolved. They are not classified as vulnerabilities or prioritized findings.

## 7. Three priority findings and risk assessment

These are preliminary, response-level risks. Revisit likelihood and severity after collecting application evidence and checking the relevant routes.

### Finding F-01 — HTTP serves the root page without redirecting to HTTPS

- **Severity:** Medium (preliminary)
- **Location:** `http://testasp.vulnweb.com/`
- **Description:** The root page answered the captured request over plain HTTP. The request was not redirected to HTTPS.
- **Evidence:** `Evidence/http-headers.txt` records `HTTP/1.1 200 OK`; `Evidence/reconnaissance.png` shows the page loaded over HTTP.
- **Potential impact:** Traffic can be observed or modified by an on-path party. Exposure of credentials or sensitive content is possible only if the application sends them over HTTP; that has not been verified.
- **Likelihood:** Medium on networks where an attacker can observe or alter traffic; actual sensitive-data exposure is unknown.
- **Risk rationale:** Medium is provisional because an unencrypted endpoint is directly observed, but no sensitive transaction or account flow was tested.
- **Recommendation:** Configure and verify HTTPS for the entire site, redirect HTTP to HTTPS, enable HSTS after confirming HTTPS works for all required hosts, and ensure forms and session cookies are never transmitted over HTTP.

### Finding F-02 — Session cookie lacks visible security attributes

- **Severity:** Medium (preliminary)
- **Location:** `Set-Cookie` header in the root HTTP response
- **Description:** The observed ASP session cookie omitted `Secure`, `HttpOnly`, and `SameSite` in the captured response.
- **Evidence:** Redacted `Set-Cookie` header in `Evidence/http-headers.txt`. The session cookie's purpose was not verified.
- **Potential impact:** Depending on cookie purpose and application behavior, missing attributes can increase exposure to network interception, script access, or cross-site request risks.
- **Likelihood:** Medium if this cookie authenticates a user or is used on sensitive pages; its function has not been established.
- **Risk rationale:** Medium is provisional; both cookie purpose and authenticated-session behavior need verification.
- **Recommendation:** Set `Secure` on cookies served only over HTTPS, set `HttpOnly` when client-side script access is unnecessary, and choose an appropriate `SameSite` policy (`Lax` or `Strict` where compatible). Review cookie scope, lifetime, and session rotation.

### Finding F-03 — Technology information disclosed in response headers

- **Severity:** Low
- **Location:** `Server` and `X-Powered-By` response headers
- **Description:** The response disclosed `Microsoft-IIS/8.5` and `ASP.NET`.
- **Evidence:** `Evidence/http-headers.txt` and the Nmap service/version result in `Evidence/nmap.png`.
- **Potential impact:** Fingerprinting can make targeted reconnaissance easier, but disclosure does not itself provide access or prove a known vulnerability is present.
- **Likelihood:** High that a visitor can observe the headers; likelihood of resulting compromise is not established.
- **Risk rationale:** Low because the direct impact is informational and no exploitable condition was verified.
- **Recommendation:** Remove or minimize unnecessary version/framework headers where supported, and keep the server and application components maintained. Do not treat banner hiding as a substitute for patching.

## 8. Risk summary

| Priority | Finding | Preliminary severity | Main reason |
|---|---|---|---|
| 1 | F-01 — HTTP response | Medium | No transport encryption was observed for the root response; sensitive-data exposure is unverified |
| 2 | F-02 — Cookie attributes | Medium | Cookie protections were absent in the captured response; cookie purpose is unverified |
| 3 | F-03 — Technology disclosure | Low | Fingerprinting information is exposed, but no direct exploitability was demonstrated |

These ratings apply only to the limited observations collected. Header observations and browser diagnostics are documented separately; confirm behavior on relevant interactive pages before assigning further security risk.

## 9. Defensive recommendations

1. Enable HTTPS for all application pages and redirects from HTTP; verify all user input and session traffic stays on HTTPS.
2. Apply cookie attributes appropriate to each cookie's purpose: `Secure`, `HttpOnly`, and an explicitly selected `SameSite` policy where compatible.
3. Remove unnecessary server/framework version headers while maintaining and patching the underlying software.
4. Inspect the actual response/document policy to establish how CSP is delivered; preserve restrictive directives such as blocking `eval` unless compatibility testing shows a justified need to change them. Review framing and MIME-sniffing protections on relevant responses.
5. Complete any additional browser reconnaissance required by the instructor; database technology was not observable from the inspected pages.
6. Investigate input handling and authentication only within the assignment rules. Use non-destructive checks and report potential weaknesses separately from confirmed findings.

## 10. Conclusion

The immediate priorities are to verify and enforce HTTPS and to review the observed session-cookie attributes. The response also discloses server/framework details, which is a lower-priority hardening issue. Homepage, about, search, login, registration, and forum pages were mapped without submitting forms. Authentication behavior and input handling were not tested. Although the site's notice describes it as intentionally vulnerable, this report makes no claims that SQL injection, directory traversal, or another specific vulnerability was independently verified.

## 11. Student reflection

> Personalize these answers based on the work and evidence you actually collect.

1. **What was the most interesting security issue you discovered?**  
   Discovering how default server banners explicitly leak exact platform versions like Microsoft IIS 8.5, illustrating how minor information disclosure points aid attackers in targeted reconnaissance.

2. **What was the most challenging part of the assessment?**  
   Maintaining strict adherence to a defensive blue-team perspective—ensuring all analysis relied strictly on non-invasive tool responses and captured evidence without overstepping into unauthorized testing.

3. **What did you learn about defensive security?**  
   That application hardening requires a multi-layered approach, combining transport encryption, strict cookie attribute configuration (`HttpOnly`, `Secure`), and secure header policies rather than relying on code functionality alone.

4. **Why is evidence important when reporting security findings?**  
   Evidence makes findings easier to reproduce, separates verified observations from assumptions, and helps developers validate and remediate issues.

5. **Which cybersecurity skill would you like to learn next?**

   Advanced Vulnerability Assessment and Penetration Testing (VAPT) methodologies and automated security integration within DevSecOps pipelines.
## Evidence checklist before submission

- [x] Add the supplied `Evidence/reconnaissance.png` showing the target homepage, navigation, and disclaimer.
- [x] Add the supplied `Evidence/nmap.png` showing the scoped Nmap result for ports 80 and 443.
- [x] Add the sanitized `Evidence/http-headers.txt` response capture with the session value redacted.
- [x] Capture search, login, registration, about, and forum-page screenshots without submitting forms or entering data.
- [x] Include a meaningful caption for each evidence image in `Evidence/README.md`.
- [x] Update Section 5 with inspected routes, form fields, and query parameter names; do not imply these inputs were security-tested.
- [x] Update the findings and risk ratings to match evidence actually collected; remove claims you cannot support.
- [x] Complete the student reflection answers.
- [ ] Confirm the student name and batch details are correct.
- [x] Ensure the response-header capture redacts the cookie value and form screenshots contain no entered personal/sensitive data.
- [ ] Capture a terminal/DevTools screenshot of the response headers if the instructor requires screenshots for each header-based observation.
