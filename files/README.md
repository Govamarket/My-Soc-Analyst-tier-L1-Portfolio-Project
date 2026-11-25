# Files and Evidence Index

This directory contains investigation reports and their supporting evidence. The order below follows the investigation workflow; original filenames are preserved.

## 1. SSH authentication investigation

**Report:** [SSH Authentication Investigation](ssh-authentication-investigation.md)

**Evidence, in report order:**

1. [01 — `/var/log` listing](01-var-log-listing.png)
2. [02 — SSH failed logins in the journal](02-journalctl-ssh-failed-logins.png)
3. [03 — `auth.log` command audit trail](03-auth-log-command-audit-trail.png)
4. [04 — `auth.log` sudo and SSH service events](04-auth-log-sudo-and-ssh-service-events.png)
5. [05 — Failed-password search self-match](05-grep-failed-password-self-match.png)
6. [06 — `last` output with no SSH sessions](06-last-output-no-ssh-sessions.png)

## 2. Email security and Windows installer triage

**Report:** [Email Security Analysis and Windows Installer Triage](Email%20Security%20Analysis%20and%20Windows%20Installer%20Triage.md)

The following image files are also present. Their filenames alone do not identify which findings or report sections they support, so they are listed without assigning interpretations:

- [MAI.PNG](MAI.PNG)
- [amdp.PNG](amdp.PNG)
- [h1.PNG](h1.PNG)
- [h2.PNG](h2.PNG)
- [h3.PNG](h3.PNG)
- [h4.PNG](h4.PNG)
- [h5.PNG](h5.PNG)
- [kk.PNG](kk.PNG)
- [pass.PNG](pass.PNG)

## 3. Entries needing a filename check

These names were included in the supplied inventory, but their purpose or exact file type is unclear. Confirm whether they are duplicates, old drafts, or supporting material before renaming or moving them:

- `Ssh authentication investigation`
- `README.md`

When possible, use descriptive, lowercase filenames and number screenshots in the order they appear in the report. Update report image links whenever files are moved or renamed.
