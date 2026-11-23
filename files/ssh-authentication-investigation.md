# SSH Authentication Investigation Report

**Environment:** Kali Linux 2026.2 (VirtualBox lab)
**Investigation:** SSH authentication monitoring
**Investigator:** Clinton Chidera
**Date:** 23 November 2026 (system timezone UTC-05:00)

**Log sources**

- `/var/log/auth.log` (via rsyslog)
- systemd journal (`journalctl`)
- SSH service logs (`journalctl -u ssh`)
- `wtmp` login records (`last`)

---

## 1. Objective

Investigate successful and unsuccessful SSH authentication attempts on a local Kali Linux machine, and understand where those events are recorded on a modern Debian-based system.

## 2. Background: why `auth.log` was missing

Modern Kali uses **systemd-journald** as its default logger and does not ship with **rsyslog**. Without rsyslog, the traditional text files (`/var/log/auth.log`, `/var/log/syslog`) are never created. The `/var/log` listing showed a `journal` directory and no `auth.log`, which confirmed this.

![/var/log listing on a fresh Kali install: a journal directory, but no auth.log or syslog](images/01-var-log-listing.png)
*Figure 1: `/var/log` before rsyslog was installed. There is a `journal` directory but no `auth.log`.*

Two related points:

- `/var/log/secure` is the equivalent file on **Red Hat-family** systems (RHEL, CentOS, Fedora). It does not exist on Debian-based systems like Kali.
- `journalctl -u ssh` returned `-- No entries --` because the SSH service had never been started, so it had produced no logs.

## 3. Activities performed

### 3.1 Install and enable rsyslog

```bash
sudo apt update
sudo apt install rsyslog
sudo systemctl enable --now rsyslog
sudo systemctl status rsyslog
```

`rsyslog` was enabled and started at **05:57:27** and verified with `systemctl status rsyslog` at **05:57:57**. `/var/log/auth.log` was populated from this point onward.

### 3.2 Check, enable and start the SSH service

```bash
sudo systemctl status ssh      # 06:08:44 - checked initial state
sudo systemctl enable ssh      # 06:09:15 - start on boot
sudo systemctl start ssh       # 06:10:34 - start now
sudo systemctl status ssh      # 06:10:46 - verified running
sudo ss -tlnp                  # confirmed listening sockets
```

### 3.3 Monitor logs live

```bash
sudo journalctl -u ssh -f
sudo tail -f /var/log/auth.log
```

### 3.4 Generate controlled failed logins

From a second terminal, I connected to the local SSH server and deliberately entered an incorrect password:

```bash
ssh kali@localhost
```

### 3.5 Examine the records

```bash
sudo journalctl -u ssh
sudo grep "sudo" /var/log/auth.log
sudo grep "failed password" /var/log/auth.log
sudo grep "successful login" /var/log/auth.log
sudo last
sudo lastb
```

## 4. Evidence collected

| Item | Observation |
|---|---|
| **SSH service status** | Enabled at 06:09:15 and started at 06:10:34. `sshd[73861]` listening on `0.0.0.0:22` and `[::]:22` (IPv4 and IPv6). |
| **Source IP address** | `::1` (IPv6 loopback, i.e. the same machine) |
| **Target username** | `kali` |
| **Failed login events** | 2 failed password attempts (06:13:49 and 06:13:57) |
| **Successful SSH login events** | See section 6 (not captured in this evidence set) |
| **Source port / session** | Port `35738`, `sshd-session[75183]` (both failures came from the same TCP connection) |

### 4.1 SSH service startup (journal)

```
Nov 23 06:10:34 kali systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
Nov 23 06:10:34 kali sshd[73861]: Server listening on 0.0.0.0 port 22.
Nov 23 06:10:34 kali sshd[73861]: Server listening on :: port 22.
Nov 23 06:10:34 kali systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
```

### 4.2 Failed authentication (journal)

```
Nov 23 06:13:47 kali unix_chkpwd[75516]: password check failed for user (kali)
Nov 23 06:13:47 kali sshd-session[75183]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=::1  user=kali
Nov 23 06:13:49 kali sshd-session[75183]: Failed password for kali from ::1 port 35738 ssh2
Nov 23 06:13:55 kali unix_chkpwd[75581]: password check failed for user (kali)
Nov 23 06:13:57 kali sshd-session[75183]: Failed password for kali from ::1 port 35738 ssh2
Nov 23 06:15:07 kali sshd[73861]: Timeout before authentication for connection from ::1 to ::1, pid = 75183
```

![journalctl -u ssh -f output showing the service starting and two failed password attempts](images/02-journalctl-ssh-failed-logins.png)
*Figure 2: `journalctl -u ssh -f` showing service startup, two failed logins and the pre-authentication timeout.*

### 4.3 Timeline

| Time (UTC-5) | Event |
|---|---|
| 05:57:27 | rsyslog enabled and started |
| 06:09:15 | SSH enabled |
| 06:10:34 | SSH started, listening on port 22 |
| 06:13:47 | First password check failure (PAM / `unix_chkpwd`) |
| 06:13:49 | `Failed password for kali from ::1 port 35738` |
| 06:13:55 | Second password check failure |
| 06:13:57 | `Failed password for kali from ::1 port 35738` |
| 06:15:07 | Connection timed out before authentication completed |

