# 👻 Attacktive Directory — Professional Technical Walkthrough

> **Enterprise Active Directory Attack Chain Documentation**
> **TryHackMe Room:** Attacktive Directory
> **Author:** Anurag Ravikumar
> **Documentation Version:** Portfolio Edition v2.0
> **Environment:** Windows Active Directory Domain Controller

---

> **Educational Use Only**

This walkthrough documents an authorized penetration testing exercise performed inside the **TryHackMe Attacktive Directory** laboratory. All techniques were executed in a controlled learning environment. Sensitive credentials, NTLM hashes, Kerberos tickets, and challenge flags have been **redacted** for responsible disclosure and plagiarism-safe portfolio publication.

---

## Table of Contents

1. Executive Summary
2. Engagement Overview
3. Threat Scenario
4. Active Directory Lab Architecture
5. Attack Chain Overview
6. Objectives
7. Environment Enumeration
8. Reconnaissance
9. Service Enumeration
10. Domain Identification
11. Kerberos Username Enumeration
12. AS-REP Roasting
13. Password Cracking
14. SMB Enumeration
15. Credential Discovery
16. DCSync / NTDS Extraction
17. Pass-the-Hash Authentication
18. Post Exploitation
19. Flag Verification *(Redacted)*
20. MITRE ATT&CK Mapping
21. Detection Engineering Notes
22. Security Hardening Recommendations
23. Lessons Learned
24. References

---

# 1. Executive Summary

## Overview

The **Attacktive Directory** room simulates a Windows Active Directory enterprise network where an attacker performs reconnaissance against a Domain Controller, abuses Kerberos misconfigurations, recovers credentials, escalates privileges through directory replication rights, and authenticates as the Domain Administrator using a Pass-the-Hash attack.

Rather than exploiting a vulnerable web application or kernel vulnerability, this engagement focuses entirely on **Active Directory attack methodology**, demonstrating several real-world enterprise attack techniques commonly observed during Red Team operations and ransomware intrusions.

The walkthrough progresses through the complete identity attack lifecycle:

* Active Directory discovery.
* Kerberos user enumeration.
* AS-REP Roasting.
* Offline password cracking.
* SMB share enumeration.
* Credential harvesting.
* DCSync abuse.
* NTDS password extraction.
* Pass-the-Hash authentication.
* Domain Administrator compromise.

The exercise emphasizes understanding **how Active Directory trust relationships and authentication mechanisms can be abused when accounts are misconfigured or overprivileged**.

---

## Assessment Summary

| Category             | Details                              |
| -------------------- | ------------------------------------ |
| Platform             | TryHackMe                            |
| Room                 | Attacktive Directory                 |
| Environment          | Active Directory Domain Controller   |
| Operating System     | Windows Server 2019                  |
| Domain               | `spookysec.local`                    |
| Attack Type          | Internal Red Team / Active Directory |
| Initial Access       | Kerberos Enumeration                 |
| Privilege Escalation | DCSync Replication Abuse             |
| Final Access         | Domain Administrator                 |

---

## Skills Demonstrated

| Domain             | Skills                                |
| ------------------ | ------------------------------------- |
| Active Directory   | LDAP, Kerberos, SMB                   |
| Credential Access  | AS-REP Roasting, NTDS Extraction      |
| Windows Security   | Pass-the-Hash, WinRM                  |
| Password Attacks   | Offline Hashcat Cracking              |
| Post Exploitation  | PowerShell, Evil-WinRM                |
| Defensive Security | Event IDs, Sigma Ideas, MITRE Mapping |

---

# 2. Engagement Overview

## Scenario

An internal attacker gains network visibility into a Windows enterprise environment containing an Active Directory Domain Controller.

The objective is to enumerate the directory infrastructure, identify authentication weaknesses, recover credentials, escalate privileges through replication permissions, and obtain administrative access to the domain.

This scenario closely resembles enterprise identity attacks where attackers laterally move using Kerberos abuse instead of exploiting operating system vulnerabilities.

---

## Objectives

### Primary Objectives

* Discover the Active Directory domain.
* Enumerate valid domain users.
* Identify accounts vulnerable to AS-REP Roasting.
* Recover reusable credentials.
* Enumerate SMB shares.
* Extract privileged credentials.
* Obtain Domain Administrator access.

### Secondary Objectives

* Understand Kerberos authentication.
* Learn DCSync abuse.
* Understand NTLM authentication.
* Practice WinRM post-exploitation.

---

## Rules of Engagement

* Authorized laboratory environment.
* Educational objectives only.
* No persistence established.
* No destructive actions performed.
* Sensitive outputs intentionally redacted.

---

# 3. Threat Scenario

## Enterprise Identity Attack Simulation

The simulated environment contains:

* Windows Domain Controller.
* Kerberos Key Distribution Center.
* LDAP Directory Services.
* SMB File Shares.
* Administrative WinRM Endpoint.

The attacker begins without credentials and abuses authentication weaknesses to compromise privileged identities.

---

## Attack Narrative

