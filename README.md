# Executive Account Takeover Investigation

## Description
I completed a hands-on simulated SOC investigation into an executive account takeover against Cloudora, a fictional organisation based in the UK.
The investigation started with an alert showing a sign-in from Lagos that did not match the user's normal activity. I used Microsoft Entra ID sign-in and audit logs to establish what happened, trace the initial access, identify persistence, scope the incident, and recommended containment.
<br />

<h2>Investigation Objectives</h2>

- Investigate the suspicious executive sign-in
- Establish the user's normal sign-in activity
- Confirm whether the account was compromised
- Identify the initial access method
- Investigate persistence mechanisms
- Check whether other accounts were affected
- Scope the password-spraying activity
- Recommend containment and recovery actions


<h2>Tools</h2>

- Azure data explorer
- Kusto Query Language (KQL)
- MITRE ATT&CK Framework
- NIST Incident Response Lifecycle

<h2>Environments Used </h2>

- Windows 10

<h2>Program walk-through:</h2>

<h3>Step 1: Triage the alert: </h3>
I  started by uploading the sign-in and audit log files into Azure data explorer. After ensuring that all the data was present and in the right format, I queried the data to obtain the executive account so I could understand their sign-in activity for the incident period. I checked out their: 

- IP address
- Location
- Application
- Sign-in result
- Time of activity

The location appeared unusual with Sign-In attempts coming from Lagos, but I did not treat the location alone as proof of compromise as I wanted to first understand what normal activity looks like for this account.

<br />
<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Suspicious-Executive-Sign-In.png"/>

*Figure 1 – Identifying Suspicious Sign-In* 
<br />