### 4.4 auth.log evidence after rsyslog was enabled

Once rsyslog was running, `auth.log` recorded the service changes (enable, start, status) and every privileged command I ran:

![auth.log entries showing rsyslog being enabled, then ssh being checked, enabled and started](images/04-auth-log-sudo-and-ssh-service-events.png)
*Figure 3: `grep "sudo" /var/log/auth.log` showing the rsyslog and SSH service commands with timestamps.*

![auth.log entries showing later commands including ss -tlnp and grep searches](images/03-auth-log-command-audit-trail.png)
*Figure 4: Later `auth.log` entries, including `ss -tlnp` and my own `grep` searches being logged.*

## 5. Findings

1. **Logging gap explained.** `auth.log` was absent because rsyslog is not installed by default on current Kali. After installing and enabling it, authentication events were written to `/var/log/auth.log` in addition to the journal.
2. **SSH was inactive by default.** The empty `journalctl -u ssh` output reflected a stopped service, not a logging fault.
3. **Failed logins were fully recorded.** Each failed attempt produced two correlated entries: a PAM/`unix_chkpwd` record and an sshd `Failed password` record. Together they give the username, source address, source port and protocol.
4. **Both failures came from one connection.** The same source port (`35738`) and process ID (`75183`) appear on both attempts, so they were two password tries within a single TCP session, not two separate connections.
5. **Pre-authentication timeout.** After the second failure the session sat idle until sshd logged `Timeout before authentication` at 06:15:07. This is consistent with sshd's default `LoginGraceTime` of 120 seconds.
6. **No suspicious activity.** The source was `::1` (loopback), the account was `kali`, and the volume was two attempts, all matching the controlled test I performed. This does not resemble brute-forcing, where you would expect many attempts, many connections, varying usernames and external source IPs.

### Lessons learned while searching the logs

- **`grep` is case-sensitive.** sshd writes `Failed password` with a capital F. My search for `"failed password"` matched only the `sudo` lines that recorded my own grep commands. Use `grep -i` or the exact capitalisation.
- **Searching auth.log pollutes auth.log.** Every `sudo grep ...` writes a new line containing the search string, so later searches match earlier searches. Exclude these with `grep -v COMMAND`.
- **"successful login" is not a string sshd ever writes.** Successful SSH logins appear as `Accepted password for ...` or `Accepted publickey for ...`.
![grep for failed password and successful login returning only sudo command lines](images/05-grep-failed-password-self-match.png)
*Figure 5: Both searches matched only the `sudo` lines that logged my own grep commands.*

- **`lastb` is not installed** on this build (`command not found`). Failed logins are available from the journal and `auth.log` instead.
- **`last` showed only local desktop sessions** (`tty7` / `:0` via lightdm), not SSH sessions, which means no completed SSH login had been recorded in `wtmp` at that point.

![last output showing only tty7 desktop sessions and no SSH pts sessions](images/06-last-output-no-ssh-sessions.png)
*Figure 6: `last` shows desktop sessions on `tty7`, with no SSH sessions yet. The `lastb` command was not found.*

## 6. Completing the successful-login evidence

A successful SSH login produces an `Accepted ...` line. To capture it, log in with the correct password and search for it:

```bash
ssh kali@localhost                                   # enter the correct password
sudo grep "Accepted" /var/log/auth.log
sudo journalctl -u ssh | grep "Accepted"
last | grep pts                                      # SSH sessions appear on pts/N
```

Expected format:

```
Accepted password for kali from ::1 port <port> ssh2
pam_unix(sshd:session): session opened for user kali(uid=1000)
```

## 7. Useful detection commands

```bash
# Failed logins (case-insensitive), excluding my own sudo/grep commands
sudo grep -i "failed password" /var/log/auth.log | grep -v COMMAND

# Successful logins
sudo grep "Accepted" /var/log/auth.log

# Top source IPs of failed logins
sudo grep -i "failed password" /var/log/auth.log | grep -v COMMAND \
  | grep -oE "from [0-9a-f:.]+" | sort | uniq -c | sort -rn

# Same data from the journal, no rsyslog needed
sudo journalctl -u ssh --since today | grep -i "failed"
sudo journalctl _COMM=sshd -p warning
```

## 8. Conclusion

The investigation confirmed that modern Kali records SSH authentication through **journald** by default, and that installing **rsyslog** restores the traditional `/var/log/auth.log`. After enabling SSH and generating a controlled failed login, the full event chain was visible in both log sources, with the source address (`::1`), target user (`kali`), timestamps and session details. No malicious activity was identified.

### Recommended follow-up

- **Complete the success evidence:** capture an `Accepted` login (section 6) and add it to the evidence table.
- **Harden SSH** in `/etc/ssh/sshd_config`: `PermitRootLogin no`, `PasswordAuthentication no` (use key-based authentication), `MaxAuthTries 3`, and `AllowUsers kali`.
- **Install fail2ban** to automatically ban sources with repeated failures.
- **Disable SSH when not needed:** `sudo systemctl disable --now ssh`.
- **Extend the lab:** repeat the test from a second VM so the source IP is a real LAN address, and write a small script to alert on repeated `Failed password` entries.
- **Retain logs:** check `logrotate` settings so authentication history is not lost.

---

*All activity was performed on my own isolated virtual machine for learning purposes.*
