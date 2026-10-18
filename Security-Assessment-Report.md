# Web Application Security Assessment Report

> **Status: Preliminary draft — evidence capture and assignment checks remain incomplete.**  
> This report records only the limited DNS lookup and HTTP `HEAD` response collected on the assessment date. It does not claim that Nmap, browser inspection, form testing, authentication testing, or vulnerability exploitation was performed. Complete the evidence checklist and replace all pending items before submitting.

## Cover page

| Item | Details |
|---|---|
| Assignment | Week 5 — Web Application Security Assessment & Defensive Analysis |
| Student | Nisha Kumari *(confirm or edit)* |
| Track | Cyber Security – 6 Week Batch *(confirm or edit)* |
| Assessment date | October 18, 2026 |
| Target | `testasp.vulnweb.com` |

## 1. Executive summary

A limited, non-invasive check of the in-scope hostname resolved it to `44.238.29.244`. An HTTP `HEAD` request to the root URL returned `200 OK` over plain HTTP and exposed an IIS/ASP.NET server banner. The captured response also set an ASP session cookie without the `Secure`, `HttpOnly`, or `SameSite` attributes, and did not include several commonly used browser security headers.

These are confirmed observations of one captured response, not proof that an exploitable vulnerability exists throughout the application. No Nmap scan, HTTPS check, browser mapping, input testing, or authentication/access-control investigation has been completed for this draft. The overall security posture therefore cannot yet be rated comprehensively. Prioritize verifying whether HTTPS is available and required, and review cookie protections; capture the remaining assignment evidence before finalizing risk ratings.

## 2. Scope

- **In scope:** `testasp.vulnweb.com` and its resolved address `44.238.29.244`, solely as the authorized assignment target.
- **Out of scope:** All other websites, systems, IP addresses, applications, and production infrastructure.
- **Activity performed for this draft:** DNS A-record lookup and one HTTP `HEAD` request to `http://testasp.vulnweb.com/`.
- **Not performed:** Port scanning, application crawling, form submission, authentication testing, exploit attempts, or checks against other hosts.

## 3. Methodology

The intended assignment workflow is:

Reconnaissance → Technology Discovery → Application Mapping → Security Investigation → Finding Identification → Risk Assessment → Defensive Recommendations

For this preliminary report, only hostname resolution and a single HTTP response-header check were completed. The other stages must be completed using the assignment's authorized scope and non-destructive rules before they can be reported as assessed.

## 4. Tools used

- PowerShell `Resolve-DnsName` — A-record lookup.
- `curl.exe` — one HTTP `HEAD` request to the target root.

Nmap, a browser, browser developer tools, and other scanners are **not** listed as tools used because their results were not collected for this draft.

## 5. Reconnaissance results

| Item | Observed result | Status / evidence |
|---|---|---|
| Target URL | `http://testasp.vulnweb.com/` | Requested by `curl.exe` |
| Resolved IPv4 address | `44.238.29.244` | A-record result from `Resolve-DnsName` |
| Protocol / response | HTTP; `200 OK` to `HEAD` | One response observed; HTTPS was not checked |
| Web server banner | `Microsoft-IIS/8.5` | `Server` response header |
| Application technology header | `ASP.NET` | `X-Powered-By` response header; this is a disclosed header, not independent verification of all application components |
| Content type | `text/html` | `Content-Type` response header |
| Cookie observed | An `ASPSESSIONID...` cookie was set | Cookie value intentionally omitted; attributes are described in Finding 02 |
| Open ports / service discovery | Not assessed | Run the permitted, scoped Nmap check and record its actual output |
| Pages, forms, inputs, authentication, endpoints, database | Not assessed | Complete browser-based mapping; do not infer these from the headers |

The captured HTTP response reported a `Date` of October 18, 2026 (UTC). Do not interpret the single `HEAD` response as a complete application inventory.

## 6. Security observations

The following are observations from the captured response. Their security impact is conditional and should not be described as a confirmed exploit.

### Observation 01 — Unencrypted HTTP response

