# Week 5: Web Application Security Assessment

**Assignment:** Web Application Security Assessment & Defensive Analysis  
**Student:** Nisha Kumari *(edit if needed)*  
**Track:** Cyber Security – 6 Week Batch  
**Target:** `testasp.vulnweb.com`  
**Assessment date:** October 18, 2026

## Repository contents

```text
Week-5-Web-Security-Assessment/
├── README.md
├── Security-Assessment-Report.md
└── Evidence/
    └── README.md
```

The report is a preliminary assessment based on the limited, non-invasive checks described in it. The evidence folder still needs your own screenshots before submission. Do not present unperformed tests as completed or treat a potential weakness as a confirmed vulnerability.

## Assessment scope and safety

Only `testasp.vulnweb.com` is in scope. This work did not submit form data, attempt authentication, exploit vulnerabilities, modify data, or scan other systems. Follow your mentor's authorization and the assignment's safety rules for any further checks.

## Finish the evidence

Capture your own screenshots and save them in `Evidence/` using the filenames and captions in [Evidence/README.md](Evidence/README.md). In particular, capture the Nmap output and the browser/application observations you personally verify. Update the report's evidence references and any marked unknowns to match those screenshots.

## Optional Nmap command

If Nmap is installed and your assignment authorization permits this check, run this limited service/version check against the authorized hostname only:

```powershell
nmap -sV -p 80,443 testasp.vulnweb.com
```

Save or screenshot the actual result as `Evidence/nmap.png`. Do not copy sample scan results into the report.

## GitHub submission

Create an empty GitHub repository, then replace the placeholder URL below with its HTTPS or SSH URL and run these commands from this folder:

```powershell
git init
git add README.md Security-Assessment-Report.md Evidence/
git commit -m "Complete Week 5 web security assessment"
git branch -M main
git remote add origin <YOUR_GITHUB_REPOSITORY_URL>
git push -u origin main
```

Submit the repository URL only after the screenshots are present, report placeholders are resolved, and the repository is accessible to your instructor.