```text
External Network
       │
       ▼
Reconnaissance
       │
       ▼
Active Directory Discovery
       │
       ▼
Kerberos Enumeration
       │
       ▼
Credential Access
       │
       ▼
Privilege Escalation
       │
       ▼
Domain Administrator
```

This mirrors common attack chains used in enterprise Active Directory compromises.

---

# 4. Active Directory Lab Architecture

## Environment Architecture

![Attacktive Directory Architecture](../docs/assets/images/architecture.png)

### Components

| Component         | Description                                 |
| ----------------- | ------------------------------------------- |
| Domain Controller | Central identity authority.                 |
| Kerberos KDC      | Issues authentication tickets.              |
| LDAP              | Directory information service.              |
| SMB               | Network file sharing.                       |
| SYSVOL            | Group Policy distribution.                  |
| NETLOGON          | Logon scripts and authentication resources. |
| WinRM             | Remote PowerShell administration.           |

---

## Authentication Workflow

```text
User
 │
 │ AS-REQ
 ▼
Kerberos KDC
 │
 │ AS-REP
 ▼
Ticket Granting Ticket
 │
 │ TGS-REQ
 ▼
Service Ticket
 │
 ▼
Requested Service
```

Understanding this workflow is critical because **AS-REP Roasting abuses the first authentication stage**.

---

# 5. Attack Chain Overview

## Complete Methodology

| Phase                 | Technique                 |
| --------------------- | ------------------------- |
| Recon                 | Nmap Enumeration          |
| Discovery             | CrackMapExec SMB          |
| Enumeration           | Kerbrute User Enumeration |
| Credential Access     | AS-REP Roasting           |
| Password Recovery     | Hashcat                   |
| SMB Discovery         | NetExec Shares            |
| Credential Harvesting | Backup Share              |
| Privilege Escalation  | DCSync                    |
| Credential Dumping    | SecretsDump               |
| Lateral Movement      | Pass-the-Hash             |
| Post Exploitation     | Evil-WinRM                |

---

## MITRE ATT&CK Lifecycle

| Tactic             | Technique         |
| ------------------ | ----------------- |
| Discovery          | Account Discovery |
| Credential Access  | Kerberos Tickets  |
| Credential Access  | Password Cracking |
| Lateral Movement   | SMB               |
| Credential Dumping | DCSync            |
| Defense Evasion    | Pass-the-Hash     |

---

# 6. Environment Enumeration

## Initial Target Information

The target machine was identified as a Windows Server operating inside an Active Directory domain.

Key observations included:

* Kerberos authentication service exposed.
* LDAP available.
* SMB signing enabled.
* WinRM exposed.
* IIS Web Server running.
* Domain Controller functionality.

These characteristics strongly indicate an enterprise authentication server rather than a workstation.

---

## Enumeration Goals

During this stage the objective was to identify:

* Domain name.
* Hostname.
* Authentication services.
* Management services.
* Potential attack surface.

---

# 7. Reconnaissance

## Objective

The first phase focused on identifying every exposed TCP service and determining whether the target functioned as an Active Directory Domain Controller.

---

## Tool Used

### Nmap

Nmap performs comprehensive TCP enumeration and service fingerprinting.

---

## Command Executed

```bash
nmap -Pn -A -p- -T4 <TARGET-IP>
```

### Why These Flags?

| Flag  | Purpose                                       |
| ----- | --------------------------------------------- |
| `-Pn` | Skip host discovery.                          |
| `-A`  | Version detection, OS detection, NSE scripts. |
| `-p-` | Scan all TCP ports.                           |
| `-T4` | Faster scan timing.                           |

---

## Screenshot — Initial Enumeration

![Nmap Enumeration](../docs/assets/images/01-nmap-recon.png)

*Figure 7.1 — Initial TCP scan identifying Kerberos, LDAP, SMB, WinRM, DNS and IIS services.*

---

## Analysis

The scan revealed several enterprise authentication services simultaneously.

### Interesting Services

| Port | Observation             |
| ---- | ----------------------- |
| 53   | DNS Service             |
| 88   | Kerberos Authentication |
| 389  | LDAP                    |
| 445  | SMB                     |
| 5985 | WinRM                   |
| 3268 | Global Catalog LDAP     |

The presence of LDAP and Kerberos immediately suggests that the target belongs to an Active Directory forest.

---

## Security Observation

SMB signing was enabled.

This mitigates:

* SMB Relay attacks.

However, authenticated SMB enumeration remains possible after credential compromise.

---

## Enumeration Findings

* Windows Server identified.
* Active Directory confirmed.
* DNS computer name leaked.
* Domain information leaked.
* Remote management exposed.

---

## Why This Matters

Reconnaissance establishes the authentication surface before interacting with Kerberos or LDAP. Enumerating these services allows attackers to transition from network discovery into identity-focused attacks.

---

# 8. Domain Discovery

## Goal

Identify the Active Directory domain name without authentication.

The SMB service leaks valuable domain metadata during negotiation.

---

## Tool Used

### CrackMapExec / NetExec

Purpose:

* SMB negotiation.
* Domain discovery.
* Hostname discovery.
* Signing status.

---

## Command

```bash
crackmapexec smb <TARGET-IP>
```

---

## Screenshot — SMB Domain Discovery

![Domain Discovery](../docs/assets/images/02-domain-discovery.png)

*Figure 8.1 — SMB negotiation revealing the Windows hostname, domain name and SMB security configuration.*

---

## Results

Information recovered:

| Attribute   | Value             |
| ----------- | ----------------- |
| Domain      | `spookysec.local` |
| Hostname    | `ATTACKTIVEDIREC` |
| SMB Signing | Enabled           |
| SMBv1       | Disabled          |

---

## Analysis

The domain name becomes essential for every subsequent Kerberos interaction.

Future tools require:

* Kerberos Realm.
* Domain Controller address.
* Username format.

Without this information Kerberos authentication requests cannot be generated correctly.

---

## Active Directory Intelligence Gathered

* Domain naming convention.
* Authentication realm.
* Internal hostname.
* SMB security posture.

---

## Security Notes

SMB metadata exposure is normal in enterprise environments but provides valuable reconnaissance information to attackers.

Organizations should:

* Restrict unnecessary SMB exposure.
* Limit anonymous enumeration.
* Monitor SMB reconnaissance attempts.

---


# 9. Kerberos Username Enumeration

> **Objective:** Identify valid Active Directory user accounts without requiring authentication.

Kerberos username enumeration is one of the earliest identity attacks performed against Windows domains. The Key Distribution Center (KDC) responds differently when a username exists versus when it does not, allowing attackers to validate usernames before attempting password attacks.

This phase establishes the list of identities used throughout the remainder of the attack chain.

---

## Attack Goal

* Discover valid domain accounts.
* Avoid password guessing.
* Prepare a target list for AS-REP roasting.
* Minimize authentication noise.

---

## Background — How Kerberos Leaks Usernames

Kerberos authentication begins with an **AS-REQ (Authentication Service Request)**.

If the supplied username:

* Exists → The KDC continues authentication.
* Does not exist → The KDC immediately returns an error indicating an unknown principal.

Attackers abuse this behavior to enumerate accounts without possessing credentials.

### Kerberos Enumeration Flow

```text
Attacker
   │
   │ Username List
   ▼
Kerberos KDC
   │
   ├── User Exists
   └── User Does Not Exist
```

---

## Tool Used — Kerbrute

**Kerbrute** is a Kerberos enumeration utility that validates usernames directly against the Domain Controller.

### Command Executed

```bash
kerbrute userenum \
-d spookysec.local \
--dc <TARGET-IP> users.txt
```

### Command Breakdown

| Option      | Description                 |
| ----------- | --------------------------- |
| `userenum`  | Username enumeration mode   |
| `-d`        | Kerberos realm / domain     |
| `--dc`      | Domain Controller IP        |
| `users.txt` | Candidate username wordlist |

---

## Screenshot — Kerberos Username Enumeration

![Kerberos Enumeration](../docs/assets/images/03-kerberos-user-enum.png)

*Figure 9.1 — Kerbrute validating usernames against the Kerberos Key Distribution Center.*

---

## Enumeration Results

Multiple domain accounts were successfully identified.

### Examples of Valid Accounts

| Username      |
| ------------- |
| Administrator |
| svc-admin     |
| backup        |
| james         |
| robin         |
| paradox       |
| ori           |
| darkstar      |

> **Note:** Only representative usernames are shown. No challenge-specific credentials are exposed.

---

## Analysis

The enumeration confirms that the domain contains several service accounts and privileged administrative identities.

Interesting observations:

* Presence of a **service account** (`svc-admin`).
* Presence of a **backup account**.
* Standard administrative account discovered.
* Multiple employee accounts.

Service accounts frequently possess elevated permissions and become attractive attack targets.

---

## Why Service Accounts Matter

Service accounts often:

* Run applications.
* Authenticate automatically.
* Use older password policies.
* Have Kerberos preauthentication disabled.
* Possess replication or backup permissions.

These characteristics make them valuable targets during credential attacks.

---

## Detection Engineering Notes

### Windows Event IDs

| Event ID | Description                     |
| -------- | ------------------------------- |
| 4768     | Kerberos TGT requested          |
| 4771     | Kerberos authentication failure |

### Blue Team Detection

Indicators include:

* Large numbers of AS-REQ requests.
* Sequential username attempts.
* Requests originating from one host.
* No successful interactive logons.

### Sigma Detection Idea

```yaml
title: Suspicious Kerberos Username Enumeration

logsource:
  product: windows
  service: security

detection:
  selection:
    EventID: 4768

condition: selection
```

---

## MITRE ATT&CK Mapping

| Technique                          | ID    |
| ---------------------------------- | ----- |
| Gather Victim Identity Information | T1589 |
| Account Discovery                  | T1087 |

---

# 10. AS-REP Roasting

> **Objective:** Obtain Kerberos authentication material for users that do not require Kerberos preauthentication.