- **Type:** Security observation
- **Location:** `http://testasp.vulnweb.com/`
- **Observed:** The root URL returned `200 OK` over HTTP rather than redirecting this request to HTTPS.
- **Evidence:** The request URL and `HTTP/1.1 200 OK` response from the recorded `curl.exe -I` check. Add your own screenshot as `Evidence/finding-01.png`.
- **Why it matters:** HTTP does not protect page content or data in transit from network interception or modification. The risk to credentials or other sensitive data depends on whether such data is sent over this HTTP origin; that was not tested.
- **Assessment:** Confirmed response behavior; impact on sensitive data not established.

### Observation 02 — Session cookie missing common protection attributes

- **Type:** Potential weakness
- **Location:** `Set-Cookie` response header from the HTTP root response
- **Observed:** An `ASPSESSIONID...` cookie was set without visible `Secure`, `HttpOnly`, or `SameSite` attributes in the captured header. The cookie value is not reproduced in this report.
- **Evidence:** The recorded `curl.exe -I` response. Add a redacted screenshot as `Evidence/finding-02.png`; ensure the cookie value is hidden.
- **Why it matters:** Without `Secure`, a cookie may be sent over unencrypted HTTP; without `HttpOnly`, client-side scripts may be able to read it; without an explicit `SameSite` policy, cross-site request protections may be weaker or dependent on browser defaults. Cookie purpose and application behavior were not tested.
- **Assessment:** Attribute omission is visible in the captured response. Exploitability and whether this cookie is an authenticated session token are unverified.

### Observation 03 — Server product/version disclosure

- **Type:** Security observation
- **Location:** HTTP `Server` response header
- **Observed:** The response disclosed `Microsoft-IIS/8.5`.
- **Evidence:** The recorded HTTP response headers. Add a screenshot as `Evidence/finding-03.png`.
- **Why it matters:** Product/version information can help an attacker tailor reconnaissance. Disclosure alone does not demonstrate that the server is vulnerable or unpatched.
- **Assessment:** Confirmed header disclosure; no associated vulnerable component was verified.

### Observation 04 — Framework disclosure

- **Type:** Security observation
- **Location:** HTTP `X-Powered-By` response header
- **Observed:** The response disclosed `ASP.NET`.
- **Evidence:** The recorded HTTP response headers.
- **Why it matters:** This provides additional technology information that may help fingerprint the application. It does not independently prove the full application stack or a vulnerability.
- **Assessment:** Confirmed header disclosure.

### Observation 05 — Several common browser security headers absent from this response

- **Type:** Security observation / potential weakness
- **Location:** HTTP response headers for the root `HEAD` request
- **Observed:** The captured response did not contain `Content-Security-Policy`, `X-Frame-Options`, or `X-Content-Type-Options`.
- **Evidence:** The complete header output from the recorded `curl.exe -I` request. Add a screenshot as `Evidence/finding-04.png`.
- **Why it matters:** Depending on the application's content and browser behavior, these controls can reduce exposure to framing, some script-injection consequences, and MIME sniffing. Their absence alone does not establish that those attacks are possible.
- **Assessment:** Confirmed for this one response only; other paths and application-level protections were not assessed.

## 7. Three priority findings and risk assessment

These are preliminary, response-level risks. Revisit likelihood and severity after collecting application evidence and checking the relevant routes.

### Finding F-01 — HTTP serves the root page without redirecting to HTTPS

- **Severity:** Medium (preliminary)
- **Location:** `http://testasp.vulnweb.com/`
- **Description:** The root page answered the captured request over plain HTTP. The request was not redirected to HTTPS.
- **Evidence:** `HTTP/1.1 200 OK` from the one `curl.exe -I` request; capture `Evidence/finding-01.png`.
- **Potential impact:** Traffic can be observed or modified by an on-path party. Exposure of credentials or sensitive content is possible only if the application sends them over HTTP; that has not been verified.
- **Likelihood:** Medium on networks where an attacker can observe or alter traffic; actual sensitive-data exposure is unknown.
- **Risk rationale:** Medium is provisional because an unencrypted endpoint is directly observed, but no sensitive transaction or account flow was tested.
- **Recommendation:** Configure and verify HTTPS for the entire site, redirect HTTP to HTTPS, enable HSTS after confirming HTTPS works for all required hosts, and ensure forms and session cookies are never transmitted over HTTP.

### Finding F-02 — Session cookie lacks visible security attributes

