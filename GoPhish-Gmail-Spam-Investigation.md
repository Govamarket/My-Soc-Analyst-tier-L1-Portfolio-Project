# Email Security Investigation: GoPhish, Gmail SMTP and Spam Classification

**Author:** Clinton Chidera  
**Category:** Email Security Analysis | SOC Analyst Tier 1  
**Environment:** Kali Linux, GoPhish, Gmail SMTP and Gmail  
**Investigation Date:** 27 November 2026  
**Status:** Investigation completed; spam classification cause undetermined

---

## 1. Investigation Overview

As part of my SOC Analyst Tier 1 practical exercises, I conducted an email security investigation using GoPhish and Gmail SMTP.

The objective was to configure GoPhish to send an authorised security awareness simulation email, inspect its delivery behaviour, analyse email authentication results and investigate why Gmail classified the message as Spam despite successful SPF, DKIM and DMARC authentication.

I used two Gmail accounts under my control:

- **Sender:** ismeclinton@gmail.com
- **Recipient:** dapplabx@gmail.com

All testing was conducted within my personal laboratory environment.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Operating System | Kali Linux |
| Simulation Platform | GoPhish |
| SMTP Provider | Gmail SMTP |
| SMTP Host | smtp.gmail.com |
| SMTP Port | 587 |
| Sender Account | ismeclinton@gmail.com |
| Recipient Account | dapplabx@gmail.com |
| Email Authentication | SPF, DKIM and DMARC |
| Analysis Method | Gmail Show Original / Email Header Analysis |

## 3. Investigation Objectives

1. Configure GoPhish to send an authorised security awareness email.
2. Verify successful SMTP submission.
3. Inspect SPF, DKIM and DMARC authentication results.
4. Examine email routing and message headers.
5. Investigate Gmail Spam classification.
6. Compare a GoPhish-generated email against a manually composed Gmail email.

## 4. GoPhish Configuration and Email Simulation

I configured a Gmail SMTP sending profile in GoPhish.

The sending profile used:

- SMTP server: smtp.gmail.com
- Port: 587
- Sender: ismeclinton@gmail.com

I created a custom HTML email template titled:

**Security Awareness Test – Account Verification**

The message clearly identified itself as an authorised security awareness exercise and instructed the recipient to report the message through an approved security reporting channel.

I also configured a campaign using my own test recipient account.

## 5. Email Header Analysis

After receiving the GoPhish email, I opened Gmail's Show Original feature to inspect the message headers.

### 5.1 Authentication Results

| Authentication Mechanism | Result |
|---|---|
| SPF | PASS |
| DKIM | PASS |
| DMARC | PASS |

The authentication results included:

```text
spf=pass
dkim=pass
dmarc=pass
```

The DMARC policy information was:

```text
dmarc=pass (p=NONE sp=QUARANTINE dis=NONE)
```

This indicates that DMARC authentication passed, while the primary domain policy was configured for monitoring rather than quarantine or rejection.

### 5.2 SMTP Routing Analysis

The relevant headers included:

```text
Received: from kali ([105.112.225.147])
by smtp.gmail.com
```

This showed that my Kali Linux machine submitted the email through Gmail SMTP.

The receiving infrastructure showed:

```text
Received: from mail-sor-f65.google.com
[209.85.220.65]
by mx.google.com
```

The message was subsequently processed by Google's receiving infrastructure.

### 5.3 GoPhish Identification

The email contained:

```text
X-Mailer: gophish
```

This identified GoPhish as the sending application.

The message also contained a tracking element:

```html
<img alt='' style='display: none'
src='/track?rid=pm4CNk3'/>
```

This is associated with GoPhish email-open tracking.

## 6. Spam Classification Investigation

Despite passing all three authentication checks, Gmail delivered the GoPhish email to the Spam folder.

I initially suspected that GoPhish's email configuration or message construction might have contributed to the classification.

To investigate this hypothesis, I performed a controlled comparison.

### 6.1 Test One: GoPhish-Generated Email

- Sender: ismeclinton@gmail.com
- Recipient: dapplabx@gmail.com
- Delivery location: Spam
- SPF: PASS
- DKIM: PASS
- DMARC: PASS

### 6.2 Test Two: Manually Composed Gmail Email

I sent a separate, manually composed email from the same Gmail account to the same recipient.

The message was a simple plain-text email without GoPhish tracking.

Results:

- Sender: ismeclinton@gmail.com
- Recipient: dapplabx@gmail.com
- Delivery location: Spam
- SPF: PASS
- DKIM: PASS
- DMARC: PASS

### 6.3 Comparative Findings

| Indicator | GoPhish Email | Normal Gmail Email |
|---|---|---|
| Sender | Same | Same |
| Recipient | Same | Same |
| Sending IP | 209.85.220.65 | 209.85.220.65 |
| SPF | PASS | PASS |
| DKIM | PASS | PASS |
| DMARC | PASS | PASS |
| Delivery | Spam | Spam |
| X-Mailer | gophish | Not present |
| Content Type | text/html | multipart/alternative |

The comparison showed that both messages were classified as Spam despite differences in their message construction.

This means the available evidence does not establish GoPhish as the sole cause of the Spam classification.

## 7. Additional Checks

I inspected the recipient Gmail account's filter settings.

- No custom filters were configured.
- No sender-specific filter was identified.

However, this does not rule out Gmail's other spam-filtering mechanisms or account-level reputation signals.

## 8. Investigation Findings

The investigation established the following:

1. GoPhish successfully submitted the email through Gmail SMTP.
2. Gmail's receiving infrastructure accepted the message.
3. SPF, DKIM and DMARC authentication passed.
4. The email contained GoPhish identification and tracking elements.
5. A manually composed email from the same sender also landed in Spam.
6. The recipient's custom Gmail filters were empty.
7. The exact reason for Gmail's Spam classification remains undetermined.

## 9. Security Lessons Learned

This investigation reinforced several important email security concepts:

- SPF validates whether a sending server is authorised for the evaluated sender domain.
- DKIM verifies the cryptographic signature of an email.
- DMARC evaluates authentication alignment against the visible From domain and applies the published policy.
- Successful email authentication does not guarantee inbox placement.
- Email deliverability and email authentication are separate areas of investigation.
- Email headers provide valuable evidence for identifying mail routing, sending infrastructure and email clients.

## 10. Conclusion

The investigation successfully demonstrated GoPhish SMTP configuration, email header inspection and SPF, DKIM and DMARC verification.

Although both the GoPhish-generated email and the manually composed Gmail email passed authentication, both were classified as Spam.

The findings demonstrate that authentication success alone is insufficient to explain email deliverability.

Further investigation into Gmail's spam classification signals, sender reputation and recipient-side filtering behaviour is required before identifying the underlying cause.

**Final Investigation Status:** Authentication verified; SMTP delivery verified; Spam classification cause undetermined.

---

**Tools Used:** Kali Linux | GoPhish | Gmail SMTP | Gmail Show Original | MXToolbox
