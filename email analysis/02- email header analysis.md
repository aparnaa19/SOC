# Topic 4: Email Header Analysis- Detailed Notes

---

## What Are Email Headers?

Every email has two parts:

```
┌─────────────────────────────────┐
│           HEADERS               │
│  Metadata- routing, servers,   │
│  authentication, timestamps     │
├─────────────────────────────────┤
│             BODY                │
│  What the user actually reads   │
└─────────────────────────────────┘
```

Think of headers like the **outside of a postal envelope:**
- Stamps, postmarks, routing labels
- All the behind-the-scenes delivery info
- The recipient only sees the letter inside (body)

### Why headers matter for security:
- The **body** can be completely faked- easy for attackers
- The **headers** contain technical routing data that's much harder to fake
- Headers reveal the **true origin** of an email even if the From address is spoofed

---

## What's Inside Email Headers

Every raw email header contains these key fields:

| Field | What it contains |
|---|---|
| `Received:` | Every server the email passed through |
| `From:` | Visible sender (can be faked) |
| `Reply-To:` | Where replies actually go (often different in phishing) |
| `To:` | Recipient |
| `Subject:` | Email subject |
| `Date:` | When it was sent |
| `Message-ID:` | Unique ID assigned to this email |
| `DKIM-Signature:` | Digital signature for verification |
| `Authentication-Results:` | SPF, DKIM, DMARC pass/fail results |
| `X-Originating-IP:` | IP address of original sender |
| `Return-Path:` | Where bounced emails go (reveals true sender domain) |

---

## How to Get Raw Headers

### Gmail:
```
Open email
→ Click 3 dots (⋮) top right of email
→ Click "Show original"
→ Full raw headers appear
```

### Outlook:
```
Open email
→ File
→ Properties
→ "Internet headers" box at bottom
```

### Thunderbird:
```
Open email
→ View
→ Message Source (Ctrl + U)
```

---

## The Received Chain- Most Important Part

### What it is:
Every time an email passes through a mail server, that server **stamps itself** onto the email as a `Received:` header. This creates a chain of every server the email touched.

### Critical reading rule:
> **Always read Received headers BOTTOM TO TOP**
> - Bottom = where the email originated
> - Top = where it was finally delivered

### Example:
```
Received: by mx.gmail.com (final delivery)          ← 4. READ LAST
Received: from relay2.mailprovider.com               ← 3.
Received: from relay1.mailprovider.com               ← 2.
Received: from mail.origin-server.com (1.2.3.4)     ← 1. READ FIRST = TRUE ORIGIN
```

### What each Received line contains:
```
Received: from mail.suspicious.ru (185.234.12.44)
              ↓                        ↓
         Server hostname           Server IP address

          by mx.gmail.com
              ↓
         Server that received it

          with ESMTP id abc123
              ↓
         Protocol used

          for victim@gmail.com
              ↓
         Who it was delivered to

          ; Mon, 20 Apr 2026 10:23:01 +0000
                    ↓
              Timestamp of this hop
```

### Why timestamps matter:
```
Received: Mon 20 Apr 2026 10:23:01  ← server A (top)
Received: Mon 20 Apr 2026 10:22:45  ← server B
Received: Mon 20 Apr 2026 10:22:30  ← server C (bottom = origin)
```
Each hop should be **slightly later** than the previous one. If timestamps go backwards- email was tampered with 🚨

---

## Authentication Results- Your Quickest Red Flag Check

This single header summarizes SPF, DKIM and DMARC results:

```
Authentication-Results: mx.google.com;
  spf=pass    smtp.mailfrom=google.com       
  dkim=pass   header.d=google.com            
  dmarc=pass  action=none                    
```

vs a phishing email:

```
Authentication-Results: mx.google.com;
  spf=fail    smtp.mailfrom=fakebank.ru      
  dkim=fail   header.d=fakebank.ru           
  dmarc=none  action=none                    
```

### What each result means:

**SPF Results:**

| Result | Meaning |
|---|---|
| `pass` | Came from authorized server  |
| `fail` | Came from unauthorized server  |
| `softfail` | Suspicious but not hard blocked  |
| `neutral` | Domain has no SPF opinion |
| `none` | No SPF record exists |

**DKIM Results:**

| Result | Meaning |
|---|---|
| `pass` | Signature valid, not tampered  |
| `fail` | Signature invalid or missing  |
| `neutral` | Signature present but not verified |

**DMARC Results:**

| Result | Meaning |
|---|---|
| `pass` | SPF or DKIM aligned with From domain  |
| `fail` | Neither aligned  |
| `none` | No DMARC policy- email delivers anyway  |

---

## Spotting a Spoofed Email- All Red Flags