- **Severity:** Medium (preliminary)
- **Location:** `Set-Cookie` header in the root HTTP response
- **Description:** The observed ASP session cookie omitted `Secure`, `HttpOnly`, and `SameSite` in the captured response.
- **Evidence:** The captured `Set-Cookie` header; store only a redacted screenshot at `Evidence/finding-02.png`.
- **Potential impact:** Depending on cookie purpose and application behavior, missing attributes can increase exposure to network interception, script access, or cross-site request risks.
- **Likelihood:** Medium if this cookie authenticates a user or is used on sensitive pages; its function has not been established.
- **Risk rationale:** Medium is provisional; both cookie purpose and authenticated-session behavior need verification.
- **Recommendation:** Set `Secure` on cookies served only over HTTPS, set `HttpOnly` when client-side script access is unnecessary, and choose an appropriate `SameSite` policy (`Lax` or `Strict` where compatible). Review cookie scope, lifetime, and session rotation.

### Finding F-03 — Technology information disclosed in response headers

- **Severity:** Low
- **Location:** `Server` and `X-Powered-By` response headers
- **Description:** The response disclosed `Microsoft-IIS/8.5` and `ASP.NET`.
- **Evidence:** The recorded HTTP response; capture `Evidence/finding-03.png`.
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

These ratings apply only to the limited observations collected. Missing headers are documented separately and should also be reviewed, especially after inspecting pages that contain interactive content.

## 9. Defensive recommendations

1. Enable HTTPS for all application pages and redirects from HTTP; verify all user input and session traffic stays on HTTPS.
2. Apply cookie attributes appropriate to each cookie's purpose: `Secure`, `HttpOnly`, and an explicitly selected `SameSite` policy where compatible.
3. Remove unnecessary server/framework version headers while maintaining and patching the underlying software.
4. Review whether `Content-Security-Policy`, `X-Frame-Options` or `frame-ancestors`, and `X-Content-Type-Options: nosniff` are appropriate for the application, then test for compatibility.
5. Complete the authorized Nmap and browser reconnaissance tasks; record actual ports, pages, forms, and technologies without guessing.
6. Investigate input handling and authentication only within the assignment rules. Use non-destructive checks and report potential weaknesses separately from confirmed findings.

## 10. Conclusion

The immediate priorities are to verify and enforce HTTPS and to review the observed session-cookie attributes. The response also discloses server/framework details, which is a lower-priority hardening issue. Because application mapping, Nmap, authentication flows, and input handling were not assessed for this draft, the report is not a complete security assessment and makes no claims about SQL injection, directory traversal, or other untested vulnerabilities.

## 11. Student reflection

> Personalize these answers based on the work and evidence you actually collect.

1. **What was the most interesting security issue you discovered?**  
   Pending personal reflection after completing the assessment. A possible topic from the limited checks is how one response can expose both transport and cookie-configuration concerns.
2. **What was the most challenging part of the assessment?**  
   Pending personal reflection.
3. **What did you learn about defensive security?**  
   Pending personal reflection.
4. **Why is evidence important when reporting security findings?**  
   Evidence makes observations reproducible, distinguishes verified facts from assumptions, and helps developers validate remediation.
5. **Which cybersecurity skill would you like to learn next?**  
   Pending personal reflection.

## Evidence checklist before submission

- [ ] Add your own `Evidence/reconnaissance.png` showing the target and relevant application pages.
- [ ] Add your own `Evidence/nmap.png` showing the actual scoped Nmap result, if performed.
- [ ] Add redacted `Evidence/finding-01.png`, `Evidence/finding-02.png`, and `Evidence/finding-03.png`.
- [ ] Add `Evidence/finding-04.png` for the response-header observation, or update the report if you use a different evidence naming scheme.
- [ ] Include a meaningful caption for each screenshot in `Evidence/README.md`.
- [ ] Complete browser mapping (pages, forms, inputs, authentication, and interesting endpoints) and update Section 5 with observed facts.
- [ ] Update the findings and risk ratings to match evidence actually collected; remove claims you cannot support.
- [ ] Replace pending reflection answers and confirm student details.
- [ ] Ensure screenshots redact cookie values and any personal or sensitive data.