This is the first credential access technique performed during the engagement.

Accounts configured without Kerberos preauthentication expose encrypted authentication material that attackers can crack offline.

---

## What is AS-REP Roasting?

AS-REP Roasting abuses a Kerberos configuration where **"Do not require Kerberos preauthentication"** is enabled.

Instead of proving identity first, the KDC returns an encrypted Authentication Service Response.

That encrypted response becomes an offline cracking target.

---

## Authentication Flow

```text
Attacker
   │
   │ AS-REQ
   ▼
Domain Controller
   │
   │ AS-REP
   ▼
Encrypted TGT
   │
   ▼
Offline Password Cracking
```

---

## Why This Vulnerability Exists

Kerberos normally requires preauthentication.

Without it:

* Password knowledge isn't verified.
* Authentication response is still generated.
* Hash becomes crackable offline.

---

## Tool Used — Impacket GetNPUsers

### Command Executed

```bash
GetNPUsers.py \
spookysec.local/ \
-usersfile valid_users.txt \
-request \
-dc-ip <TARGET-IP>
```

---

## Screenshot — AS-REP Enumeration

![ASREP Enumeration](../docs/assets/images/04-asrep-enumeration.png)

*Figure 10.1 — Impacket requesting Kerberos AS-REP responses for roastable accounts.*

---

## Output Analysis

The tool identified a Kerberos roastable account.

Sensitive output has been redacted.

Example format:

```text
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:<REDACTED_HASH>
```

---

## Why Offline Cracking is Powerful

Advantages for attackers:

* No account lockouts.
* Unlimited cracking attempts.
* No further communication with the Domain Controller.

---

## Detection Opportunities

Monitor for:

* Event ID 4768.
* Accounts missing preauthentication.
* Unexpected AS-REQ traffic.

---

## Hardening Recommendation

Always enable **Kerberos preauthentication** for domain accounts.

Audit using:

* Active Directory Users and Computers.
* PowerShell.
* BloodHound.
* Defender for Identity.

---

## MITRE ATT&CK Mapping

| Technique       | ID        |
| --------------- | --------- |
| AS-REP Roasting | T1558.004 |

---

# 11. Offline Password Cracking

> **Objective:** Recover plaintext credentials from the AS-REP response.

Once Kerberos authentication material has been obtained, password recovery becomes a completely offline activity.

---

## Tool Used — Hashcat

Hashcat supports Kerberos AS-REP hashes.

### Command

```bash
hashcat \
-m 18200 \
hash.txt \
passwords.txt
```

---

## Hash Mode

| Mode  | Hash Type                 |
| ----- | ------------------------- |
| 18200 | Kerberos AS-REP (etype23) |

---

## Screenshot — Password Recovery

![Hashcat Crack](../docs/assets/images/05-hashcat-crack.png)

*Figure 11.1 — Offline recovery of credentials using Hashcat.*

---

## Analysis

A weak password policy allowed successful credential recovery.

The recovered account was a privileged service account used later during SMB enumeration.

> Passwords are intentionally omitted from this public documentation.

---

## Password Security Discussion

Weak passwords remain one of the largest enterprise security risks.

Characteristics observed:

* Dictionary-compatible password.
* Service account reuse.
* Privileged account exposure.

---

## Blue Team Recommendations

* Minimum 14-character passwords.
* Fine-Grained Password Policies.
* Password rotation.
* MFA for privileged accounts.
* Monitor cracked password reuse.

---

## MITRE ATT&CK Mapping

| Technique         | ID        |
| ----------------- | --------- |
| Password Cracking | T1110.002 |

---

# 12. SMB Enumeration

> **Objective:** Enumerate network shares using recovered credentials.

With authenticated credentials, SMB becomes an excellent source of sensitive files.

---

## Why SMB Matters

Administrators frequently store:

* Backup files.
* Scripts.
* Credentials.
* Configuration exports.

---

## Tool Used — NetExec / CrackMapExec

### Command

```bash
netexec smb <TARGET-IP> \
-u svc-admin \
-p '<REDACTED_PASSWORD>' \
--shares
```

---

## Screenshot — SMB Share Enumeration

![SMB Shares](../docs/assets/images/06-smb-share-enum.png)

*Figure 12.1 — Enumerating accessible SMB shares using authenticated credentials.*

---

## Shares Identified

| Share    | Purpose                    |
| -------- | -------------------------- |
| ADMIN$   | Administrative             |
| IPC$     | Interprocess Communication |
| NETLOGON | Authentication Scripts     |
| SYSVOL   | Group Policy Objects       |
| backup   | Backup Storage             |

---

## Analysis

The **backup** share appeared interesting because privileged users often store backup artifacts inside dedicated shares.

---

## Accessing Backup Share

### Command

```bash
smbclient //<TARGET-IP>/backup
```

---

## Enumeration Checklist

* [x] Enumerate shares.
* [x] Identify readable shares.
* [x] Download files.
* [x] Inspect backup artifacts.

---

## Security Observation

