# GoPhish / Gmail Spam Investigation — MITRE ATT&CK Mapping

**Investigation:** GoPhish, Gmail SMTP and Spam Classification  
**Date:** 27 November 2026  
**Scope:** Authorized personal lab; two investigator-controlled Gmail accounts  
**Disposition:** Informational / low-risk; Spam classification cause undetermined

## Mapping principle

This mapping describes the *defensive and authorized-simulation context* of the test. The report does not establish phishing activity, malicious intent, compromise, or victim execution. Techniques below are relevant only if their supporting behavior was observed or intentionally configured in the exercise.

| ATT&CK technique | ID | Tactic | Evidence in this exercise | Assessment |
|---|---|---|---|---|
| Spearphishing Link | T1566.002 | Initial Access | An awareness-simulation HTML message containing verification/account-themed wording was sent. The supplied record mentions GoPhish tracking, but does not confirm a link-click event. | Contextual / not confirmed as malicious |
| User Execution: Malicious Link | T1204.001 | Execution | The template was part of an authorized awareness simulation and may have included a link. No recipient endpoint telemetry or click-confirmation evidence is supplied. | Potential simulation behavior only; not observed |
| Phishing for Information: Spearphishing Link | T1598.003 | Reconnaissance | The simulation included an authorized awareness-test flow. No evidence was supplied that recipient credentials or identity information were collected. | Contextual only; no reconnaissance claim |
| Web Protocols | T1071.001 | Command and Control | The GoPhish tracking element used a tracked web path, but no post-delivery C2 activity was reported. | No evidence of C2 |

## Observed GoPhish and mail artifacts

- `X-Mailer: gophish` — application-identification marker in the test message.
- `/track?rid=[redacted]` — GoPhish-style tracking path.
- `smtp.gmail.com`, port 587 — authorized Gmail SMTP submission path.
- SPF, DKIM and DMARC PASS — authentication evidence for both compared messages.
- Both the GoPhish message and a manually composed message reached the Spam folder.

These artifacts are not sufficient to attribute an attack or determine Gmail's internal Spam-classification logic.

## ATT&CK sequence for the exercise

```text
Authorized simulation setup
        |
        v
T1566.002 — simulated phishing email delivery
        |
        v
T1204.001 — possible user-link interaction (unconfirmed)
        |
        v
Email and tracking evidence recorded for defensive analysis
```

## Detection and analyst guidance

- Correlate mail gateway, Gmail headers, tracking clicks, endpoint events, identity-provider logs and DNS activity before classifying a message as a campaign.
- Preserve original headers and screenshots; redact mailbox addresses, IP addresses and unique tracking identifiers from public reports.
- Treat passing SPF, DKIM and DMARC as authentication outcomes, not proof of legitimacy or maliciousness.
- Do not use Spam-folder placement alone as evidence of phishing or compromise.
- Compare controlled messages against normal mail to reduce false assumptions about GoPhish-specific causes.

## Confidence

- **Technique T1566.002:** Low confidence for attack attribution; moderate confidence that the message was intentionally designed as an awareness-simulation phishing email.
- **Technique T1204.001:** Not assessed; no click or endpoint evidence supplied.
- **Technique T1598.003:** No evidence of successful credential or information collection.
- **Technique T1071.001:** No C2 activity established.

## Closure

No confirmed malicious activity or ATT&CK intrusion was identified. The mapping is for portfolio and defensive-learning purposes and should not be presented as proof of adversary behavior.
