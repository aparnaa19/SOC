# SPF, DKIM & DMARC 

## The Big Picture First

Email was invented in the 1970s - security was never built in. By default anyone can claim to be anyone. SPF, DKIM and DMARC were added later to fix this.

**Without SPF/DKIM/DMARC:**
Attacker types `From: ceo@apple.com` → email delivers fine 

**With SPF/DKIM/DMARC:**
Attacker types `From: ceo@apple.com` → gets caught 

---

## SPF - Sender Policy Framework

### What it does:
SPF lets a domain publish a list of servers that are allowed to send email on its behalf.

### How it works step by step:

1. Apple publishes SPF record in their DNS:
   > "Only these IP addresses can send @apple.com emails"

2. You receive email claiming to be from `apple.com`

3. Your mail server checks:
   > "Did this email come from an IP in Apple's SPF list?"

4. `YES` → SPF Pass  
   `NO`  → SPF Fail 

### What SPF looks like in DNS:

```
v=spf1 ip4:17.32.0.0/12 include:icloud.com ~all
  ↓         ↓                    ↓              ↓
version   allowed IPs      also allow        ~all = soft fail
                           icloud.com's      (don't reject,
                           servers           just mark suspicious)
```

### The `all` mechanism - important:

| Symbol | Meaning |
|--------|---------|
| `-all` | Hard fail - reject anything not listed |
| `~all` | Soft fail - mark suspicious but deliver |
| `+all` | Allow everything (dangerous, bad practice) |

> **SPF Limitation - critical to know:**
> SPF only checks the server that sent it, NOT the From address you see in your email client. This is why SPF alone isn't enough.

---

## DKIM - DomainKeys Identified Mail

### What it does:
DKIM adds a digital signature to every outgoing email. The receiver can verify this signature to confirm:
- The email really came from that domain
- The email was not modified in transit

### How it works step by step:

1. Apple generates two keys:
   - `Private key` → kept secret on Apple's mail server
   - `Public key`  → published in DNS for everyone to see

2. Apple sends you an email:
   - Mail server signs the email with the Private key
   - Signature is added to email headers

3. Your mail server receives it:
   - Looks up Apple's Public key in DNS
   - Uses it to verify the signature

4. Signature valid?   → DKIM Pass  (not tampered)  
   Signature invalid? → DKIM Fail  (tampered or fake)

### What DKIM looks like in email headers:

```
DKIM-Signature: v=1; a=rsa-sha256; d=apple.com;
                s=selector; h=from:to:subject;
                bh=base64hash; b=digitalsignature

d=apple.com  → signed by apple.com
a=rsa-sha256 → encryption algorithm used
b=           → the actual signature
```

### DKIM vs SPF key difference:

| | SPF | DKIM |
|---|---|---|
| Checks | Where email came FROM | Whether email was TAMPERED |
| Can detect forwarding issues |  breaks on forward |  survives forwarding |

---

## DMARC - Domain-based Message Authentication Reporting & Conformance

### What it does:
DMARC is the policy layer that sits on top of SPF and DKIM. It answers:
> "If SPF or DKIM fails, what should receiving servers DO with that email?"

It also adds reporting - domain owners get reports about who is sending email using their domain.

### How it works step by step:

1. Email arrives claiming to be from `apple.com`

2. SPF check runs  → Pass or Fail  
   DKIM check runs → Pass or Fail

3. DMARC looks at both results:

   | Result | Outcome |
   |--------|---------|
   | SPF Pass + DKIM Pass | Deliver  |
   | SPF Fail + DKIM Pass | Check DMARC policy |
   | SPF Pass + DKIM Fail | Check DMARC policy |
   | SPF Fail + DKIM Fail | Almost certainly fake  |

4. DMARC policy decides the outcome

### DMARC Policies:

| Policy | What happens | Strength |
|--------|-------------|----------|
| `p=none` | Deliver email, just send report | Weak  |
| `p=quarantine` | Send to spam/junk folder | Medium  |
| `p=reject` | Block email completely | Strong  |

### What DMARC looks like in DNS:

```
v=DMARC1; p=reject; rua=mailto:dmarc@apple.com; pct=100

v=DMARC1  → version
p=reject  → policy: block failing emails
rua=      → send reports to this email
pct=100   → apply policy to 100% of emails
```

### DMARC Alignment - the extra check:

DMARC also checks that the domain in SPF/DKIM matches the From address you see:

```
Visible From: ceo@apple.com
SPF passes for: sneakyattacker.com
                      ↓
DMARC alignment FAILS 
Because apple.com ≠ sneakyattacker.com
```

> This is what catches **display name spoofing** - where attackers pass SPF but use a different domain.

---

## How All Three Work Together

```
EMAIL ARRIVES
      ↓
┌─────────────────────────────────────┐
│  SPF CHECK                          │
│  "Did it come from allowed server?" │
│  Pass  or Fail                      │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  DKIM CHECK                         │
│  "Is the signature valid?"          │
│  Pass  or Fail                      │
└─────────────────────────────────────┘
      ↓
┌─────────────────────────────────────┐
│  DMARC DECISION                     │
│  Both pass → Deliver                │
│  One fails → Quarantine or Reject   │
│  Both fail → Almost always Reject   │
└─────────────────────────────────────┘
```

---

## Real World Phishing Example

**Attacker goal:** fake an email from `hr@microsoft.com`

```
Step 1: Attacker sends from their own server
        SPF Check → FAIL (not Microsoft's server) 

Step 2: Attacker didn't have Microsoft's private key
        DKIM Check → FAIL (no valid signature) 

Step 3: DMARC policy for microsoft.com = p=reject
        DMARC Decision → BLOCKED 

Result: Email never reaches victim 
```

**But if Microsoft had `p=none`:**
```
Same attack → email DELIVERS to inbox 
Victim sees "From: hr@microsoft.com" and trusts it
```

---

## Quick Reference Cheat Sheet

| | SPF | DKIM | DMARC |
|---|---|---|---|
| **Purpose** | Verify sending server | Verify email integrity | Define failure policy |
| **Stored in** | DNS TXT record | DNS TXT record | DNS TXT record |
| **Protects against** | Server spoofing | Email tampering | Both + adds policy |
| **Can work alone?** | Partially | Partially | Needs SPF/DKIM |
| **Check with** | mxtoolbox.com/spf | mxtoolbox.com/dkim | mxtoolbox.com/dmarc |

---

## Check any domain:

```powershell
# SPF record
Resolve-DnsName -Name microsoft.com -Type TXT

# DMARC record
Resolve-DnsName -Name _dmarc.microsoft.com -Type TXT

# Or use MXToolbox (easier, visual):
# mxtoolbox.com → SuperTool → type any domain
```