<h3> Step 2: Establishing a Baseline - Rule out the innocent explanation (User's Normal Activity):</h3>

I went ahead to review the executive's previous sign-ins to help establish a baseline. I compared suspicious activity against the user's normal activity. 
I also compared the investigation against another suspicious-looking sign-in involving another user(Omar) who took a trip to Dubai, and was ultimately cleared as a false positive

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Baseline1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Baseline-Omar.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*Figure 2 – Establishing Baseline for normal daiy activities*

<h3>Step 3:  How did they get in? Find the attack that came first </h3>

For a successful login from an attacker, the password must have been obtained somehow. Hunting backwards for credential attacks shows login failures from the Lagos IPs (102.89.44.17, .23, 102.89.45.101) spread across diffferent accounts, one to three attempts each, over three nights. This indicates a **password spray** (MITRE T1110.003) - designed to stay under lockout thresholds.

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Sign-In-Attempts.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Failed%20Attempts%20From%20Lagos%20IP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*Figure 3 – Password Spray Investigation*

<h3>Step 4: Investigating persistence - What did they do after getting in? </h3>

I queried the audit logs to try an understand the attacker's activity once they gained accessed into the account. Attackers will always seek to maintain their presence in an account even after they are caught. 
Two findings: at 03:18 the attacker registered their own authenticator app ('Pixel 6') - MITRE T1098.005, they now survive a password reset. At 03:31 UCT, they created an inbox rule (RSS Subscriptions) that hides any email from finance or containing 'invoice' - MITRE T1564.008, the classic setup for invoice fraud (BEC). This is why 'just reset the password' fails as a response.


<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Attackers%20activity.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*Figure 4 - Tracing attackers activity*

<h3> Step 5: Scoping the Incident </h3>

Once the executive account was confirmed as compromised, I expanded the investigation to know if the attacker had accessed any other accounts.
The attacker had targeted **26 accounts**. Further investigation showed a second successfully compromised account belonging to Priya.

Priya was successfully accessed from the same Lagos IP at approximately 03:47 UTC, around 30 minutes after the executive account.

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Scoping1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Scoping2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*Figure 5 – Scoping the extend of the attack*

<h3> Step 6: Investigating Priya</h3>

I went further to investigate the attack on Priya's account and noticed Password Spraying with multiple Login failures on thesame night before a successful login, all coming from the same IP address in Lagos. 

Using the Audit logs, I noticed no activities were carried out yet in Priya's account as there were no records of activities during the time of compromise. 

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Priya1.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

<img src="https://github.com/kamgaE-hub/SOCProject2/blob/main/evidence/Priya2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

*Figure 6 – Query on Priya's account*

<h3>Step7: Separating the Affected Accounts:</h3>

I separated the accounts into three groups rather than treating every targeted account as compromised. This separation helped avoid both overestimating the incident and missing accounts that still required action.

| Category                           | Result                 |
| ---------------------------------- | ---------------------- |
| **Confirmed compromised**          | Daniel and Priya       |
| **Targeted but not breached**      | 24 accounts            |
| **Investigated and cleared**       | Omar                   |


- The 24 accounts that were targeted by the password spray but did not show successful access will still require precautionary password resets and monitoring.

- Omar's Dubai activity was investigated and determined to be legitimate.


<h2>Overall Findings:</h2>

**Finding 1 - The CEO account was accessed by an attacker, not by Daniel travelling**
daniel.reeve@cloudora.io authenticated successfully at 03:12:05 UTC on 10 Aug from 102.89.44.17 (Lagos, Nigeria) on a Windows 10 / Chrome 125 device never seen on this account before. The success was immediately preceded by failed attempts at 03:09 and 03:10 and followed by mailbox access (Outlook Web, 03:14) and Azure Portal access (03:26). The account's 8-day baseline is exclusively United Kingdom (39 London sign-ins); Daniel signed in normally from London at 08:41 the same morning on his usual macOS/Safari device. Travel and VPN are excluded by the failure pattern, the 3AM timing, the unfamiliar device fingerprint, and the same-day London activity.

**Finding 2 - Initial access came from a three-night password spray, not phishing**
The three Lagos IPs generated 114 failed sign-ins (ResultType 50126) across 26 distinct accounts between 8 and 10 Aug, concentrated between 00:00 and 05:00 UTC, with only 1-3 attempts per account per night: 102.89.44.17 (48 failures, 23 accounts), 102.89.45.101 (38 failures, 20 accounts), 102.89.44.23 (28 failures, 18 accounts). This low-and-slow, many-accounts pattern is password spraying (T1110.003), designed to stay under account lockout thresholds.

**Finding 3 - The attacker established MFA persistence on the CEO account**
At 03:18:44, six minutes after initial access, the audit log records 'User registered security info' on daniel.reeve from 102.89.44.17: an authenticator app on device 'Pixel 6' (T1098.005). With their own MFA method registered, the attacker would survive a password reset - which is why credential reset alone was not treated as containment.

**Finding 4 - A BEC inbox rule was staged for invoice fraud**
At 03:31:09 the attacker created inbox rule 'RSS Subscriptions' on the CEO mailbox: any mail from finance@cloudora.io or containing 'invoice' is moved to the RSS Feeds folder and marked read (T1564.008). This hides finance conversations from the mailbox owner so the attacker can run invoice fraud from the CEO's identity undetected. No outbound fraudulent mail was observed in the available logs; the rule was staged but, on current evidence, not yet used.

**Finding 5 - A second account, priya.nair, was compromised the same night**
priya.nair@cloudora.io was successfully accessed from 102.89.45.101 at 03:47:18 after a failed attempt at 03:44:55, followed by SharePoint Online access at 03:52:40. Her baseline is 33 London sign-ins from 203.0.113.10 with no prior foreign activity. No persistence actions appear in the audit log for this account, but the same containment steps were applied.

**Finding 6 - omar.farah's Dubai activity is legitimate travel (false positive, cleared)**
omar.farah@cloudora.io shows 12 sign-ins from Dubai (185.93.245.66) across 8-10 Aug. All are daytime (09:00-19:00), all succeed first try with zero failures, and all come from his usual iOS 17 / Mobile Safari device seen throughout his London baseline. This is consistent travel, not compromise. Note: Omar was also among the 26 spray-targeted accounts (6 failed Lagos attempts against him, none successful), so he is cleared as a victim but still gets a precautionary reset.


<h2> MITRE ATT&CK Mapping: </h2>

| Technique                | ID        | Evidence from Investigation                            |
| ------------------------ | --------- | ------------------------------------------------------ |
| **Password Spraying**    | T1110.003 | Multiple accounts targeted over three nights           |
| **Valid Accounts**       | T1078     | Successful access using legitimate credentials         |
| **Account Manipulation** | T1098     | New authentication method added                        |
| **Device Registration**  | T1098.005 | Unrecognised Pixel 6 registered as an MFA device       |
| **Email Hiding Rules**   | T1564.008 | Finance and invoice emails moved out of the main inbox |

<h2> Recommendations:</h2>

1. Revoke all active sessions and refresh tokens for daniel.reeve@cloudora.io and priya.nair@cloudora.io
2. Force password resets for the 24 targeted-but-not-breached accounts - the spray may have come closer than the logs show.
3. Require MFA on all accounts and disable legacy authentication protocols that bypass it; two accounts fell to password-only guessing.
4. Alert on new MFA method registration and new inbox rules for executive accounts - both attacker persistence actions were loggable but unmonitored.
5. Deploy a password-spray detection rule (failures per IP across distinct accounts in a rolling window) - this attack was visible from night one, two days before the breach.
6. Review Conditional Access: block or step-up authentication from countries with no business presence.
7. Brief the finance team on BEC risk before the enterprise deal closes: verify any payment-detail change by phone on a known number.

