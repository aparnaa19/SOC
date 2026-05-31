# SQL Injection detection

## Where Do SQL Injection Attacks Show Up?

There are three places you look:

1. Web Server Logs     — the most important
2. Database Logs       — deeper evidence
3. WAF/IDS Logs        — if they have one

We go through each one.

---

## 1. Web Server Logs — Your Primary Source

Every request that hits a web server gets logged. A typical log line looks like this:

```text
192.168.1.105 - - [30/May/2026:13:42:01] "GET /product?id=1' OR 1=1-- HTTP/1.1" 200 4523
```

Break it down:

```text
192.168.1.105          → attacker's IP address
30/May/2026:13:42:01   → exact time of request
GET /product?id=        → the endpoint being hit
1' OR 1=1--            → THE INJECTION PAYLOAD
200                    → server responded normally
4523                   → response size in bytes
```

---

## What SQL Injection Looks Like in Logs

Basic detection — look for these characters and keywords in URLs and request bodies:

```text
'                    Single quote — always suspicious in a parameter
--                   SQL comment
#                    SQL comment (MySQL)
OR 1=1               Classic bypass
AND 1=1              Condition testing
UNION SELECT         Data extraction attempt
SLEEP(               Time-based blind injection
WAITFOR DELAY        Time-based on MSSQL
information_schema   Attacker mapping the database
extractvalue(        Error-based injection
exp(~                Error-based injection
load_file(           File read / out-of-band
```

---

## Real Log Examples of Each Attack Type

### Union-based attack in logs:

```http
GET /search?q=' UNION SELECT username,password FROM users-- HTTP/1.1
GET /item?id=1 UNION SELECT null,table_name FROM information_schema.tables--
```

### Error-based attack in logs:

```http
GET /item?id=exp(~(select * from(select database())x))
GET /item?id=1 AND extractvalue(1,concat(0x7e,database()))--
```

### Boolean-based blind in logs:

```http
GET /item?id=1 AND 1=1--
GET /item?id=1 AND 1=2--
GET /item?id=1 AND SUBSTRING(database(),1,1)='a'--
GET /item?id=1 AND SUBSTRING(database(),1,1)='b'--
GET /item?id=1 AND SUBSTRING(database(),1,1)='c'--
```

See that pattern? Same endpoint, same parameter, tiny variation every time. That's the fingerprint of blind injection.

### Time-based blind in logs:

```http
GET /item?id=1 AND SLEEP(5)--
GET /item?id=1 AND IF(1=1,SLEEP(5),0)--
GET /item?id=1 AND IF(SUBSTRING(database(),1,1)='a',SLEEP(5),0)--
```

For time-based you also look at the response time column in logs. Normal requests take milliseconds. Time-based injection requests take 5000ms, 10000ms. That spike is unmistakable.

### sqlmap automated attack in logs:

```http
GET /item?id=1--
GET /item?id=1 ORDER BY 1--
GET /item?id=1 ORDER BY 2--
GET /item?id=1 ORDER BY 3--
GET /item?id=1 UNION ALL SELECT NULL--
GET /item?id=1 UNION ALL SELECT NULL,NULL--
GET /item?id=1 UNION ALL SELECT NULL,NULL,NULL--
```

This is the most obvious pattern. sqlmap sends requests in a very specific sequence. Hundreds of requests from one IP in under a minute. Any analyst who sees this knows immediately it's automated.

---

## 2. What We Did Today — In Log Form

Let me show you exactly what your uncle's Azure logs would show from our session today:

```text
13:30:01  POST /LoginAuth  {"EmailId":"'","Password":"test"}                          401
13:30:15  POST /LoginAuth  {"EmailId":"' OR 1=1 --","Password":"test"}                401
13:30:28  POST /LoginAuth  {"EmailId":"' OR '1'='1' --","Password":"test"}            401
13:30:41  POST /LoginAuth  {"EmailId":"admin@insops.com","Password":"test"}            401
13:30:55  POST /LoginAuth  {"EmailId":"admin@insops.com' --","Password":"test"}        401
13:31:10  POST /LoginAuth  {"EmailId":"admin@insops.com","Password":"' OR 1=1 --"}    401
13:31:22  POST /LoginAuth  {"EmailId":"' OR SLEEP(5) --","Password":"test"}           401  5043ms
```

A SOC analyst looking at these logs would immediately see:

- Single IP hitting the same endpoint repeatedly
- SQL keywords in the request body
- One request took 5 seconds — time-based attempt
- Clear enumeration pattern — trying admin@insops.com specifically
- All within 2 minutes — automated or focused manual testing

This is a textbook SQL injection investigation in logs.

---

## 3. The Three Patterns That Always Give It Away

No matter how clever the attacker is, these three patterns always appear:

### Pattern 1 — High Volume, Same Endpoint

Normal traffic:

```text
13:00:01  POST /LoginAuth  1 request
13:05:33  POST /LoginAuth  1 request
13:22:17  POST /LoginAuth  1 request
```

Attack traffic:

