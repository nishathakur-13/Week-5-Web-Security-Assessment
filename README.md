# Week 5: Web Application Security Assessment

**Assignment:** Web Application Security Assessment & Defensive Analysis  
**Student:** Nisha Kumari 
**Track:** Cyber Security – 6 Week Batch  
**Target:** `testasp.vulnweb.com`  
**Assessment date:** October 18, 2026

## Repository contents

```text
Week-5-Web-Security-Assessment/
├── README.md
├── Security-Assessment-Report.md
└── Evidence/
    ├── README.md
    ├── about-page.png
    ├── devtools-issues.png
    ├── forum-page.png
    ├── http-headers.txt
    ├── login-page.png
    ├── nmap.png
    ├── reconnaissance.png
    ├── register-page.png
    └── search-page.png
```

The preliminary report now includes the Nmap result, homepage, search/login/registration/about/forum page screenshots, and a sanitized HTTP response-header capture. It still does not claim that authentication or input security was tested. Do not present unperformed tests as completed or treat a potential weakness as a confirmed vulnerability.

## Assessment scope and safety

Only `testasp.vulnweb.com` is in scope. This work did not submit form data, attempt authentication, exploit vulnerabilities, modify data, or scan other systems. Follow your mentor's authorization and the assignment's safety rules for any further checks.

## Finish the evidence

The evidence folder contains the supplied screenshots plus screenshots captured during non-submitting inspection of in-scope application pages. Cookie values in the HTTP header capture are redacted. See [Evidence/README.md](Evidence/README.md) for captions and remaining evidence needs.

## Assessment completion status

The required Nmap result, homepage screenshot, browser-mapping screenshots, and sanitized HTTP header capture are included. The report records five response-based security observations and three prioritized findings, while distinguishing browser diagnostics from confirmed weaknesses. Do not claim that SQL injection, directory traversal, authentication, or input validation were tested.

The Nmap screenshot shows the actual scoped service/version check:

```powershell
nmap -sV -p 80,443 testasp.vulnweb.com
```

The screenshot is saved as `Evidence/nmap.png`. Do not copy sample scan results into the report.

Browser mapping identified the visible search, login, registration, about, and forum features and recorded observed form fields and route parameters without submitting inputs. Database technology was not observable and is documented as unknown.

Before submitting, verify all of the following:

- [x] Complete the authorized Nmap check and add `Evidence/nmap.png`.
- [x] Add the supplied browser homepage screenshot as `Evidence/reconnaissance.png`.
- [x] Capture browser evidence for the search, login, registration, about, and forum pages without submitting forms or entering personal information.
- [x] Record actual HTTP response headers in `Evidence/http-headers.txt`, with the session cookie value redacted.
- [ ] Confirm the student name and batch details are correct.
- [ ] If your instructor specifically requires screenshots of raw response headers, capture one with the cookie value redacted.
- [x] Confirm the report includes at least five evidence-supported observations and three prioritized risk assessments.
- [ ] Commit and push the final files to GitHub, or submit the report and evidence using another allowed option.

## GitHub submission

This working copy already has the GitHub remote configured. Confirm that it is your intended repository, then run these commands from this folder after your final review:

```powershell
git add README.md Security-Assessment-Report.md Evidence/
git commit -m "Complete Week 5 web security assessment"
git push origin main
```

Submit the repository URL only after the commit is pushed and you have confirmed that your instructor can access it.