Authenticated SMB enumeration is often overlooked during security assessments because organizations focus primarily on anonymous enumeration.

---

## Detection Notes

Windows Event IDs:

| Event ID | Description               |
| -------- | ------------------------- |
| 5140     | Network Share Access      |
| 5145     | Detailed SMB Share Access |

---

## MITRE ATT&CK Mapping

| Technique                  | ID        |
| -------------------------- | --------- |
| SMB / Windows Admin Shares | T1021.002 |

---

# 13. Credential Discovery

> **Objective:** Recover additional credentials stored inside accessible SMB shares.

This phase demonstrates a common enterprise weakness where operational backups contain reusable credentials.

---

## Backup File Discovery

A credential backup file was discovered inside the SMB share.

The file contained an encoded string rather than plaintext credentials.

---

## Screenshot — Backup Credentials

![Backup Credentials](../docs/assets/images/07-backup-credential-decode.png)

*Figure 13.1 — Backup credential file recovered from the SMB share.*

---

## Base64 Credential Recovery

### Decode Command

```bash
echo "<REDACTED_BASE64>" | base64 -d
```

---

## Result

Recovered:

* Backup account username.
* Backup account password.

Sensitive values remain hidden.

---

## Why Encoding Isn't Encryption

Base64 provides:

* Encoding.
* Transport formatting.

It provides **zero security**.

Anyone with access can decode the contents instantly.

---

## Enterprise Security Risk

Common examples include:

* Password exports.
* Configuration backups.
* Scheduled task credentials.
* API keys.
* Service account passwords.

---

## Blue Team Recommendations

* Never store plaintext credentials.
* Encrypt backups.
* Restrict SMB access.
* Rotate backup credentials regularly.

---

## MITRE ATT&CK Mapping

| Technique            | ID        |
| -------------------- | --------- |
| Credentials in Files | T1552.001 |

---

# 14. Backup Account Analysis

> **Objective:** Understand why the recovered backup account becomes the privilege escalation vector.

The backup account possessed permissions significantly beyond ordinary user privileges.

---

## Why Backup Accounts Are Sensitive

Backup accounts commonly receive:

* Backup Operators membership.
* Replication permissions.
* Read access across the directory.
* Administrative synchronization rights.

---

## Replication Permissions

Important permissions include:

* Replicating Directory Changes.
* Replicating Directory Changes All.

These permissions enable **DCSync attacks**.

---

## Active Directory Security Concept

A user with replication permissions can impersonate a Domain Controller requesting password replication.

This allows retrieval of NTDS password hashes without logging into the Domain Controller interactively.

---

## Privilege Escalation Preview

The next phase abuses these permissions to:

1. Synchronize directory secrets.
2. Extract NTLM hashes.
3. Recover Administrator authentication material.
4. Authenticate using Pass-the-Hash.

---

## Blue Team Detection

Monitor:

* Event ID 4662.
* Replication permission changes.
* Unusual DRSUAPI traffic.
* SecretsDump activity.

---

## MITRE ATT&CK Mapping

| Technique | ID        |
| --------- | --------- |
| DCSync    | T1003.006 |

---

# 15. Domain Privilege Escalation — DCSync & NTDS Extraction

> **Objective:** Abuse Active Directory replication permissions assigned to the backup account to retrieve NTDS password hashes from the Domain Controller.

At this stage, authenticated credentials belonging to the **backup** account had already been recovered from an SMB backup share. Instead of logging into the Domain Controller interactively, the assessment leveraged **Active Directory replication privileges** to synchronize credential secrets directly from the directory database.

This technique is commonly referred to as **DCSync**.

---

## Understanding DCSync

Active Directory Domain Controllers continuously replicate directory objects between one another using the **Directory Replication Service (DRSUAPI)**.

If an account possesses replication permissions, it can impersonate a Domain Controller and request password hashes for every user in the domain.

### Replication Workflow

```text
Backup Account
      │
      │ DRSUAPI Request
      ▼
Domain Controller
      │
      │ Directory Replication
      ▼
NTDS Password Hashes
      │
      ▼
Offline Credential Access
```

Unlike LSASS dumping, DCSync retrieves hashes **remotely** without executing code on the Domain Controller.

---

## Why the Backup Account Was Dangerous

The recovered account possessed permissions equivalent to:

* Replicating Directory Changes
* Replicating Directory Changes All

These permissions effectively authorize password synchronization operations.

> **Security Observation:** Backup and synchronization accounts frequently become high-value targets because organizations assign replication privileges for operational convenience.

---

## Tool Used — Impacket SecretsDump

The Impacket suite includes **SecretsDump**, a tool capable of performing DCSync operations through authenticated RPC communication.

### Command Executed

```bash
secretsdump.py spookysec.local/backup:<REDACTED_PASSWORD>@<TARGET-IP>
```

### Command Breakdown

| Parameter         | Description                       |
| ----------------- | --------------------------------- |
| `backup`          | Authenticated replication account |
| `spookysec.local` | Active Directory domain           |
| `<TARGET-IP>`     | Domain Controller                 |
| `SecretsDump`     | Performs DRSUAPI replication      |

