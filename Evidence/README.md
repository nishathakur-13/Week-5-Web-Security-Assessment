# Evidence checklist and captions

Capture screenshots yourself during the authorized assessment. Do not use screenshots from websites, tutorials, or other students. Keep the files in this folder and verify that each image supports the related statement in `../Security-Assessment-Report.md`.

| Filename | Suggested caption / what to capture | Status |
|---|---|---|
| `reconnaissance.png` | Figure 1 — Acuforum homepage showing the target, visible navigation/features, and the site's training/demo disclaimer. | Added |
| `nmap.png` | Figure 2 — Nmap 7.92 service/version scan output for `testasp.vulnweb.com`, ports 80 and 443. | Added |
| `http-headers.txt` | Sanitized `curl.exe -I` response showing status, server/framework headers, cookie attributes with value redacted, and headers absent from this one response. | Added |
| `search-page.png` | Figure 3 — Search form with the `tfSearch` text field; no search was submitted. | Added |
| `login-page.png` | Figure 4 — Login form showing username/password fields; no credentials entered or submitted. | Added |
| `register-page.png` | Figure 5 — Registration form fields; no personal data entered or submitted. | Added |
| `about-page.png` | Figure 6 — About page warning users not to post sensitive information, email addresses, or real names. | Added |
| `forum-page.png` | Figure 7 — Forum page listing threads and their `id` parameters; no parameter manipulation performed. | Added |
| `devtools-issues.png` | Figure 8 — Browser DevTools Issues panel showing CSP/eval and Quirks Mode diagnostics; resource/context not established. | Added |

The Nmap, homepage, and DevTools images were supplied by the student and copied into this folder without editing. The search, login, registration, about, and forum-page screenshots were captured during non-submitting inspection. The HTTP headers file is a text capture with the cookie value redacted. The supplied Network panel screenshot had no visible request rows and is not used to claim any network result. Do not enter real credentials or personal information in this demonstration application.
