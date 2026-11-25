# Email Authentication Analysis & Windows Installer Triage

**Analyst:** Clinton Chidera | **Date:** 25 Nov 2026
**Focus:** Email security, phishing analysis, threat intelligence, malware triage

A short, evidence-based investigation of (1) a "legacy rewards" email from Kaisar and (2) the Kaisar Node Windows installer linked to it. **The installer was never executed.**

---

## Summary

| Area | Finding | Verdict |
|---|---|---|
| Kaisar email authentication | SPF, DKIM, DMARC all pass, with aligned domains | Likely legitimate |
| Installer hash | SHA-256 matches the value on the official Kaisar downloads page | Matches publisher |
| VirusTotal | 2/70 vendors flag it (ML-based detections) | Inconclusive |
| Code signing | No Authenticode signature, but publisher discloses this and uses Minisign | Consistent with disclosure |
| Local Minisign / Defender checks | Not yet completed | Pending |

**Conclusion:** the evidence does not justify calling the installer definitively malicious or completely safe. Local signature verification is still required.

---

## 1. Email Header Analysis (Kaisar)

**From:** `hello@news.kaisar.io` | **Reply-To:** `support@kaisar.io`
**Subject:** Your legacy rewards are ready in Kaisar | **Sent via:** Amazon SES (`23.251.232.3`)

| Check | Result | Evidence |
|---|---|---|
| SPF | Pass | Return-Path `mail.news.kaisar.io` authorises `23.251.232.3`. Record: `v=spf1 include:amazonses.com -all` (hard fail) |
| DKIM (`d=news.kaisar.io`) | Pass, aligned | Domain matches the From header |
| DKIM (`d=amazonses.com`) | Pass, not aligned | Second Amazon signature, which is normal for SES |
| DMARC | Pass | `p=quarantine; adkim=s; aspf=r; pct=100`, reports to `dmarc@kaisar.io` |

**Notes**

- The DKIM checker flagged "signature domain not aligned" for the `amazonses.com` signature. This is not a failure: DMARC needs only **one** aligned mechanism, and the `news.kaisar.io` signature satisfies it.
- A DMARC policy of `quarantine` at 100% means spoofed mail from this domain should be quarantined.
- The email also includes a one-click `List-Unsubscribe` header, consistent with legitimate bulk mail.
- The visible link goes to `https://app.kaisar.io/` via an Amazon SES click-tracking redirect. The redirect should still be inspected rather than trusted on appearance.

**Limits:** passing authentication proves who *sent* the mail, not that its claims (reward balance, token value, payouts) are true.

### Second sample: ALX Africa Community CV-review email

Sent via Mailgun from `notification.circle.so`, which is not `alxafrica.com`. SPF, DKIM and DMARC passed for the sending domain, but the DMARC policy was `p=none` (monitoring only). Assessed as likely legitimate community mail. Sender identity needs context and it should not be classified as phishing on this evidence alone.

---

## 2. Installer Triage: `Kaisar.Node_2.1.2_x64-setup.exe`

| Attribute | Value |
|---|---|
| Platform / version | Windows x64 / 2.1.2 |
| Size | ~3.53 MB |
| SHA-256 | `0f930057d68140f374f536becd2818ddf403519a2ae52a2b132e8f02e21f4cdf` |
| Executed | **No** |

### VirusTotal

- **Detections: 2/70.** SecureAge (`Malicious`) and Trapmine (`Malicious.high.ml.score`). Both are generic or machine-learning style labels.
- **Tags:** `peexe`, `overlay`, `detect-debug-environment`, `obfuscated`
- **Sigma matches:**
  - *Sysmon File Executable Creation Detected* (Medium): generic, depends on Sysmon config.
  - *Load of RstrtMgr.DLL by an uncommon process* (Low): Restart Manager is commonly used by installers, though it has also been seen in ransomware and wipers.

These are reasons to look closer, not proof of malware. Obfuscation, anti-debugging and overlays also occur in legitimate packagers and installers.

### Publisher verification

- The official downloads page lists v2.1.2 with the same SHA-256 as above, so **the hash matches**.
- The page states the release is **not code-signed for Windows/macOS yet**. Files are instead signed with the Kaisar release key (Minisign), with SHA-256 values published.
- Publisher guidance: download only from `kaisar.io` and compare hashes. A file from anywhere else is not from Kaisar.
- An Authenticode status of `NotSigned` is therefore expected. It is not the same as a failed Minisign check.

Sources: <https://kaisar.io/downloads> | <https://kaisar.io/docs/node/install> | <https://kaisar.io/security>

---

## 3. Local Verification Steps

```powershell
# 1. Hash
Get-FileHash "$env:USERPROFILE\Downloads\Kaisar.Node_2.1.2_x64-setup.exe" -Algorithm SHA256

# 2. Minisign (download the .minisig from the official page into the same folder)
winget install jedisct1.minisign
minisign -Vm "$env:USERPROFILE\Downloads\Kaisar.Node_2.1.2_x64-setup.exe" -P "RWQZTGW5zf2SxIa2AoOHPrzNxzVB98P7Abqe9ZeSzMMWzZbjH5CUHdig"
# Expected: "Signature and comment signature verified"

# 3. Authenticode status
Get-AuthenticodeSignature "$env:USERPROFILE\Downloads\Kaisar.Node_2.1.2_x64-setup.exe" | Format-List Status,StatusMessage,SignerCertificate

# 4. Defender health, then 5. custom scan
Get-MpComputerStatus | Select-Object AMServiceEnabled,AntivirusEnabled,RealTimeProtectionEnabled,AntivirusSignatureLastUpdated
Start-MpScan -ScanType CustomScan -ScanPath "$env:USERPROFILE\Downloads\Kaisar.Node_2.1.2_x64-setup.exe"
```

**Note:** `winget install minisign` returned *multiple packages matching* (`jedisct1.minisign` and `aead.minisign`). Specify the ID explicitly as above. `jedisct1` is the original Minisign author. Confirm the project page before trusting any package.

### Status

- [x] Hash matches the official page
- [ ] Local SHA-256 recomputed
- [ ] Minisign signature verified
- [ ] Authenticode status recorded
- [ ] Defender scan result recorded

No successful signature verification or clean Defender result is claimed yet.

---

## 4. Lessons Learned

1. Display names prove nothing. Read the actual headers.
2. SPF, DKIM and DMARC authenticate the sender and its alignment, not the truth of the message.
3. Inspect tracking redirects and final destinations.
4. Read a VirusTotal ratio in context: look at *which* vendors flagged it, whether the labels are generic ML, and whether the hash matches the publisher's.
5. A matching hash plus a verified signature is far stronger evidence than a filename or a download source.
6. An unsigned file is not automatically malicious when another signing method is documented, and it is not automatically safe either.
7. Don't run suspicious executables just to see what happens.

## 5. Limitations

Not done: Minisign verification, Defender scan result, static code analysis, sandboxed dynamic analysis, network traffic capture, and independent verification of Kaisar's reward or token claims.

---

*Independent educational investigation. This is not an endorsement of the software, a formal malware verdict, or an audit of the publisher's security claims. No executable was run.*