---

## Screenshot — NTDS Extraction

![NTDS Extraction](../docs/assets/images/08-ntds-dump.png)

*Figure 15.1 — SecretsDump performing Active Directory replication and extracting NTDS credential material.*

---

## Results

SecretsDump successfully synchronized credential material from the Active Directory database.

Recovered information included:

* Domain account NTLM hashes.
* Kerberos AES keys.
* Machine account hashes.
* Administrator authentication material.

### Redacted Example Output

```text
Administrator:500:<LM_HASH>:<REDACTED_NTLM_HASH>
backup:1104:<LM_HASH>:<REDACTED_NTLM_HASH>
svc-admin:1105:<LM_HASH>:<REDACTED_NTLM_HASH>
```

> **All hashes have been intentionally redacted for responsible public disclosure.**

---

## Technical Analysis

Unlike password cracking, this stage **does not recover plaintext passwords**.

Instead, attackers obtain reusable authentication material that can authenticate directly using NTLM-based protocols.

### Why NTDS.dit Matters

`NTDS.dit` stores:

* Password hashes.
* Kerberos secrets.
* Domain credentials.
* Service account hashes.
* Computer account secrets.

Compromise of NTDS effectively compromises the entire domain.

---

## Detection Engineering Notes

### Windows Event IDs

| Event ID | Description                                |
| -------- | ------------------------------------------ |
| 4662     | Directory Replication Operation            |
| 4672     | Privileged Authentication                  |
| 4624     | Successful Logon                           |
| 4688     | SecretsDump Process Execution *(if local)* |

---

## Indicators of DCSync

Blue Teams should investigate:

* Replication requests from non-Domain Controllers.
* DRSUAPI network traffic.
* Replication by user accounts.
* Replication originating from workstations.

---

## MITRE ATT&CK Mapping

| Technique                      | ID        |
| ------------------------------ | --------- |
| OS Credential Dumping — DCSync | T1003.006 |

---

# 16. Administrator NTLM Hash Analysis

> **Objective:** Understand how extracted NTLM authentication material enables lateral movement without recovering plaintext passwords.

The Administrator NTLM hash obtained during DCSync becomes a reusable authentication artifact.

### NTLM Authentication Concept

```text
Password
   │
   ▼
NTLM Hash
   │
   ▼
Authentication Protocol
   │
   ▼
Authenticated Session
```

The password itself is unnecessary once the NTLM hash is available.

---

## Pass-the-Hash Explained

Pass-the-Hash authenticates using the NTLM hash directly.

### Requirements

* NTLM hash.
* Username.
* Network service supporting NTLM authentication.

---

## Security Impact

Pass-the-Hash allows attackers to:

* Authenticate remotely.
* Avoid password cracking.
* Move laterally.
* Access administrative services.

---

## Authentication Targets

Common protocols vulnerable to Pass-the-Hash:

| Service | Supported |
| ------- | --------- |
| SMB     | ✅         |
| WinRM   | ✅         |
| WMI     | ✅         |
| PsExec  | ✅         |
| RPC     | ✅         |

---

## Why WinRM Was Selected

The Domain Controller exposed **TCP 5985**, making WinRM an ideal administrative entry point.

---

# 17. Pass-the-Hash Authentication (Evil-WinRM)

> **Objective:** Authenticate as the Domain Administrator using the extracted NTLM hash.

---

## Tool Used — Evil-WinRM

Evil-WinRM provides remote PowerShell access over Windows Remote Management.

### Command Executed

```bash
evil-winrm \
-i <TARGET-IP> \
-u Administrator \
-H <REDACTED_NTLM_HASH>
```

### Command Breakdown

| Parameter | Purpose               |
| --------- | --------------------- |
| `-i`      | Target IP             |
| `-u`      | Administrator account |
| `-H`      | NTLM Hash             |

---

## Screenshot — Administrator Shell

![Administrator Shell](../docs/assets/images/09-administrator-shell.png)

*Figure 17.1 — Successful Pass-the-Hash authentication resulting in an interactive PowerShell session.*

---

## Authentication Result

A remote administrative PowerShell session was successfully established.

Capabilities included:

* Administrative command execution.
* User enumeration.
* File access.
* Domain administration.

---

## Why WinRM Matters

WinRM is Microsoft's remote management protocol for PowerShell remoting.

Enterprise administrators commonly use WinRM for:

* Server management.
* Automation.
* Configuration management.
* PowerShell Remoting.

---

## Detection Opportunities

Monitor:

| Event ID | Description                     |
| -------- | ------------------------------- |
| 4624     | WinRM Logon                     |
| 4648     | Explicit Credential Logon       |
| 4688     | PowerShell Process              |
| 4104     | PowerShell Script Block Logging |

---

## MITRE ATT&CK Mapping

| Technique       | ID        |
| --------------- | --------- |
| Pass-the-Hash   | T1550.002 |
| Remote Services | T1021     |

---

# 18. Post Exploitation

