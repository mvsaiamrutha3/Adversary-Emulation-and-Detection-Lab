#Advanced Logging Configuration on Active Directory

This section documents how advanced logging was enabled on the Active Directory Domain Controller to capture detailed authentication and process execution activity. These logs are essential for analyzing attacker behavior and building detections in Splunk.

##Step 1: Review Authentication Events

After Active Directory was configured, I reviewed the Windows Security Event Log to confirm that authentication activity was being recorded, including successful logons (Event ID 4624) and failed logon attempts (Event ID 4625), which provide baseline identity telemetry.

![Auth_Event_Logs](../Images/Auth_Event_Logs.png)

##Step 2: Verify Kerberos TGT Request Logging

Since Active Directory uses Kerberos authentication, I verified that Kerberos Ticket Granting Ticket (TGT) request events (Event ID 4768) were present in the security logs. These events provide visibility into initial authentication requests and ticket issuance behavior.
![TGT_Request_Event](../Images/TGT_Request_Event.png)

##Step 3: Enable Process Creation Auditing

To gain visibility into execution activity on the Domain Controller, I enabled auditing for process creation using 
```bash
Advanced Audit Policy Configuration (Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Detailed Tracking → Audit Process Creation)
```
I also enabled command-line logging to capture detailed execution context.
![Enabling_Advanced_Logging](../Images/Enabling_Advanced_Logging.png)

##Step 4: Confirm Audit Categories

After enabling advanced auditing, I verified that the following audit categories were enabled:

- Account Logon

- Logon and Logoff

- Kerberos Authentication

- Process Creation

These categories ensure coverage of authentication and execution activity required for detection.

##Step 5: Apply Policy Changes

To immediately apply the updated audit settings, I forced a Group Policy update on the Domain Controller by running the following command from the console:
```bash
gpupdate /force
```
##Why This Matters

Advanced Active Directory logging provides high-quality telemetry for detecting authentication abuse, credential misuse, and suspicious process execution. These logs form a critical foundation for building effective Splunk detections.

##Next Step

With advanced logging enabled on the Domain Controller, the next step is to forward these security events to Splunk for centralized analysis.