### Red Flag 1- From vs Reply-To Mismatch
```
From:     security@paypal.com        ← what victim sees
Reply-To: collect@attacker.ru        ← where replies actually go 
```
**Why attackers do this:** Victim thinks they're emailing PayPal but their reply goes straight to the attacker.

---

### Red Flag 2- Received Chain Doesn't Match From Domain
```
From: security@paypal.com
Received: from mail.attacker.ru (185.234.12.44)  
```
PayPal's emails should originate from PayPal's servers- not attacker.ru. The origin server domain **must match** the claimed sender domain.

---

### Red Flag 3- Authentication Failures
```
spf=fail   → didn't come from authorized server
dkim=fail  → signature invalid or missing
dmarc=fail → neither check passed
```
All three failing together = almost certainly a spoofed email.

---

### Red Flag 4- Return-Path Mismatch
```
From:        ceo@microsoft.com
Return-Path: bounce@randomdomain.com   
```
Return-Path is where bounced emails go. It should match the From domain. If it doesn't- the true sending domain is exposed here.

---

### Red Flag 5- Suspicious X-Originating-IP
```
X-Originating-IP: 185.234.12.44
```
Check this IP on:
- **ipinfo.io/185.234.12.44** → shows country, ISP, org
- If it's from an unexpected country or random ISP 

---

### Red Flag 6- Message-ID Domain Mismatch
```
From:       security@apple.com
Message-ID: <abc123@randomserver.ru>  
```
Message-ID is auto-generated by the sending server. Its domain should match the From domain.

---

### Red Flag 7- Urgency Language in Subject
Not a header field technically but always combined with spoofing:
```
Subject: URGENT: Your account will be suspended in 24 hours!
Subject: Immediate Action Required- Verify Now
Subject: Your payment has been declined
```
Urgency + failed authentication = textbook phishing.

---

## Full Phishing Header Example- Annotated

```
Delivered-To: victim@gmail.com

Received: by mx.gmail.com                 ← Final delivery server
Received: from mail.fakebank.ru           ←  TRUE ORIGIN- doesn't match From
         (185.234.12.44)                  ←  External IP- check on ipinfo.io

Authentication-Results:
  spf=fail                                ←  Not from authorized server
  dkim=fail                               ←  Signature invalid
  dmarc=none                              ←  No policy, email delivers anyway

From: "RealBank Security"                 ← Looks legit to victim
      <security@realbank.com>             ←  But SPF/DKIM say it's fake

Reply-To: steal@fakebank.ru              ←  Replies go to attacker

Return-Path: bounce@fakebank.ru          ←  Doesn't match From domain

Message-ID: <abc123@fakebank.ru>         ←  Doesn't match From domain

Date: Mon, 20 Apr 2026 10:23:01 +0000
Subject: URGENT: Your account has been suspended
```

**Summary of red flags in this email:**
1. Received chain originates from fakebank.ru not realbank.com
2. SPF fail- unauthorized server
3. DKIM fail- no valid signature
4. DMARC none- no protection policy
5. Reply-To goes to attacker domain
6. Return-Path exposes true sender domain
7. Message-ID domain doesn't match From

---

## How to Analyze Headers- Step by Step Process

```
Step 1: Get raw headers
        Gmail → Show Original
        Outlook → File → Properties
              ↓
Step 2: Check Authentication-Results first
        SPF / DKIM / DMARC pass or fail?
              ↓
Step 3: Read Received chain bottom to top
        Does origin server match From domain?
              ↓
Step 4: Check From vs Reply-To
        Do they match? If not → red flag
              ↓
Step 5: Check Return-Path and Message-ID
        Do their domains match From domain?
              ↓
Step 6: Look up suspicious IPs
        ipinfo.io → check country and ISP
              ↓
Step 7: Document all red flags found
        Write them up as IOCs for your report
```

---

## Tools for Header Analysis

| Tool | URL | What it does |
|---|---|---|
| MXToolbox Header Analyzer | mxtoolbox.com/EmailHeaders.aspx | Visual hop-by-hop breakdown |
| Google Admin Toolbox | toolbox.googleapps.com/apps/messageheader | Clean visual analysis |
| IPInfo | ipinfo.io | Look up suspicious IPs |
| MXToolbox SuperTool | mxtoolbox.com/SuperTool.aspx | Check SPF/DKIM/DMARC for any domain |

---

## Quick Reference Cheat Sheet

| Header Field | What to check |
|---|---|
| `Received:` | Read bottom to top- does origin match From domain? |
| `Authentication-Results:` | SPF/DKIM/DMARC all pass? |
| `From:` | Does it match Reply-To and Return-Path? |
| `Reply-To:` | Different from From? = red flag |
| `Return-Path:` | Different domain from From? = red flag |
| `Message-ID:` | Different domain from From? = red flag |
| `X-Originating-IP:` | Look up on ipinfo.io |
| `Date/Timestamps:` | Do hop timestamps go forward in time? |