After obtaining administrative access, the engagement transitioned into post-exploitation enumeration.

The objective was **verification**, not persistence.

---

## Initial Validation Commands

```powershell
whoami
hostname
systeminfo
ipconfig
whoami /groups
whoami /priv
```

Purpose:

* Confirm Administrator privileges.
* Confirm Domain Controller.
* Identify privilege assignments.

---

## User Enumeration

```powershell
net users
net localgroup administrators
```

Purpose:

* Enumerate domain accounts.
* Identify privileged groups.

---

## Active Directory Enumeration

Useful PowerShell commands include:

```powershell
Get-ADUser
Get-ADComputer
Get-ADGroup
```

*(Demonstration only — not required to solve the room.)*

---

## Privilege Verification

Administrative privileges allowed access to protected directories including:

* Administrator Desktop.
* Backup User Desktop.
* Service Account Desktop.

---

## Operational Notes

No persistence techniques were established.

No scheduled tasks.

No registry persistence.

No service persistence.

The engagement concluded after flag verification.

---

# 19. Flag Verification *(Redacted)*

The objective of the room is to retrieve three challenge flags corresponding to different privilege levels.

---

## Screenshot — Flag Verification

![Flags Redacted](../docs/assets/images/10-flag-redacted.png)

*Figure 19.1 — Challenge completion with sensitive flag values intentionally hidden.*

---

## Flag Summary

| Flag                      | Status      |
| ------------------------- | ----------- |
| User Flag                 | ✅ Retrieved |
| Privilege Escalation Flag | ✅ Retrieved |
| Root / Administrator Flag | ✅ Retrieved |

---

## Public Portfolio Policy

Flags have been replaced with placeholders.

```text
THM{************************}
```

This keeps the repository suitable for public GitHub portfolios while preventing plagiarism.

---

# 20. MITRE ATT&CK Mapping

The following ATT&CK techniques were demonstrated during the assessment.

| Tactic            | Technique                          | ID        |
| ----------------- | ---------------------------------- | --------- |
| Reconnaissance    | Gather Victim Identity Information | T1589     |
| Discovery         | Account Discovery                  | T1087     |
| Discovery         | Network Service Discovery          | T1046     |
| Credential Access | AS-REP Roasting                    | T1558.004 |
| Credential Access | Password Cracking                  | T1110.002 |
| Credential Access | Credentials in Files               | T1552.001 |
| Credential Access | DCSync                             | T1003.006 |
| Lateral Movement  | SMB / Admin Shares                 | T1021.002 |
| Defense Evasion   | Pass-the-Hash                      | T1550.002 |
| Command Execution | PowerShell                         | T1059.001 |

---

## ATT&CK Kill Chain

```text
Recon
 │
 ▼
Discovery
 │
 ▼
Credential Access
 │
 ▼
Privilege Escalation
 │
 ▼
Lateral Movement
 │
 ▼
Domain Administrator
```

---

# 21. Detection Engineering Notes

## Windows Event IDs

| Event ID | Detection Purpose            |
| -------- | ---------------------------- |
| 4624     | Successful Authentication    |
| 4625     | Failed Authentication        |
| 4648     | Explicit Credentials         |
| 4662     | DCSync Detection             |
| 4688     | Suspicious Process Execution |
| 4768     | Kerberos AS Request          |
| 4769     | Kerberos Service Ticket      |
| 4771     | Kerberos Failure             |
| 5140     | SMB Share Access             |
| 5145     | SMB Object Access            |

---

## Detection Opportunities

### Kerberos Enumeration

Indicators:

* Many AS Requests.
* Sequential usernames.
* Unknown workstation.

---

### AS-REP Roasting

Indicators:

* Authentication requests without preauthentication.
* Roastable account activity.
* Event 4768 anomalies.

---

### SMB Enumeration

Indicators:

* Share listing.
* Backup share access.
* Administrative share enumeration.

---

### DCSync

Indicators:

* Event ID 4662.
* Replication from user accounts.
* DRSUAPI traffic.
* Replication outside Domain Controllers.

---

### Pass-the-Hash

Indicators:

* NTLM authentication.
* Administrator WinRM logon.
* Explicit credential logons.

---

## Sigma Rule Ideas

### AS-REP Roasting

```yaml
title: Kerberos ASREP Request Without Preauthentication
status: experimental
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4768
condition: selection
```

---

### DCSync Detection

```yaml
title: Active Directory DCSync Attempt
status: stable
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4662
condition: selection
```

---

# 22. Indicators of Compromise (IOCs)

| IOC                                  | Description          |
| ------------------------------------ | -------------------- |
| Large volume of Kerberos AS Requests | Username Enumeration |
| GetNPUsers Activity                  | AS-REP Roasting      |
| Hashcat Offline Attack               | Password Recovery    |
| SMB Backup Share Access              | Credential Discovery |
| DRSUAPI Replication                  | DCSync               |
| WinRM NTLM Authentication            | Pass-the-Hash        |

---

## Network Indicators

Potential network artifacts include:

