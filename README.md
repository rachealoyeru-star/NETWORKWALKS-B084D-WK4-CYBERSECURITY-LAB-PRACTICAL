# 👨‍💻 PENETRATION TESTING REPORT
**AUTHORIZED BLACK-BOX WEB APPLICATION PENTEST — MEDIROZA GENERAL HOSPITAL (TRAINING TARGET)**

![Skill-Cybersecurity](https://img.shields.io/badge/Skill-Cybersecurity-404040?style=flat-square&labelColor=C00000)
![Web Pentest](https://img.shields.io/badge/Black--Box%20Pentest-C00000?style=flat-square&labelColor=000000&logo=kalilinux&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-404040?style=flat-square&labelColor=C00000&logo=kalilinux&logoColor=white)
![Windows OS](https://img.shields.io/badge/windows%20os-404040?style=flat-square&labelColor=C00000&logo=windows&logoColor=white)
![NetworkWalks](https://img.shields.io/badge/NetworkWalks-404040?style=flat-square&labelColor=C00000)
![GitHub](https://img.shields.io/badge/GitHub-0070C0?style=flat-square&labelColor=000000&logo=github&logoColor=white)

![Pentester: Oyeru Rachael Emalohi](https://img.shields.io/badge/👨🏽‍💻‍💻Pentester:%20Oyeru%20Rachael%20Emalohi-C00000?style=flat-square)

---

W4-PM | CYBERSECURITY | NETWORKWALKS

🚥 Project Overview

| **Field**                | **Detail**                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------ |
| **Pentester**             | **Oyeru Rachael Emalohi**                                                                        |
| **Program/Batch**         | Networkwalks-B084D                                                                               |
| **Date**                 | 24th–25th November, 2026                                                                          |
| **Modules completed**     | W4-PM (Black-Box Web Application Penetration Test)                                                |
| **Client/Target**         | Mediroza General Hospital — `medirozahospital.com` (fictional training target)                    |
| **Permission secured?**   | Yes — explicit written authorization granted for this Networkwalks educational engagement         |
| **Milestones covered**    | **M1:** Website Attack & Confidential PDF Retrieval &nbsp;**M2:** PDF Password Cracking &nbsp;**M3:** Staff Salary & Shareholder Data Exposure |

---

# ⛔ 1. Liability Disclaimer

The activities documented in this report were conducted strictly against `medirozahospital.com`, a fictional training target provided for this Networkwalks educational lab, under explicit written authorization granted for the duration of the exercise. No real patient, staff, or shareholder data was involved — all records recovered are simulated training data. The contents of this document are intended strictly for educational and research purposes. The instructor, author, and Networkwalks assume no liability for the misuse or misapplication of this knowledge; all subsequent actions taken by the reader are their sole responsibility. Unauthorized access to systems or data constitutes a criminal offense in most jurisdictions, regardless of whether data or systems are damaged, and can result in severe legal penalties, significant fines, career impact (employment termination), and a permanent criminal record.

> ⚠️ **Note: Only test systems you own, or systems you have explicit written authorization to test.**

---

# 🔦 2. Introduction

As part of my Networkwalks internship program (Week 4), this report documents a black-box web application penetration test carried out against Mediroza General Hospital's training website, `medirozahospital.com`. The engagement followed a standard black-box methodology — reconnaissance, entry-point identification, authentication-behaviour analysis, input-handling testing, and exploitation — with the goal of retrieving confidential patient lab reports, recovering their encryption passwords, and identifying any further critical data exposure on the server.

### 💡 Reconnaissance

Performed using `curl` from a Kali Linux VM to enumerate `robots.txt`, `sitemap.xml`, and the directories the site itself tried to hide from search engines and casual visitors.

### 🔎 Exploitation

A username-enumeration flaw and a classic SQL-injection authentication bypass (`admin'--`) were used to gain access to the Patient Portal without valid credentials, from which three password-protected pathology PDFs were retrieved. An exposed `/old/` directory also yielded a full SQL database backup containing staff and shareholder records.

### 📝 Methodology, analysis framework, and documentation

To ensure reproducibility and rigorous documentation, every technical step outlines the exact command or tool used, the result obtained, verification screenshots, and a brief analytical note on the security implications of each finding.

---

# 🧰 3. Tools Used

| Tool/commands        | Purpose                                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------- |
| Kali Linux            | Operating system used to run all reconnaissance, exploitation, and password-cracking commands.             |
| `curl`                | Used throughout for reconnaissance — headers, `robots.txt`, `sitemap.xml`, directory probing, and raw HTTP responses. |
| Firefox (inside Kali) | Used to interact with the Patient Portal login form and download the retrieved PDF reports.               |
| `wget`                | Used to download the exposed SQL database backup file from the `/old/` directory.                         |
| `pdf2john`            | Attempted to extract a crackable hash from each encrypted PDF for John the Ripper.                         |
| John the Ripper       | Attempted offline dictionary attack against the extracted PDF hashes (hash loading failed in this environment). |
| `pdfcrack`             | Alternative offline PDF password-cracking tool; successfully recovered all three PDF passwords directly against the files. |
| `grep`                | Used to parse the SQL backup file and cleanly extract the `staff` and `shareholders` table contents.       |

---

---

⚒️ Activities Performed

---

# 4. M1 — Website Reconnaissance & Confidential PDF Retrieval

## 4.1. Initial Reconnaissance

`curl` was used to check the site's headers, `robots.txt`, and `sitemap.xml`. The `robots.txt` file disclosed three directories the site was trying to keep out of search engines: `/patient/`, `/staff/`, and `/old/` — an immediate signal of where to dig further.

[![robots.txt and sitemap.xml recon](screenshots/m1-recon-robots-txt-sitemap.jpg)](screenshots/m1-recon-robots-txt-sitemap.jpg)

> **Disclosed paths:** `Disallow: /patient/` · `Disallow: /staff/` · `Disallow: /old/`

A follow-up check confirmed the `/patient/` path resolves with `HTTP/2 200`, confirming it as a live, reachable directory rather than a dead link.

[![Confirming /patient/ is reachable](screenshots/m1-recon-patient-directory-check.jpg)](screenshots/m1-recon-patient-directory-check.jpg)

## 4.2. Username Enumeration on the Patient Portal

Submitting a non-existent username on `/patient/login.php` returned a distinct **"Username not found"** message, while a valid username with the wrong password returned **"Incorrect password"**. This differential error messaging confirms a username-enumeration vulnerability — an attacker can confirm valid usernames before ever guessing a password.

[![Username not found](screenshots/m1-username-enum-not-found.jpg)](screenshots/m1-username-enum-not-found.jpg)
[![Incorrect password](screenshots/m1-username-enum-incorrect-password.jpg)](screenshots/m1-username-enum-incorrect-password.jpg)

Viewing the page source confirmed the login form posts plain `username`/`password` fields to `login.php` with no visible CSRF token or rate-limiting hint.

[![Login form source, part 1](screenshots/m1-login-viewsource-fields-1.jpg)](screenshots/m1-login-viewsource-fields-1.jpg)
[![Login form source, part 2](screenshots/m1-login-viewsource-fields-2.jpg)](screenshots/m1-login-viewsource-fields-2.jpg)

## 4.3. Confirming the `/staff/` and `/old/` Directories

Both `/staff/` and `/old/` — disallowed in `robots.txt` — returned `HTTP/2 200`, confirming both are live and browsable.

[![/staff/ and /old/ both return 200](screenshots/m1-staff-old-dirs-200.jpg)](screenshots/m1-staff-old-dirs-200.jpg)

Browsing to `/old/` revealed **directory listing was left enabled**, exposing a single file: `mediroza_db_backup_2019.sql`. This single misconfiguration is the root cause behind the M3 finding later in this report.

[![Index of /old/ — mediroza_db_backup_2019.sql exposed](screenshots/m1-old-directory-listing.jpg)](screenshots/m1-old-directory-listing.jpg)

The backup file was downloaded directly with `wget`:

[![wget download of mediroza_db_backup_2019.sql](screenshots/m1-db-backup-wget-download.jpg)](screenshots/m1-db-backup-wget-download.jpg)

> **Downloaded:** `mediroza_db_backup_2019.sql` (6.2 KB) — content and analysis covered in **Section 6 (M3)**.

The `/patient/` directory itself also had listing enabled, exposing its internal file structure (`login.php`, `portal.php`, `logout.php`, `error_log`):

[![/patient/ directory listing source](screenshots/m1-patient-directory-listing-source.jpg)](screenshots/m1-patient-directory-listing-source.jpg)

A guessed `/patient/reports/` path correctly returned `403 Forbidden` — a properly locked-down path, included here for contrast with the misconfigured directories above.

[![/patient/reports/ returns 403 Forbidden](screenshots/m1-patient-reports-403-forbidden.jpg)](screenshots/m1-patient-reports-403-forbidden.jpg)

## 4.4. Session Behaviour

Requesting `portal.php` while unauthenticated returned `HTTP/2 302`, redirecting to `login.php` and issuing a new `PHPSESSID` session cookie — standard session-gated access control, confirming the portal pages are protected by server-side session checks (not just hidden by obscurity).

[![portal.php 302 redirect + PHPSESSID cookie](screenshots/m1-session-cookie-redirect.jpg)](screenshots/m1-session-cookie-redirect.jpg)

## 4.5. SQL Injection Authentication Bypass

The Patient Portal login form was tested with the classic SQL-injection payload `admin'--` in the username field (any value in the password field). The application accepted the payload and logged straight into the portal with no valid credentials supplied — confirming the login query is not parameterised and is vulnerable to authentication bypass via SQL injection.

This granted access to the **"My lab reports"** page, listing three password-protected pathology PDFs:

[![SQLi bypass — My lab reports page](screenshots/m1-sqli-bypass-lab-reports-page.jpg)](screenshots/m1-sqli-bypass-lab-reports-page.jpg)
[![Patient portal — three encrypted lab reports listed](screenshots/m2-patient-portal-lab-reports-list.png)](screenshots/m2-patient-portal-lab-reports-list.png)

> **Reports retrieved:** Pathology Report – S. Dlamini (`LR-2024-1187`) · Pathology Report – P. Reddy (`LR-2024-1192`) · Pathology Report – E. Thompson (`LR-2024-1205`)

> ⚠️ **Security relevance:** An unauthenticated attacker can fully bypass login with a single, decades-old SQL-injection payload and directly retrieve confidential patient lab reports — the most severe finding in this engagement.

---

# 5. M2 — PDF Password Cracking

## 5.1. Retrieving the Files Correctly

The three PDFs were first downloaded via Chrome on the Windows host rather than inside the Kali VM, so they were not available to the cracking tools. This was corrected by logging back into the Patient Portal from Firefox **inside** the Kali VM (same SQL-injection bypass) and re-downloading all three files directly into `~/Downloads`.

[![First attempt — files downloaded outside the Kali VM](screenshots/m2-downloads-wrong-browser-first-attempt.jpg)](screenshots/m2-downloads-wrong-browser-first-attempt.jpg)
[![patient_report_1/2/3.pdf correctly saved inside Kali, password prompt confirms encryption](screenshots/m2-downloads-pdfs-password-prompt.jpg)](screenshots/m2-downloads-pdfs-password-prompt.jpg)

## 5.2. Hash Extraction Attempt (John the Ripper)

`pdf2john` successfully extracted a valid `$pdf$...` hash from each file, and John the Ripper's format list confirmed native `PDF` format support. However, both a default dictionary attack and an explicit `--format=PDF` attack returned **"No password hashes loaded (see FAQ)"** for all three hash files — John was unable to load its own correctly-formatted hashes in this environment.

[![pdf2john extraction + John the Ripper failing to load the hash](screenshots/m2-pdf2john-hash-extraction.jpg)](screenshots/m2-pdf2john-hash-extraction.jpg)
[![--format=PDF confirmed supported, attack still fails](screenshots/m2-john-format-pdf-confirmed-but-fails.jpg)](screenshots/m2-john-format-pdf-confirmed-but-fails.jpg)

## 5.3. Pivoting to `pdfcrack`

`pdfcrack` was installed as a working alternative, running directly against each PDF file (no separate hash-extraction step needed):

[![Installing pdfcrack](screenshots/m2-pdfcrack-install.jpg)](screenshots/m2-pdfcrack-install.jpg)

**`patient_report_1.pdf`** — cracked on the first run:

[![pdfcrack recovers patient_report_1.pdf password: 123456](screenshots/m2-pdfcrack-success-pdf1-123456.jpg)](screenshots/m2-pdfcrack-success-pdf1-123456.jpg)

> **Result: password cracked — `123456`**

**`patient_report_2.pdf`** — cracked with the same command; recovered password: **`password`**.

**`patient_report_3.pdf`** — an early attempt failed because the command was run from the wrong working directory (`~` instead of `~/Downloads`); re-running it from the correct directory recovered the password:

[![Wrong-directory error for patient_report_3.pdf](screenshots/m2-pdfcrack-wrong-directory-error.jpg)](screenshots/m2-pdfcrack-wrong-directory-error.jpg)
[![pdfcrack recovers patient_report_3.pdf password: !@#$%^& after cd into ~/Downloads](screenshots/m2-pdfcrack-success-pdf3.jpg)](screenshots/m2-pdfcrack-success-pdf3.jpg)

> **Result: password cracked — `!@#$%^&`**

## 5.4. Recovered Report Contents

Opening each file with its recovered password confirmed the full pathology report contents:

[![Unlocked — Pathology Report, S. Dlamini](screenshots/m2-unlocked-pdf1-dlamini.png)](screenshots/m2-unlocked-pdf1-dlamini.png)
[![Unlocked — Pathology Report, P. Reddy](screenshots/m2-unlocked-pdf2-reddy.png)](screenshots/m2-unlocked-pdf2-reddy.png)
[![Unlocked — Pathology Report, E. Thompson](screenshots/m2-unlocked-pdf3-thompson.png)](screenshots/m2-unlocked-pdf3-thompson.png)

> ⚠️ **Security relevance:** All three passwords — a top-10 common password, a dictionary word, and a short symbol string — fell to an off-the-shelf wordlist in seconds. PDF encryption offers no real protection when paired with weak, guessable passwords.

---

# 6. M3 — Staff Salary & Shareholder Data Exposure

## 6.1. Staff Portal — Contrasting Result

Before relying on the `/old/` backup, the Staff Login form (`/staff/login.php`) was tested with the same `admin'--` SQL-injection payload used successfully against the Patient Portal. It was correctly rejected with a generic **"Invalid username or password"** message, confirming this form — unlike the Patient Portal — properly sanitises its input.

[![Staff login correctly rejects the same SQLi payload](screenshots/m3-staff-login-sqli-invalid.jpg)](screenshots/m3-staff-login-sqli-invalid.jpg)

A guessed `staff/index.php` path was also probed directly and consistently returned `404`, and the `/staff/` directory listing confirmed only `login.php` exists server-side — there is no separate dashboard, payroll, or shareholders page to find:

[![staff/index.php consistently returns 404](screenshots/m3-staff-index-404.jpg)](screenshots/m3-staff-index-404.jpg)
[![Index of /staff/ — only login.php present](screenshots/m3-staff-directory-listing-login-only.jpg)](screenshots/m3-staff-directory-listing-login-only.jpg)

> No live staff dashboard exists on the server. The true exposure for this milestone is the database backup identified below.

## 6.2. The Real Exposure — `mediroza_db_backup_2019.sql`

The SQL backup file downloaded from the misconfigured `/old/` directory (Section 4.3) contains two complete tables: `staff` and `shareholders`. `grep` was used against the raw file to cleanly isolate each table's structure and data for this report.

**`staff` table** — full employee roster including names, job titles, departments, emails, phone numbers, **salaries**, and hire dates:

[![CREATE TABLE staff + dump header](screenshots/m3-db-backup-staff-table-structure.jpg)](screenshots/m3-db-backup-staff-table-structure.jpg)
[![staff table rows — names, salaries, contact details](screenshots/m3-db-backup-staff-table-rows.jpg)](screenshots/m3-db-backup-staff-table-rows.jpg)

**`shareholders` table** — 10 shareholder records including names, share percentage, shares held, and share class:

[![shareholders table rows](screenshots/m3-db-backup-shareholders-table.jpg)](screenshots/m3-db-backup-shareholders-table.jpg)
[![Full INSERT INTO shareholders via grep, including all 10 records](screenshots/m3-db-backup-shareholders-full-grep.jpg)](screenshots/m3-db-backup-shareholders-full-grep.jpg)

> ⚠️ **Security relevance:** A single forgotten backup file, left in a publicly browsable `/old/` directory with no authentication, exposes the hospital's **entire internal payroll and ownership structure** — a far more damaging disclosure than the patient PDFs in M1, since it affects every member of staff and every shareholder on record, not just three patients.

---

# 📑 7. Results Summary

| **#** | **Milestone** | **Finding**                                   | **Evidence**                                                     | **Outcome**                          |
| ----- | ------------- | ----------------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| 1     | M1            | SQLi login bypass (`admin'--`) on Patient Portal | Logged into `portal.php` with no valid credentials                   | 3 encrypted PDFs retrieved                 |
| 2     | M1            | Exposed `/old/` directory listing                | `mediroza_db_backup_2019.sql` downloaded via `wget`                  | Full DB backup obtained                    |
| 3     | M2            | `patient_report_1.pdf` password cracked          | `pdfcrack`                                                            | `123456`                                   |
| 4     | M2            | `patient_report_2.pdf` password cracked          | `pdfcrack`                                                            | `password`                                 |
| 5     | M2            | `patient_report_3.pdf` password cracked          | `pdfcrack`                                                            | `!@#$%^&`                                  |
| 6     | M3            | Staff/shareholder data exposure                  | `staff` + `shareholders` tables extracted from the SQL backup         | Full payroll & ownership data recovered    |
| 7     | M3            | Staff Portal SQLi test                           | `admin'--` correctly rejected                                         | Not vulnerable (contrast finding)          |

---

# ⛔ 8. Risk Analysis / Impact

| **#** | **Risk / Finding**                                 | **Evidence / Observation**                                               | **⚠️ Potential Impact**                                                                                       | **Risk Level** |
| ----- | ----------------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ----------------- |
| 1     | SQL injection authentication bypass (Patient Portal)   | `admin'--` logged in with no valid credentials                               | Full bypass of authentication; direct access to confidential patient records for any unauthenticated attacker          | 🔴 **Critical**    |
| 2     | Exposed database backup in `/old/`                     | `mediroza_db_backup_2019.sql` publicly downloadable, no auth required        | Complete disclosure of staff salaries, personal contact details, and shareholder ownership structure                   | 🔴 **Critical**    |
| 3     | Directory listing enabled on multiple paths            | `/old/`, `/patient/` both browsable                                          | Attackers can enumerate server files without guessing names, dramatically speeding up reconnaissance                   | 🟡 **Medium**      |
| 4     | Username enumeration on Patient Portal                  | Distinct "Username not found" vs "Incorrect password" messages               | Allows attackers to build a valid-username list before attempting credential or brute-force attacks                    | 🟡 **Medium**      |
| 5     | Weak PDF passwords                                      | `123456`, `password`, `!@#$%^&` — all cracked in seconds with a wordlist     | Encryption on sensitive documents provides no real protection if the password is weak or guessable                     | 🟡 **Medium**      |
| 6     | `robots.txt` discloses sensitive paths                  | `Disallow: /patient/`, `/staff/`, `/old/`                                    | Effectively acts as a roadmap for attackers, pointing directly at the areas the site is trying to hide                 | 🟢 **Low**         |

### Risk Rating Key

| Indicator | Severity     |
| --------- | ------------ |
| 🔴         | **Critical** |
| 🟡         | **Medium**   |
| 🟢         | **Low**      |

> ⚠️ **Note: These findings reflect a controlled educational exercise against a fictional training target, conducted under explicit written authorization. No real patient, staff, or shareholder data was involved.**

---

# 🪛 9. Recommendations

## Based on the observations from this exercise, I recommend the following remediations:

1. **Parameterise all database queries**
   Replace raw string-concatenated SQL in `login.php` with prepared statements/parameterised queries to eliminate the SQL-injection authentication bypass entirely.

2. **Disable directory listing server-wide**
   Set `Options -Indexes` (or the LiteSpeed/Apache equivalent) so that `/old/`, `/patient/`, and every other directory refuse to display a file index to visitors.

3. **Remove or properly secure old/backup files**
   Database backups, archives, and other legacy files should never be stored inside the public web root. If retention is required, store them outside the webroot or behind strong authentication.

4. **Normalise authentication error messages**
   Return the same generic message ("Invalid username or password") for both "unknown username" and "wrong password" cases to prevent username enumeration.

5. **Enforce strong password policies on sensitive documents**
   Patient lab reports should use randomly generated, sufficiently long passwords rather than common words or short symbol strings.

6. **Restrict `robots.txt` disclosures**
   Avoid listing sensitive paths in `robots.txt`; rely on proper server-side access control (authentication, directory restrictions) rather than "hiding" paths from well-behaved crawlers only.

7. **Add rate-limiting and account lockout**
   Both the Patient and Staff login forms should throttle repeated failed attempts to slow down both enumeration and brute-force/injection probing.

8. **Perform security testing with authorization only**
   All penetration-testing exercises should only be performed against systems one owns or has explicit written authorization to test.

---

# 📜 10. Security Assessment Summary

| **Category**                         | **Status**   |
| --------------------------------------- | --------------- |
| 🔎 Reconnaissance (`robots.txt`, `sitemap.xml`, directory probing) | ✔️ Completed |
| 🧩 Username Enumeration                  | ✔️ Confirmed    |
| 💉 SQL Injection — Patient Portal        | ✔️ Confirmed & exploited |
| 💉 SQL Injection — Staff Portal          | ✔️ Tested — not vulnerable |
| 📂 Directory Listing Misconfiguration    | ✔️ Confirmed    |
| 🔑 PDF Password Cracking (M2)            | ✔️ Completed — 3/3 recovered |
| 🗄️ Database Backup Exposure (M3)         | ✔️ Confirmed & extracted |
| 🖥️ Hidden Staff Dashboard Search         | ✔️ Checked — none found |

---

# 🔚 11. Conclusion

During Week 4 of my Cybersecurity & Ethical Hacking internship, I carried out an authorized black-box penetration test against Mediroza General Hospital's training website, `medirozahospital.com`, across three milestones.

Reconnaissance via `robots.txt` pointed directly at the `/patient/`, `/staff/`, and `/old/` directories. A username-enumeration flaw and a classic SQL-injection payload (`admin'--`) bypassed the Patient Portal's login entirely, granting access to three password-protected pathology PDFs (M1). All three PDF passwords were subsequently recovered using `pdfcrack` after John the Ripper failed to load its own extracted hashes (M2). Finally, the misconfigured `/old/` directory was found to expose a complete SQL database backup containing the hospital's full staff payroll and shareholder ownership records — a far more significant exposure than the patient PDFs alone (M3). By contrast, the Staff Portal's login form correctly rejected the same SQL-injection payload, demonstrating that proper input sanitisation does prevent this class of attack when implemented.

This engagement reinforced that a single misconfiguration — directory listing left enabled on a forgotten backup file — can expose far more sensitive data than a targeted attack against a single form, and that authentication bypass vulnerabilities remain just as dangerous today as when SQL injection was first documented decades ago.

---

## 📌 Project Information

| **Project Details**   | **Information**                                          |
| ---------------------- | ------------------------------------------------------------ |
| 1️⃣ **Program Name**    | Cybersecurity Program at Networkwalks                        |
| 2️⃣ **Week**            | 04                                                             |
| 3️⃣ **Module**          | W4-PM — Authorized Black-Box Web Application Penetration Test |
| 4️⃣ **Project:**       | Mediroza General Hospital Pentest (M1–M3: PDF Retrieval, Password Cracking, Data-Exposure Discovery) |
| 5️⃣ **Repository:**    | GitHub                                                        |

---

# 👤 Author

**Oyeru Rachael Emalohi**

**Cybersecurity Intern | Networkwalks | Batch B084D**

🔗 [LinkedIn](https://www.linkedin.com/in/oyeru-rachael-067a39253)

---
