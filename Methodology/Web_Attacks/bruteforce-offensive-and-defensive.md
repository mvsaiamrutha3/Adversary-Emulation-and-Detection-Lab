# Web Attack 01 – Brute Force & Credential Stuffing

This section documents brute-force authentication attacks against the DVWA login functionality. Authentication bypass is required as an initial step before proceeding with any further attacks, making brute-force and credential stuffing a common starting point for attackers.

The attack was performed in both noisy and stealthy ways to simulate realistic attacker behavior and generate useful detection telemetry.

---

## Offensive Methodology

### Initial Authentication Attempt

The login request was first intercepted using Burp Suite to understand the request structure, parameters, and application behavior. This step allows attackers to craft accurate brute-force and credential stuffing attacks.

![Burp Intercepted Login Request](../../Images/burp_intercept_login.png)

---

### Brute Force Login Attack

A brute-force login attack was performed using Burp Intruder. Multiple username and password combinations were tested against the login endpoint in a short period of time. This approach generates several incorrect authentication attempts and is commonly used when account lockout policies are weak or absent.

This phase produces a high volume of failed login attempts and is easily detectable but useful to establish baseline behavior.

![Burp Intruder Brute Force Attack](../../Images/burp_intruder_bruteforce.png)

---

### Stealthy Credential Stuffing

After the baseline brute-force attack, a stealthier credential stuffing approach was used. Instead of rapidly testing credentials, common leaked credentials were tried with time gaps between requests.

Stealth techniques used:
- Introducing delays between authentication attempts
- Using common username and password combinations
- Mixing multiple incorrect logins with one or two successful logins in between
- Limiting the number of attempts per account

This approach helps attackers evade simple threshold-based detection mechanisms and blend in with legitimate user activity.

---

### OSINT-Assisted Brute Force

OSINT can be leveraged to improve brute-force success rates by identifying:
- Common usernames used within the organization
- Frequently reused passwords from breach data
- High-probability credential combinations

Using OSINT reduces the number of authentication attempts required and increases the likelihood of successful login.

---

## Logs Generated

During the attack, Apache access logs recorded multiple POST requests to the DVWA login endpoint. The logs show repeated authentication attempts from the same source, including both failed and successful logins.

![Apache Access Logs Showing Login Attempts](../../Images/apache_access_logs.png)

---

## Defensive Detection

From a defensive perspective, brute-force and credential stuffing attacks can be detected by identifying repeated authentication attempts with similar request patterns. Even when attackers change IP addresses, the request structure and behavior often remain consistent.

Indicators used for detection:
- Multiple POST requests to the login endpoint
- High number of authentication attempts within a time window
- Similar request structure across attempts
- Successful login after several failures
- Consistent behavior despite IP rotation

---

### Splunk Query – Detect Brute Force & Credential Stuffing

```bash
index=web sourcetype="apache:access" "/dvwa/login.php"
| rex field=_raw "^(?<src_ip>\d{1,3}(?:\.\d{1,3}){3})"
| rex field=_raw "\"(?<http_method>\w+)\s+(?<uri>\S+)\s+HTTP\/[0-9.]+\"\s+(?<status>\d{3})"
| where http_method="POST" AND uri="/dvwa/login.php"
| bin _time span=5m
| stats count as attempts, values(status) as statuses by _time, src_ip
| where attempts >= 10
| sort - attempts
```

![Bruteforce search query](../../Images/brute_force_search_query.png)
## Summary

This attack demonstrates how brute-force and credential stuffing techniques can be used as an initial access vector. While noisy attacks are easy to detect, stealthy credential stuffing with time gaps and mixed login outcomes can bypass basic defenses. Behavioral analysis of authentication requests remains the most effective method for detecting such attacks.