* LDAP queries.
* Kerberos traffic.
* SMB file access.
* WinRM PowerShell sessions.

---

# 23. Security Hardening Recommendations

## Kerberos Security

* Enable Kerberos preauthentication.
* Disable legacy authentication.
* Audit service accounts.

---

## Password Security

* Minimum 14-character passwords.
* Password rotation.
* Fine-Grained Password Policies.
* Multi-Factor Authentication.

---

## Active Directory Hardening

* Remove unnecessary replication permissions.
* Audit Backup Operators.
* Review privileged groups.
* Monitor AdminSDHolder changes.

---

## SMB Hardening

* Restrict sensitive shares.
* Encrypt backups.
* Remove stored credentials.
* Enable auditing.

---

## NTLM Hardening

* Restrict NTLM authentication.
* Deploy Credential Guard.
* Enable LAPS.
* Disable NTLM where possible.

---

## WinRM Hardening

* Restrict WinRM to administrators.
* Enable PowerShell logging.
* Enable Just Enough Administration (JEA).

---

## Microsoft Defender for Identity

Recommended detections include:

* AS-REP Roasting.
* DCSync.
* Pass-the-Hash.
* Kerberoasting.
* Lateral Movement.

---

# 24. Lessons Learned

## Offensive Security Takeaways

This room demonstrates how several seemingly minor misconfigurations combine into a complete Domain Administrator compromise.

Important offensive concepts learned include:

* Kerberos reconnaissance.
* Username validation.
* AS-REP Roasting.
* Offline credential attacks.
* SMB credential discovery.
* DCSync privilege escalation.
* NTLM authentication abuse.
* WinRM remote administration.

---

## Defensive Security Takeaways

Security teams should prioritize:

* Kerberos preauthentication enforcement.
* Password policy improvements.
* Monitoring replication permissions.
* DCSync detection.
* WinRM monitoring.
* PowerShell logging.
* SMB auditing.

---

## SOC Analyst Perspective

Potential detection content includes:

* Sigma Rules.
* Windows Event Correlation.
* Kerberos monitoring.
* Defender for Identity alerts.
* Sentinel / Splunk detections.

---

# 25. Conclusion

The **Attacktive Directory** room provides an excellent introduction to enterprise Active Directory attack methodology and demonstrates how identity infrastructure becomes the primary attack surface during Windows domain compromises.

The assessment progressed from anonymous reconnaissance to complete Domain Administrator access through authentication abuse rather than software exploitation.

Major techniques demonstrated included:

* Active Directory discovery.
* Kerberos username enumeration.
* AS-REP Roasting.
* Offline password cracking.
* SMB enumeration.
* Credential harvesting.
* DCSync replication abuse.
* NTDS credential extraction.
* Pass-the-Hash authentication.
* Administrative PowerShell access.

This walkthrough documents each stage using a professional reporting methodology suitable for cybersecurity portfolios, interview demonstrations, GitHub Pages documentation, and Active Directory learning.

---

# References

## Official Documentation

* Microsoft Learn — Active Directory Domain Services.
* Microsoft Learn — Kerberos Authentication.
* Microsoft Learn — Windows Event IDs.

## Security References

* MITRE ATT&CK Framework.
* Impacket Documentation.
* Hashcat Documentation.
* Evil-WinRM Documentation.
* SigmaHQ Detection Rules.

## Training Platform

* TryHackMe — Attacktive Directory.

---

# Appendix

## Tools Used Throughout Assessment

| Tool                   | Purpose                    |
| ---------------------- | -------------------------- |
| Nmap                   | Network Enumeration        |
| CrackMapExec / NetExec | SMB Enumeration            |
| Kerbrute               | Username Enumeration       |
| GetNPUsers             | AS-REP Roasting            |
| Hashcat                | Offline Password Cracking  |
| SMBClient              | SMB File Access            |
| SecretsDump            | NTDS Extraction            |
| Evil-WinRM             | Remote Administrator Shell |

---

## Repository Screenshots

| Screenshot                        | Description                         |
| --------------------------------- | ----------------------------------- |
| `01-nmap-recon.png`               | Initial TCP Service Enumeration     |
| `02-domain-discovery.png`         | SMB Domain Discovery                |
| `03-kerberos-user-enum.png`       | Kerberos Username Enumeration       |
| `04-asrep-enumeration.png`        | AS-REP Roasting                     |
| `05-hashcat-crack.png`            | Offline Password Recovery           |
| `06-smb-share-enum.png`           | SMB Share Enumeration               |
| `07-backup-credential-decode.png` | Backup Credential Recovery          |
| `08-ntds-dump.png`                | NTDS / DCSync Extraction            |
| `09-administrator-shell.png`      | Evil-WinRM Administrator Session    |
| `10-flag-redacted.png`            | Challenge Completion (Flags Hidden) |

---

<div align="center">

## 🛡️ Enterprise Active Directory Security Walkthrough

**Professional Cybersecurity Portfolio Documentation**

*Red Team Methodology • Blue Team Detection • Active Directory Security*

**Author:** **Anurag Revankar**

</div>
