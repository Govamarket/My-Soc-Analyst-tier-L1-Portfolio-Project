# Evidence Directory

This directory contains evidence for the investigations in this repository.

## SSH Authentication Investigation

**Report:** [SSH Authentication Investigation](../Ssh%20authentication%20investigation)

1. [01 — `/var/log` listing](01-var-log-listing.png)
2. [02 — SSH failed logins in the journal](02-journalctl-ssh-failed-logins.png)
3. [03 — `auth.log` command audit trail](03-auth-log-command-audit-trail.png)
4. [04 — `auth.log` sudo and SSH service events](04-auth-log-sudo-and-ssh-service-events.png)
5. [05 — Failed-password search self-match](05-grep-failed-password-self-match.png)
6. [06 — `last` output with no SSH sessions](06-last-output-no-ssh-sessions.png)

## Email Security and Windows Installer Triage

**Report:** [Email Security Analysis and Windows Installer Triage](../Email%20Security%20Analysis%20and%20Windows%20Installer%20Triage.md)

Evidence supplied for the report consists of four screenshots:

1. DKIM review for `amazonses.com`
2. DKIM review for `news.kaisar.io`
3. SPF review for `mail.news.kaisar.io`
4. DMARC review for `dmarc.news.kaisar.io`

The original screenshots should be placed in this directory with descriptive filenames before they are embedded in the report.

---

Keep each investigation in a separate report and link supporting evidence by descriptive filename.