```text
13:42:01  POST /LoginAuth  1 request
13:42:02  POST /LoginAuth  1 request
13:42:02  POST /LoginAuth  1 request
13:42:03  POST /LoginAuth  1 request
... 200 more requests in 60 seconds ...
```

Real users don't hit the same endpoint 200 times a minute.

### Pattern 2 — Sequential Slight Variations

```http
?id=1 AND SUBSTRING(database(),1,1)='a'--
?id=1 AND SUBSTRING(database(),1,1)='b'--
?id=1 AND SUBSTRING(database(),1,1)='c'--
?id=1 AND SUBSTRING(database(),1,1)='d'--
```

Same parameter, one character changing. This is a machine doing blind injection character by character.

### Pattern 3 — Response Time Anomalies

```text
Normal requests:   45ms  23ms  67ms  31ms
Attack requests:   45ms  23ms  5043ms  23ms  10021ms
```

Those 5 and 10 second spikes are SLEEP() commands being obeyed. Time-based blind injection has a very distinct timing signature.

---

## 4. How to Investigate a SQLi Alert — Step by Step

When your SIEM fires a SQL injection alert, here's exactly what you do:

### Step 1 — Identify the source IP

Which IP is sending the requests?

Is it internal or external?

Has this IP appeared before in logs?

Is it a known scanner or attacker IP? (Check against threat intel)

### Step 2 — Scope the attack

When did it start?

Is it still ongoing?

Which endpoints were targeted?

How many requests were sent?

### Step 3 — Determine the attack type

Do you see UNION SELECT? → Union-based

Do you see SLEEP()? → Time-based blind

Do you see tiny variations repeating? → Boolean-based blind

Do you see extractvalue()? → Error-based

Do you see information_schema? → Reconnaissance phase

### Step 4 — Did it succeed?

This is the most critical question. Look for:

Response codes:

```text
200 with large response size → might have returned data
302 redirect → login bypass might have worked
500 errors → injection is breaking the query
401/403 → blocked or wrong credentials
```

Response size anomalies:

```text
Normal login fail: 234 bytes every time
Suspicious: one response is 15,000 bytes → data was returned
```

### Step 5 — Check database logs if available

Look for:

Unusual queries running — especially SELECT on sensitive tables

Queries with UNION

Queries coming from the web app user at unusual times

Large data reads — attacker dumping tables

### Step 6 — Contain and report

Block the attacker IP at firewall

Preserve logs — don't let them rotate

Document timeline of the attack

Escalate if data was accessed

Write incident report

---

## 5. Splunk Queries for SQLi Detection

Since you'll use Splunk as a SOC analyst, here are real detection queries:

### Detect SQL keywords in web logs:

```spl
index=web_logs
| search uri="*UNION*" OR uri="*SELECT*" OR uri="*SLEEP*"
OR uri="*OR+1=1*" OR uri="*information_schema*"
| table _time, src_ip, uri, status
```

### Detect high volume from single IP:

```spl
index=web_logs
| stats count by src_ip, uri
| where count > 50
| sort -count
```

### Detect time-based injection by response time:

```spl
index=web_logs
| where response_time > 4000
| table _time, src_ip, uri, response_time
| sort -response_time
```

### Detect blind injection pattern — sequential requests:

```spl
index=web_logs
| stats count, values(uri) as requests by src_ip
| where count > 30
| search requests="*SUBSTRING*" OR requests="*ASCII*"
```

---

## 6. User Enumeration Detection — Relevant to Today

Remember what we found on insops.ai — different error messages for valid vs invalid emails? Here's how a SOC analyst detects someone exploiting that:

Normal login failure pattern:

```text
Same error message every time
1-2 attempts per minute
```

Enumeration attack pattern:

```text
Many different emails tried
Rapid succession
Attacker slows down after finding "Invalid Password!" response
Then focuses all attempts on that confirmed email
```

In Splunk:

```spl
index=web_logs sourcetype=api_logs
| search message="Invalid Password"
| stats count by src_ip
| where count > 10
```

Ten "Invalid Password" responses to the same IP means they found a valid account and are now brute forcing it.

---

## 7. Quick Reference — SQLi IOCs for SOC

Print this and keep it:

### CHARACTERS TO ALERT ON IN PARAMETERS:

```text
'  "  --  #  ;  /*  */
```

### SQL KEYWORDS TO ALERT ON:

```text
UNION    SELECT    INSERT    UPDATE    DELETE
DROP     SLEEP     WAITFOR   BENCHMARK
information_schema    table_name    column_name
extractvalue    load_file    exp(~
```

### BEHAVIORAL INDICATORS:

- 50+ requests to same endpoint in 60 seconds
- Sequential parameter variations
- Response time spikes of 5+ seconds
- Response size anomalies
- 500 errors suddenly appearing
- Same IP, many different SQL payloads

### AUTOMATED TOOL SIGNATURES:

```text
sqlmap  → ORDER BY 1,2,3 sequence then UNION NULL,NULL,NULL
Havij   → specific payload formatting
Burp    → scanner sends payloads with specific patterns
```
