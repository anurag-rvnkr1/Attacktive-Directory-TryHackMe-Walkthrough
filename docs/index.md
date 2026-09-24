# 👻 Attacktive Directory

## Enterprise Active Directory Attack Chain — Premium Red Team Case Study

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-Attacktive_Directory-red?style=for-the-badge\&logo=tryhackme)
![Windows](https://img.shields.io/badge/Windows-Active_Directory-0078D6?style=for-the-badge\&logo=windows)
![Kerberos](https://img.shields.io/badge/Kerberos-Authentication-orange?style=for-the-badge)
![MITRE ATT\&CK](https://img.shields.io/badge/MITRE-T1003.006-critical?style=for-the-badge)
![Blue Team](https://img.shields.io/badge/Detection-Engineering-00C853?style=for-the-badge)
![Red Team](https://img.shields.io/badge/Privilege-Escalation-darkred?style=for-the-badge)

---

### Professional Cybersecurity Portfolio Documentation

**Windows Active Directory • Kerberos Abuse • AS-REP Roasting • SMB Enumeration • DCSync • Pass-the-Hash • Detection Engineering**

**Author:** **Anurag R**

Cybersecurity • SOC Analyst • Detection Engineering • Active Directory Security

</div>

---

> **Enterprise Identity Attack Simulation**

This project documents a complete compromise of a Windows **Active Directory** environment inside the authorized **TryHackMe Attacktive Directory** laboratory. The walkthrough follows a realistic Red Team methodology beginning with infrastructure reconnaissance and ending with authenticated **Domain Administrator** access through Kerberos abuse and Active Directory replication.

Unlike a traditional CTF write-up, this project is written as an **enterprise penetration testing report** suitable for recruiters, SOC analysts, detection engineers, and cybersecurity portfolios.

---

# Threat Assessment Dashboard

| Assessment Metric    | Value                                     |
| -------------------- | ----------------------------------------- |
| Platform             | TryHackMe                                 |
| Environment          | Windows Active Directory                  |
| Difficulty           | Easy *(AD Fundamentals)*                  |
| Attack Surface       | Kerberos • LDAP • SMB • WinRM             |
| Initial Access       | Kerberos Username Enumeration             |
| Credential Access    | AS-REP Roasting                           |
| Privilege Escalation | DCSync / NTDS Extraction                  |
| Final Access         | Domain Administrator                      |
| Report Style         | Enterprise Penetration Test Documentation |

---

# Navigation

## Documentation Sections

| Section               | Description                                    |
| --------------------- | ---------------------------------------------- |
| Executive Summary     | Assessment overview and engagement objectives. |
| Lab Architecture      | Active Directory infrastructure overview.      |
| Reconnaissance        | Nmap service enumeration and AD discovery.     |
| Kerberos Enumeration  | Username validation using Kerbrute.            |
| AS-REP Roasting       | Kerberos preauthentication abuse.              |
| Password Cracking     | Offline credential recovery.                   |
| SMB Enumeration       | Authenticated share discovery.                 |
| Credential Recovery   | Backup credential extraction.                  |
| DCSync Attack         | NTDS replication abuse.                        |
| Pass-the-Hash         | Administrator authentication using NTLM.       |
| Detection Engineering | MITRE mapping, Sigma ideas, Event IDs.         |
| Security Hardening    | Enterprise defensive recommendations.          |

---

# Executive Summary

## Assessment Overview

The **Attacktive Directory** room simulates a corporate Windows infrastructure where the attacker gains visibility into a Domain Controller and abuses several identity-based weaknesses to escalate privileges.

The engagement demonstrates how authentication mechanisms — rather than software vulnerabilities — become the primary attack vector inside enterprise Windows environments.

### Objectives

* Identify exposed Active Directory services.
* Discover the Windows domain.
* Enumerate valid domain users.
* Abuse Kerberos preauthentication.
* Recover reusable credentials.
* Enumerate SMB shares.
* Escalate privileges using replication permissions.
* Authenticate as Domain Administrator.

---

## Assessment Timeline

```text
Reconnaissance
      │
      ▼
Active Directory Discovery
      │
      ▼
Kerberos Username Enumeration
      │
      ▼
AS-REP Roasting
      │
      ▼
Offline Password Cracking
      │
      ▼
Authenticated SMB Enumeration
      │
      ▼
Backup Credential Recovery
      │
      ▼
DCSync / NTDS Extraction
      │
      ▼
Pass-the-Hash Authentication
      │
      ▼
Domain Administrator Access
```

---

# Engagement Scope

## Lab Information

| Property         | Value                           |
| ---------------- | ------------------------------- |
| Target           | Windows Domain Controller       |
| Domain           | `spookysec.local`               |
| Operating System | Windows Server                  |
| Services         | Kerberos, LDAP, SMB, WinRM, DNS |
| Authentication   | Active Directory Kerberos       |
| Objective        | Complete Domain Compromise      |

---

## Tools Used During Assessment

| Tool                   | Purpose                           |
| ---------------------- | --------------------------------- |
| Nmap                   | Network reconnaissance            |
| NetExec / CrackMapExec | SMB and domain discovery          |
| Kerbrute               | Kerberos username enumeration     |
| GetNPUsers (Impacket)  | AS-REP Roasting                   |
| Hashcat                | Offline password cracking         |
| SMBClient              | SMB share access                  |
| CyberChef / Base64     | Credential decoding               |
| SecretsDump (Impacket) | NTDS extraction                   |
| Evil-WinRM             | Pass-the-Hash administrator shell |

---

# Active Directory Lab Architecture

## Enterprise Authentication Infrastructure

![Active Directory Architecture](assets/images/architecture.png)

*Figure 1 — High-level Active Directory authentication architecture used throughout this assessment.*

---

### Components Identified

| Component         | Purpose                                     |
| ----------------- | ------------------------------------------- |
| Domain Controller | Identity provider for the domain.           |
| Kerberos KDC      | Issues authentication tickets.              |
| LDAP              | Directory information service.              |
| DNS               | Domain name resolution.                     |
| SMB               | Network file sharing.                       |
| SYSVOL            | Group Policy distribution.                  |
| NETLOGON          | Logon scripts and authentication resources. |
| WinRM             | Remote administrative PowerShell access.    |

---

## Kerberos Authentication Flow

```text
User
 │
 │ AS-REQ
 ▼
Key Distribution Center
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
Windows Service
```

Understanding this authentication flow is essential because the room abuses **AS-REP ticket generation** to obtain offline-crackable authentication material.

---

# Red Team Attack Path Visualization

## Enterprise Identity Attack Chain

```text
Internet
   │
   ▼
Port Enumeration
   │
   ▼
Domain Discovery
   │
   ▼
Kerberos Enumeration
   │
   ▼
AS-REP Roasting
   │
   ▼
Password Recovery
   │
   ▼
SMB Enumeration
   │
   ▼
Credential Harvesting
   │
   ▼
Backup Account Compromise
   │
   ▼
DCSync Attack
   │
   ▼
Administrator NTLM Hash
   │
   ▼
Pass-the-Hash
   │
   ▼
Domain Administrator Shell
```

---

## ATT&CK Kill Chain Summary

| Phase              | MITRE ATT&CK |
| ------------------ | ------------ |
| Reconnaissance     | T1595        |
| Discovery          | T1087        |
| Credential Access  | T1558.004    |
| Credential Access  | T1110.002    |
| Credential Access  | T1552.001    |
| Credential Dumping | T1003.006    |
| Lateral Movement   | T1021        |
| Defense Evasion    | T1550.002    |

---

# Skills Demonstrated

## Offensive Security

* Windows Domain Enumeration
* Kerberos Username Enumeration
* AS-REP Roasting
* Offline Password Cracking
* SMB Share Enumeration
* Credential Harvesting
* DCSync
* Pass-the-Hash
* Evil-WinRM Administration

---

## Defensive Security

* Windows Event IDs
* Kerberos Monitoring
* Replication Monitoring
* Sigma Detection Ideas
* IOC Identification
* NTLM Security
* SMB Auditing
* Active Directory Hardening

---

# Lab Objectives Completed

| Objective                        | Status                    |
| -------------------------------- | ------------------------- |
| Identify Active Directory Domain | 🟢 Completed              |
| Enumerate Users                  | 🟢 Completed              |
| AS-REP Roast Vulnerable Account  | 🟢 Completed              |
| Recover Credentials              | 🟢 Completed              |
| Enumerate SMB Shares             | 🟢 Completed              |
| Recover Backup Credentials       | 🟢 Completed              |
| Dump NTDS Credentials            | 🟢 Completed              |
| Authenticate as Administrator    | 🟢 Completed              |
| Retrieve Challenge Flags         | 🟢 Completed *(Redacted)* |

---

# Screenshot Gallery

## Assessment Evidence

| Phase                  | Screenshot                                      |
| ---------------------- | ----------------------------------------------- |
| Initial Reconnaissance | `assets/images/01-nmap-recon.png`               |
| Domain Discovery       | `assets/images/02-domain-discovery.png`         |
| Kerberos Enumeration   | `assets/images/03-kerberos-user-enum.png`       |
| AS-REP Attack          | `assets/images/04-asrep-enumeration.png`        |
| Password Recovery      | `assets/images/05-hashcat-crack.png`            |
| SMB Enumeration        | `assets/images/06-smb-share-enum.png`           |
| Backup Credentials     | `assets/images/07-backup-credential-decode.png` |
| NTDS Extraction        | `assets/images/08-ntds-dump.png`                |
| Administrator Shell    | `assets/images/09-administrator-shell.png`      |
| Challenge Completion   | `assets/images/10-flag-redacted.png`            |

> Every screenshot has been recreated and organized specifically for this portfolio documentation.

---

# Learning Outcomes

This assessment teaches the complete identity attack lifecycle inside Windows enterprise environments.

After completing this room you should understand:

* Active Directory architecture.
* Kerberos authentication internals.
* LDAP reconnaissance.
* Username enumeration.
* AS-REP Roasting.
* Offline password cracking.
* SMB credential discovery.
* DCSync attacks.
* NTDS extraction.
* Pass-the-Hash authentication.
* Detection engineering for Active Directory attacks.

---

# Documentation Roadmap

```text
Part I
──────
Executive Summary
Architecture
Threat Model
Reconnaissance

        ▼

Part II
────────
Kerberos Enumeration
AS-REP Roasting
Password Cracking
SMB Enumeration

        ▼

Part III
─────────
Credential Discovery
DCSync
Pass-the-Hash
Administrator Shell

        ▼

Part IV
────────
Detection Engineering
MITRE ATT&CK
Windows Event IDs
Hardening Recommendations
Lessons Learned
```

---

<div align="center">

## 🛡️ Enterprise Active Directory Red Team Case Study

**Professional GitHub Pages Documentation**

*Designed for cybersecurity recruiters, SOC analysts, penetration testers, and detection engineers.*

</div>

---

---

# Reconnaissance & Active Directory Discovery

> *Every successful Active Directory compromise begins with understanding the identity infrastructure.*

The first phase of the assessment focused on identifying exposed network services, discovering whether the target operated as a Windows Domain Controller, and collecting enough intelligence to begin attacking the Kerberos authentication infrastructure.

Unlike web application CTFs, this engagement starts by mapping enterprise authentication services rather than searching for HTTP vulnerabilities.

---

# Phase 01 — Network Reconnaissance

## Objective

The initial reconnaissance stage had four primary goals:

- Identify all exposed TCP services.
- Determine the operating system.
- Detect Active Directory services.
- Discover authentication protocols exposed to the network.

---

## Attack Methodology

```text
Internet Access
      │
      ▼
Host Discovery
      │
      ▼
TCP Port Enumeration
      │
      ▼
Service Fingerprinting
      │
      ▼
Operating System Identification
      │
      ▼
Domain Controller Confirmation
```

---

## Tool Used

### Nmap

Nmap was used to enumerate every TCP port and identify service versions running on the target Windows Server.

| Tool | Purpose |
|------|---------|
| Nmap | TCP Service Enumeration |
| NSE Scripts | Version Detection |
| OS Detection | Windows Fingerprinting |

---

## Command Executed

```bash
nmap -Pn -A -p- -T4 <TARGET-IP>
```

---

## Why These Flags?

| Flag | Explanation |
|------|-------------|
| `-Pn` | Skip ICMP discovery. |
| `-A` | Version detection + OS detection + NSE scripts. |
| `-p-` | Scan every TCP port. |
| `-T4` | Faster timing profile. |

---

# Screenshot — Initial Enumeration

![Nmap Enumeration](assets/images/01-nmap-recon.png)

> **Figure 2.1 — Full TCP scan identifying Windows Active Directory services.**

---

## Reconnaissance Analysis

The scan immediately revealed indicators that the target was functioning as a Domain Controller.

### Interesting Services Identified

| Port | Service | Security Importance |
|------|---------|--------------------|
| 53 | DNS | Active Directory name resolution. |
| 80 | IIS | Web Server. |
| 88 | Kerberos | Authentication service. |
| 135 | RPC | Remote procedure calls. |
| 139 | NetBIOS | SMB legacy transport. |
| 389 | LDAP | Directory Services. |
| 445 | SMB | Network Shares. |
| 464 | Kerberos Password Change | Kerberos management. |
| 593 | RPC over HTTP | Remote management. |
| 636 | LDAPS | Secure LDAP. |
| 3268 | Global Catalog | Forest-wide LDAP queries. |
| 5985 | WinRM | PowerShell Remoting. |

---

## Why These Ports Matter

Presence of **Kerberos (88)** and **LDAP (389)** together is one of the strongest indicators that a Windows server belongs to an Active Directory domain.

The Global Catalog LDAP service further confirms forest-level directory functionality.

---

# Initial Attack Surface Assessment

| Service | Risk |
|---------|------|
| Kerberos | Username Enumeration, AS-REP Roasting |
| LDAP | Directory Enumeration |
| SMB | Credential Discovery |
| WinRM | Pass-the-Hash Target |
| DNS | Domain Discovery |

---

## Security Observation

SMB signing was enabled.

### Security Impact

✅ Prevents SMB Relay attacks.

⚠️ Does **not** prevent authenticated SMB enumeration after credentials are compromised.

---

# Blue Team Perspective

Reconnaissance activity often generates:

| Event Source | Detection |
|-------------|-----------|
| Firewall | Port Scan |
| IDS/IPS | SYN Sweep |
| Defender for Endpoint | Network Discovery |
| SIEM | Multiple Port Connection Attempts |

---

## MITRE ATT&CK

| Tactic | Technique |
|--------|-----------|
| Reconnaissance | Active Scanning |
| Discovery | Network Service Discovery |

---

<div align="center">

## Stage Complete — Windows Active Directory Identified

</div>

---

# Phase 02 — Domain Discovery

After confirming that the target exposed SMB services, authenticated negotiation information was used to identify the Active Directory domain.

---

## Objective

Discover:

- Domain Name.
- Hostname.
- SMB Signing Status.
- SMB Version.
- Authentication Realm.

---

## Tool Used

### CrackMapExec / NetExec

CrackMapExec performs SMB negotiation without requiring credentials.

---

## Command Executed

```bash
crackmapexec smb <TARGET-IP>
```

---

# Screenshot — SMB Domain Discovery

![SMB Discovery](assets/images/02-domain-discovery.png)

> **Figure 2.2 — SMB negotiation exposing the Active Directory domain name and host information.**

---

## Domain Intelligence Collected

| Attribute | Value |
|-----------|-------|
| Domain | `spookysec.local` |
| Hostname | `ATTACKTIVEDIREC` |
| SMB Signing | Enabled |
| SMB Version | SMBv2 |

---

## Why This Information Is Valuable

The domain name becomes the Kerberos Realm used during authentication attacks.

Future tools require:

- Kerberos Realm.
- Domain Controller address.
- Username format.

---

## Authentication Context

```text
Domain
   │
   ▼
spookysec.local
   │
   ▼
Kerberos Realm
   │
   ▼
KDC Authentication
```

Without the domain name, Kerberos requests cannot be generated correctly.

---

# Active Directory Intelligence Summary

- Windows Server identified.
- Domain Controller confirmed.
- Kerberos Realm discovered.
- Internal hostname exposed.
- SMB security posture identified.

---

## Blue Team Insight

SMB negotiation metadata is legitimate functionality but provides attackers with valuable reconnaissance information.

Organizations should:

- Restrict anonymous SMB negotiation where possible.
- Monitor SMB enumeration.
- Audit SMB connection sources.

---

<div align="center">

## Stage Complete — Kerberos Realm Identified

</div>

---

# Phase 03 — Kerberos Username Enumeration

The assessment now transitioned from infrastructure discovery into **identity reconnaissance**.

Instead of guessing passwords, the objective became discovering **valid Active Directory users**.

---

## Objective

- Validate usernames.
- Avoid password guessing.
- Identify service accounts.
- Build target list for AS-REP Roasting.

---

# Kerberos Authentication Refresher

```text
User
 │
 │ AS-REQ
 ▼
Kerberos KDC
 │
 ├── Valid User
 └── Invalid User
```

The KDC responds differently depending on whether the username exists.

---

## Why This Works

Kerberos leaks identity information before authentication completes.

Responses reveal:

- User exists.
- User disabled.
- User invalid.

---

## Tool Used — Kerbrute

Kerbrute validates usernames directly against the Key Distribution Center.

---

## Command Executed

```bash
kerbrute userenum \
-d spookysec.local \
--dc <TARGET-IP> users.txt
```

---

## Screenshot — Username Enumeration

![Kerberos Enumeration](assets/images/03-kerberos-user-enum.png)

> **Figure 3.1 — Kerberos username validation against the Active Directory Domain Controller.**

---

## Enumeration Results

Several usernames were validated successfully.

### Representative Accounts

| Username | Observation |
|----------|-------------|
| Administrator | Built-in Administrator |
| svc-admin | Service Account |
| backup | Backup Account |
| james | Standard User |
| robin | Standard User |
| paradox | Standard User |
| ori | Standard User |
| darkstar | Standard User |

---

## Service Account Discovery

The service account (`svc-admin`) immediately became a high-value target because service accounts commonly possess:

- Elevated permissions.
- Weak passwords.
- Legacy Kerberos settings.
- Automation privileges.

---

## Attack Surface Expansion

```text
Users.txt
    │
    ▼
Kerberos Enumeration
    │
    ▼
Valid Domain Users
    │
    ▼
AS-REP Roast Candidates
```

---

## Security Impact

User enumeration dramatically reduces the password attack surface.

Instead of guessing usernames and passwords simultaneously, attackers only attack confirmed identities.

---

# Detection Engineering Notes

### Windows Event IDs

| Event ID | Meaning |
|----------|----------|
| 4768 | Kerberos TGT Requested |
| 4771 | Kerberos Authentication Failure |

---

### Indicators

- Hundreds of AS Requests.
- Sequential usernames.
- Unknown workstation requesting Kerberos tickets.

---

### Sigma Detection Concept

```yaml
title: Kerberos Username Enumeration

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

| Tactic | Technique |
|--------|-----------|
| Discovery | Account Discovery |
| Reconnaissance | Gather Victim Identity Information |

---

# Identity Reconnaissance Summary

| Task | Status |
|------|--------|
| Kerberos Realm Identified | ✅ |
| Valid Users Enumerated | ✅ |
| Service Accounts Found | ✅ |
| Backup Account Found | ✅ |
| Target List Created | ✅ |

---

<div align="center">

# Reconnaissance Phase Complete

**Infrastructure Mapped → Identity Infrastructure Discovered → Kerberos Attack Surface Identified**

</div>

---

---

# Credential Access — Kerberos Abuse & SMB Enumeration

> *Identity attacks begin where authentication trust is weakest.*

With valid Active Directory usernames identified during reconnaissance, the engagement transitioned into **Credential Access**. This phase abuses a Kerberos configuration weakness to obtain encrypted authentication material, cracks it offline, authenticates to SMB services, and recovers additional privileged credentials from enterprise backup storage.

This represents one of the most common attack paths seen in Windows Active Directory environments.

---

# Phase 04 — AS-REP Roasting

## Objective

Identify domain accounts configured without **Kerberos Preauthentication** and recover AS-REP authentication material for offline password cracking.

---

## What is AS-REP Roasting?

AS-REP Roasting is a Kerberos attack that targets user accounts where **"Do not require Kerberos preauthentication"** is enabled.

Instead of validating knowledge of the password first, the Key Distribution Center immediately returns an encrypted authentication response.

The attacker can then crack this response offline.

---

## Kerberos Authentication Comparison

### Normal Authentication

```text
Client
   │
   │ AS-REQ + Preauthentication
   ▼
KDC
   │
   ▼
AS-REP (Encrypted)
```

### Vulnerable Authentication

```text
Client
   │
   │ AS-REQ
   ▼
KDC
   │
   ▼
AS-REP Returned Immediately
   │
   ▼
Offline Password Cracking
```

---

## Why This Misconfiguration Exists

Organizations sometimes disable Kerberos preauthentication for:

- Legacy applications.
- Compatibility reasons.
- Misconfigured service accounts.

This exposes authentication material without requiring credentials.

---

## Tool Used

### Impacket — GetNPUsers

Impacket requests AS-REP responses for users discovered during Kerberos enumeration.

---

## Command Executed

```bash
GetNPUsers.py \
spookysec.local/ \
-usersfile valid_users.txt \
-request \
-dc-ip <TARGET-IP>
```

---

# Screenshot — AS-REP Enumeration

![ASREP Roasting](assets/images/04-asrep-enumeration.png)

> **Figure 4.1 — Requesting Kerberos AS-REP responses for roastable Active Directory accounts.**

---

## Attack Analysis

A vulnerable service account returned an encrypted Kerberos authentication response.

The response contains:

- Kerberos encryption type.
- Username.
- Authentication material.

### Redacted Example

```text
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:<REDACTED_HASH>
```

Sensitive authentication material has been intentionally removed.

---

## Security Impact

AS-REP roasting enables attackers to:

- Perform unlimited offline password cracking.
- Avoid account lockouts.
- Generate minimal authentication noise.

---

## Blue Team Perspective

### Detection Opportunities

| Event ID | Detection Purpose |
|----------|-------------------|
| 4768 | Kerberos Ticket Request |
| 4771 | Kerberos Authentication Failure |

### Indicators

- Large numbers of AS Requests.
- Requests without successful authentication.
- Requests targeting multiple usernames.

---

## MITRE ATT&CK

| Technique | ID |
|-----------|----|
| AS-REP Roasting | T1558.004 |

---

<div align="center">

## Stage Complete — Kerberos Authentication Material Obtained

</div>

---

# Phase 05 — Offline Password Cracking

## Objective

Recover reusable credentials from the AS-REP authentication material without interacting with the Domain Controller.

---

## Why Offline Cracking Matters

Offline attacks eliminate:

- Account lockouts.
- Password attempt limits.
- Authentication logging for every password guess.

Only the initial AS-REP request is visible.

---

## Tool Used

### Hashcat

Hashcat supports Kerberos AS-REP hashes using **Mode 18200**.

---

## Command Executed

```bash
hashcat \
-m 18200 \
hash.txt \
passwords.txt
```

---

## Hashcat Workflow

```text
AS-REP Hash
     │
     ▼
Hashcat
     │
     ▼
Dictionary Attack
     │
     ▼
Recovered Password
```

---

# Screenshot — Password Recovery

![Hashcat Password Recovery](assets/images/05-hashcat-crack.png)

> **Figure 5.1 — Offline recovery of the vulnerable service account password using Hashcat.**

---

## Analysis

The password was successfully recovered using a dictionary-based attack.

### Why This Worked

Weak password policy.

Characteristics included:

- Predictable structure.
- Dictionary-compatible word.
- Service account credential reuse.

---

## Password Security Discussion

Weak service account passwords remain a significant enterprise security issue.

### Risks

- Lateral movement.
- Privilege escalation.
- Kerberoasting.
- Password spraying.

---

## Blue Team Recommendations

- Strong password policies.
- Password rotation.
- Service account management.
- Managed Service Accounts (gMSA).

---

## MITRE ATT&CK

| Technique | ID |
|-----------|----|
| Password Cracking | T1110.002 |

---

<div align="center">

## Stage Complete — Service Account Credentials Recovered

</div>

---

# Phase 06 — Authenticated SMB Enumeration

## Objective

Use the recovered service account credentials to enumerate accessible SMB shares.

---

## Enterprise Context

SMB is frequently used for:

- File servers.
- Backup storage.
- Administrative shares.
- Group Policy distribution.

Once valid credentials are obtained, SMB becomes an excellent source of sensitive information.

---

## Tool Used

### NetExec (CrackMapExec)

---

## Command Executed

```bash
netexec smb <TARGET-IP> \
-u svc-admin \
-p '<REDACTED_PASSWORD>' \
--shares
```

---

# Screenshot — SMB Share Enumeration

![SMB Enumeration](assets/images/06-smb-share-enum.png)

> **Figure 6.1 — Enumerating SMB shares using authenticated service account credentials.**

---

## Shares Identified

| Share | Purpose |
|-------|---------|
| ADMIN$ | Administrative Share |
| IPC$ | Named Pipes |
| NETLOGON | Login Scripts |
| SYSVOL | Group Policy Objects |
| backup | Backup Storage |

---

## Share Risk Assessment

| Share | Risk |
|-------|------|
| SYSVOL | Login scripts & GPOs |
| NETLOGON | Authentication resources |
| backup | Sensitive backup artifacts |

The **backup** share became the primary investigation target.

---

## SMB Enumeration Checklist

- [x] Enumerate accessible shares.
- [x] Verify permissions.
- [x] Browse contents.
- [x] Download backup files.

---

## Accessing Backup Share

### SMBClient Command

```bash
smbclient //<TARGET-IP>/backup
```

---

## Why Backup Shares Matter

Organizations frequently store:

- Credential exports.
- Password backups.
- Configuration files.
- Automation scripts.
- Service account secrets.

---

## Blue Team Detection

### Windows Event IDs

| Event ID | Description |
|----------|-------------|
| 5140 | Network Share Access |
| 5145 | SMB Object Access |

Monitor:

- Administrative shares.
- Backup shares.
- Unusual workstation access.

---

## MITRE ATT&CK

| Technique | ID |
|-----------|----|
| SMB / Windows Admin Shares | T1021.002 |

---

<div align="center">

## Stage Complete — Sensitive Backup Share Located

</div>

---

# Phase 07 — Backup Credential Discovery

## Objective

Recover additional credentials stored inside enterprise backup files.

---

## Backup File Analysis

The backup share contained a credential backup file encoded using Base64.

Although encoded, it was not encrypted.

---

# Screenshot — Backup Credential File

![Backup Credentials](assets/images/07-backup-credential-decode.png)

> **Figure 7.1 — Encoded backup credential recovered from an SMB share.**

---

## Base64 Recovery Workflow

### Decode Using Base64

```bash
echo "<REDACTED_BASE64>" | base64 -d
```

---

### Alternative Method

CyberChef may also decode Base64 safely during investigations.

---

## Analysis

Recovered information included:

- Backup username.
- Backup password.

Sensitive values remain redacted.

---

## Why Base64 is Dangerous

Base64 provides:

- Encoding.
- Formatting.

It **does not provide confidentiality**.

Anyone with read access can recover the contents instantly.

---

## Enterprise Security Lessons

Never store:

- Passwords.
- Tokens.
- Secrets.
- API Keys.
- Service Account Credentials.

inside readable backup locations.

---

## Blue Team Recommendations

- Encrypt backups.
- Remove plaintext credentials.
- Restrict SMB permissions.
- Monitor backup share access.

---

## MITRE ATT&CK

| Technique | ID |
|-----------|----|
| Credentials in Files | T1552.001 |

---

<div align="center">

## Stage Complete — Backup Account Credentials Recovered

</div>

---

# Phase 08 — Backup Account Security Analysis

The recovered backup account possessed elevated directory permissions that exceeded ordinary user privileges.

This became the privilege escalation vector.

---

## Why Backup Accounts Are High Value Targets

Backup accounts commonly receive permissions including:

- Backup Operators.
- Replication Rights.
- Read access to directory secrets.
- Synchronization privileges.

---

## Active Directory Replication Permissions

Important permissions include:

| Permission | Security Impact |
|-----------|-----------------|
| Replicating Directory Changes | Directory Synchronization |
| Replicating Directory Changes All | NTDS Credential Replication |

These permissions enable **DCSync**.

---

## Attack Preview

```text
Backup Credentials
       │
       ▼
Directory Replication Permissions
       │
       ▼
SecretsDump
       │
       ▼
Administrator NTLM Hash
```

---

## Detection Engineering Preview

Blue Teams should monitor:

- Event ID 4662.
- Replication requests.
- DRSUAPI traffic.
- Backup account privilege usage.

---

<div align="center">

# Credential Access Phase Complete

**Kerberos Compromised → Credentials Recovered → Backup Account Identified**

</div>

---
---

# Domain Privilege Escalation — Active Directory Compromise

> *The engagement transitions from credential access into complete domain compromise by abusing Active Directory replication privileges.*

At this stage, authenticated access to the **backup** account had already been established through SMB credential recovery. The backup account possessed directory replication permissions, allowing the attacker to impersonate a Domain Controller and synchronize password hashes directly from the Active Directory database.

This technique is widely known as **DCSync** and is one of the highest-impact Active Directory attacks.

---

# Phase 09 — DCSync Privilege Escalation

## Objective

Abuse Active Directory replication permissions assigned to the backup account to retrieve NTDS password hashes from the Domain Controller.

---

## Understanding DCSync

Active Directory Domain Controllers continuously replicate objects using the **Directory Replication Service (DRSUAPI)**.

If an account has replication rights, it can request password hashes for every user in the domain.

### Replication Attack Flow

```text
Backup Account
      │
      │ Authenticated DRSUAPI Request
      ▼
Domain Controller
      │
      │ Directory Replication
      ▼
NTDS Credential Database
      │
      ▼
Password Hashes Returned
```

Unlike LSASS dumping, **DCSync does not require code execution on the Domain Controller**.

---

## Why Replication Permissions Are Dangerous

The backup account possessed permissions equivalent to:

| Permission                        | Security Impact               |
| --------------------------------- | ----------------------------- |
| Replicating Directory Changes     | Read directory changes        |
| Replicating Directory Changes All | Replicate password secrets    |
| Directory Synchronization         | Domain-wide credential access |

These permissions effectively allow an attacker to behave like another Domain Controller.

---

## Attack Surface Analysis

### High-Risk Active Directory Permissions

| Group / Permission | Risk              |
| ------------------ | ----------------- |
| Domain Admins      | Full compromise   |
| Enterprise Admins  | Forest compromise |
| Backup Operators   | Sensitive backups |
| Replication Rights | NTDS extraction   |

---

## Tool Used — Impacket SecretsDump

Impacket's `SecretsDump.py` performs DCSync by requesting password replication through DRSUAPI.

### Command Executed

```bash
secretsdump.py spookysec.local/backup:<REDACTED_PASSWORD>@<TARGET-IP>
```

---

# Screenshot — NTDS Extraction

![NTDS Extraction](assets/images/08-ntds-dump.png)

*Figure 9.1 — SecretsDump synchronizing credential material from the Active Directory database using replication permissions.*

---

## SecretsDump Analysis

The operation successfully extracted:

* NTLM password hashes.
* Kerberos AES keys.
* Machine account hashes.
* Administrator authentication material.
* Service account credential hashes.

---

## Redacted Output Example

```text
Administrator:500:<LM_HASH>:<REDACTED_NTLM_HASH>

backup:1104:<LM_HASH>:<REDACTED_NTLM_HASH>

svc-admin:1105:<LM_HASH>:<REDACTED_NTLM_HASH>
```

> **Important:** All hashes have been intentionally removed from this public portfolio.

---

## Why NTDS.dit Is Critical

The **NTDS** database stores identity secrets for the entire Active Directory domain.

### Credential Types Stored

| Secret            | Description                     |
| ----------------- | ------------------------------- |
| NTLM Hashes       | Windows authentication material |
| Kerberos Keys     | AES128 / AES256 secrets         |
| Machine Passwords | Computer accounts               |
| Service Accounts  | Authentication secrets          |

Compromise of NTDS effectively compromises the domain.

---

## Red Team Observation

DCSync is often preferred over LSASS dumping because:

* Remote execution not required.
* No memory dumping.
* Lower operational footprint.
* Native replication protocol abuse.

---

## Blue Team Detection Engineering

### Windows Event IDs

| Event ID | Description                        |
| -------- | ---------------------------------- |
| 4662     | Directory Replication              |
| 4672     | Privileged Authentication          |
| 4624     | Successful Authentication          |
| 4688     | SecretsDump Execution *(if local)* |

---

### Indicators of DCSync

Investigate:

* Replication requests from non-Domain Controllers.
* User accounts performing directory replication.
* DRSUAPI network traffic.
* Replication originating from workstations.

---

## MITRE ATT&CK Mapping

| Technique | ID            |
| --------- | ------------- |
| DCSync    | **T1003.006** |

---

<div align="center">

## Domain Credential Database Successfully Accessed

</div>

---

# Phase 10 — Administrator NTLM Hash Analysis

## Objective

Understand how NTLM authentication material enables administrator authentication without recovering plaintext passwords.

---

## NTLM Authentication Overview

Windows stores passwords as NTLM hashes.

Authentication compares hashes rather than transmitting passwords.

### NTLM Authentication Model

```text
Password
   │
   ▼
NTLM Hash
   │
   ▼
Authentication Challenge
   │
   ▼
Authenticated Session
```

---

## Why Pass-the-Hash Works

Instead of recovering:

* Password

Attackers reuse:

* NTLM Hash

This bypasses password knowledge entirely.

---

## Authentication Material Recovered

Sensitive values removed.

```text
Administrator
NTLM : <REDACTED_HASH>

Backup
NTLM : <REDACTED_HASH>
```

---

## Enterprise Security Impact

NTLM hashes allow authentication against:

* SMB.
* WinRM.
* WMI.
* PsExec.
* RPC.

without revealing plaintext passwords.

---

## Defensive Observation

Organizations should restrict NTLM authentication whenever possible and prioritize Kerberos.

---

# Phase 11 — Pass-the-Hash Authentication

## Objective

Authenticate to Windows Remote Management using the Administrator NTLM hash.

---

## Why WinRM?

WinRM provides:

* Remote PowerShell.
* Administrative automation.
* Enterprise server management.

The service was exposed during reconnaissance.

---

## Tool Used — Evil-WinRM

### Command Executed

```bash
evil-winrm \
-i <TARGET-IP> \
-u Administrator \
-H <REDACTED_NTLM_HASH>
```

---

# Screenshot — Administrator Shell

![Administrator Shell](assets/images/09-administrator-shell.png)

*Figure 11.1 — Successful Pass-the-Hash authentication resulting in an interactive Administrator PowerShell session.*

---

## Authentication Result

Administrative PowerShell access was successfully established.

### Capabilities Obtained

* PowerShell execution.
* Administrative filesystem access.
* User enumeration.
* Privileged group enumeration.
* Domain administration.

---

## Administrator Verification

Useful validation commands include:

```powershell
whoami
hostname
systeminfo
whoami /groups
whoami /priv
```

These commands confirmed administrative privileges inside the Domain Controller environment.

---

## Enterprise Security Notes

WinRM is frequently targeted because administrators commonly enable PowerShell remoting across servers.

---

## MITRE ATT&CK Mapping

| Technique       | ID            |
| --------------- | ------------- |
| Pass-the-Hash   | **T1550.002** |
| Remote Services | **T1021**     |

---

## Detection Opportunities

### Windows Event IDs

| Event ID | Description                     |
| -------- | ------------------------------- |
| 4624     | Successful WinRM Authentication |
| 4648     | Explicit Credentials Used       |
| 4688     | PowerShell Process              |
| 4104     | Script Block Logging            |

---

### PowerShell Logging Recommendations

Enable:

* Module Logging.
* Script Block Logging.
* Transcription.
* AMSI.

---

<div align="center">

## Administrator Authentication Successfully Completed

</div>

---

# Phase 12 — Post Exploitation

## Objective

Validate administrative access without establishing persistence.

This walkthrough intentionally avoids persistence mechanisms and focuses on verification.

---

## Initial Enumeration

Useful commands include:

```powershell
hostname
systeminfo
whoami
ipconfig
net users
net localgroup administrators
```

Purpose:

* Verify privilege level.
* Confirm Domain Controller.
* Enumerate administrative groups.

---

## Active Directory Enumeration

Potential PowerShell commands include:

```powershell
Get-ADUser
Get-ADComputer
Get-ADGroup
```

These commands demonstrate Active Directory visibility from an administrative context.

---

## Access Validation

Administrator privileges provided access to protected directories belonging to privileged users.

Examples include:

* Administrator Desktop.
* Backup User Desktop.
* Service Account Desktop.

---

## Operational Security Notes

No persistence was created.

The assessment did **not**:

* Create users.
* Modify groups.
* Install services.
* Schedule tasks.
* Change registry keys.

---

## Objective Achieved

Domain Administrator access successfully verified.

---

<div align="center">

## Administrative Objectives Completed

</div>

---

# Phase 13 — Challenge Completion (Redacted)

The room concludes after retrieving three challenge flags representing different privilege levels.

---

# Screenshot — Challenge Completion

![Flags Redacted](assets/images/10-flag-redacted.png)

*Figure 13.1 — Successful completion of the room with challenge flags intentionally hidden.*

---

## Flag Verification

| Challenge Objective       | Status      |
| ------------------------- | ----------- |
| User Flag                 | ✅ Retrieved |
| Privilege Escalation Flag | ✅ Retrieved |
| Root / Administrator Flag | ✅ Retrieved |

---

## Public Portfolio Policy

Challenge flags are intentionally hidden.

```text
THM{************************}
```

This preserves educational value while preventing plagiarism.

---

# Attack Timeline Summary

## End-to-End Attack Chain

```text
Nmap Enumeration
        │
        ▼
Domain Discovery
        │
        ▼
Kerberos Enumeration
        │
        ▼
AS-REP Roasting
        │
        ▼
Password Recovery
        │
        ▼
SMB Enumeration
        │
        ▼
Backup Credential Discovery
        │
        ▼
DCSync Privilege Escalation
        │
        ▼
Administrator NTLM Hash
        │
        ▼
Pass-the-Hash
        │
        ▼
Administrator PowerShell
        │
        ▼
Challenge Complete
```

---

# Red Team Success Metrics

| Objective                      | Result |
| ------------------------------ | ------ |
| Active Directory Identified    | ✅      |
| Kerberos Realm Enumerated      | ✅      |
| Valid Users Identified         | ✅      |
| Roastable Account Found        | ✅      |
| Password Cracked Offline       | ✅      |
| SMB Credentials Recovered      | ✅      |
| Replication Permissions Abused | ✅      |
| NTDS Hashes Extracted          | ✅      |
| Administrator Authentication   | ✅      |
| Challenge Completed            | ✅      |

---

<div align="center">

# Domain Compromise Successfully Achieved

**Credential Access → Privilege Escalation → Domain Administrator**

</div>

---

---

# 🛡️ Detection Engineering & Blue Team Analysis

> *Every offensive technique leaves defensive evidence.*

The final phase of this assessment transitions from attacker methodology into **Detection Engineering**. Each attack performed during the engagement is mapped to Windows telemetry, MITRE ATT&CK, Indicators of Compromise (IOCs), Sigma detection ideas, and Microsoft security monitoring recommendations.

This section is written from the perspective of a **SOC Analyst** investigating an Active Directory compromise.

---

# Detection Engineering Overview

## SOC Investigation Timeline

```text
Reconnaissance Detected
          │
          ▼
Kerberos Enumeration
          │
          ▼
AS-REP Ticket Requests
          │
          ▼
Password Cracking (Offline)
          │
          ▼
SMB Share Access
          │
          ▼
Credential Discovery
          │
          ▼
DCSync Replication Activity
          │
          ▼
Administrator WinRM Logon
          │
          ▼
PowerShell Administrative Session
```

A mature SOC should correlate these independent events into a single Active Directory compromise investigation.

---

# MITRE ATT&CK Coverage Matrix

## Enterprise ATT&CK Mapping

| ATT&CK Tactic     | Technique                  | Technique ID  |
| ----------------- | -------------------------- | ------------- |
| Reconnaissance    | Active Scanning            | T1595         |
| Discovery         | Network Service Discovery  | T1046         |
| Discovery         | Account Discovery          | T1087         |
| Credential Access | AS-REP Roasting            | **T1558.004** |
| Credential Access | Password Cracking          | **T1110.002** |
| Credential Access | Credentials in Files       | **T1552.001** |
| Credential Access | DCSync / NTDS Extraction   | **T1003.006** |
| Lateral Movement  | SMB / Windows Admin Shares | **T1021.002** |
| Lateral Movement  | WinRM Remote Services      | **T1021**     |
| Defense Evasion   | Pass-the-Hash              | **T1550.002** |
| Execution         | PowerShell                 | **T1059.001** |

---

## ATT&CK Attack Progression

```text
Reconnaissance
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

Every stage in this walkthrough maps directly to enterprise ATT&CK tactics.

---

# Windows Security Event IDs

## Primary Detection Events

| Event ID | Description                     | Detection Value                 |
| -------- | ------------------------------- | ------------------------------- |
| **4624** | Successful Logon                | WinRM / SMB Authentication      |
| **4625** | Failed Logon                    | Password Spraying / Enumeration |
| **4648** | Explicit Credential Logon       | Pass-the-Hash Detection         |
| **4662** | Directory Replication           | DCSync Detection                |
| **4672** | Privileged Logon                | Administrative Session          |
| **4688** | Process Creation                | PowerShell / SecretsDump        |
| **4768** | Kerberos TGT Request            | Username Enumeration / AS-REP   |
| **4769** | Kerberos Service Ticket         | Kerberos Activity               |
| **4771** | Kerberos Authentication Failure | Enumeration Attempts            |
| **5140** | SMB Share Access                | Backup Share Enumeration        |
| **5145** | SMB Object Access               | Sensitive File Retrieval        |

---

## Windows Event Investigation Dashboard

| Attack Stage                 | Primary Events |
| ---------------------------- | -------------- |
| Kerberos Enumeration         | 4768, 4771     |
| AS-REP Roasting              | 4768           |
| SMB Enumeration              | 5140, 5145     |
| Backup Credential Access     | 5145           |
| DCSync                       | 4662           |
| Administrator Authentication | 4624, 4672     |
| PowerShell Session           | 4688, 4104     |

---

# Kerberos Detection Engineering

## Stage 1 — Username Enumeration Detection

### Attack Behavior

* Multiple Kerberos AS Requests.
* Sequential usernames.
* Same source host.

### Investigation Questions

* Which workstation generated requests?
* How many usernames were tested?
* Were requests successful?

---

### Windows Security Logs

| Event | Meaning                        |
| ----- | ------------------------------ |
| 4768  | Authentication Service Request |
| 4771  | Failed Kerberos Authentication |

---

## Detection Logic

Suspicious characteristics include:

* Hundreds of usernames.
* Low authentication success rate.
* New workstation.
* Short request interval.

---

### Sigma Detection Example

```yaml
title: Kerberos Username Enumeration

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

## Defender for Identity Alert

Potential alert category:

* Identity reconnaissance.
* Suspicious Kerberos activity.
* Account enumeration.

---

# AS-REP Roasting Detection

## Attack Behavior

The attacker requests AS-REP responses from accounts that do not require Kerberos preauthentication.

---

## Detection Indicators

| Indicator                  | Why Important     |
| -------------------------- | ----------------- |
| Multiple AS Requests       | User Enumeration  |
| No Interactive Login       | Offline Attack    |
| Vulnerable Service Account | Identity Exposure |

---

## SOC Investigation

Questions analysts should ask:

* Which account requested AS-REP?
* Does account require preauthentication?
* Is account privileged?

---

### MITRE Mapping

**T1558.004**

---

## Sigma Concept

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

# SMB Enumeration Detection

## Attack Behavior

Authenticated SMB enumeration followed successful credential recovery.

---

## Indicators

| Event | Description   |
| ----- | ------------- |
| 5140  | Share Access  |
| 5145  | File Access   |
| 4624  | Network Logon |

---

## Investigation Workflow

```text
Successful Authentication
        │
        ▼
Share Enumeration
        │
        ▼
Sensitive Share Access
        │
        ▼
Credential Backup Download
```

---

## High Value Shares

| Share    | Investigation Priority |
| -------- | ---------------------- |
| SYSVOL   | High                   |
| NETLOGON | High                   |
| backup   | Critical               |
| ADMIN$   | Critical               |

---

## Defender Recommendations

Monitor:

* Backup share access.
* Credential file downloads.
* Administrative share enumeration.

---

# Credential Discovery Detection

## Attack Behavior

A backup credential file containing encoded credentials was downloaded and decoded.

---

## IOC Examples

| IOC                    | Explanation               |
| ---------------------- | ------------------------- |
| Base64 Credential File | Credential Storage        |
| Backup Configuration   | Secret Exposure           |
| Password Export        | Sensitive Backup Artifact |

---

## Security Recommendations

* Encrypt backup archives.
* Remove plaintext credentials.
* Rotate compromised accounts.
* Audit backup permissions.

---

# DCSync Detection Engineering

## Attack Behavior

The attacker impersonated a Domain Controller and requested directory replication.

---

## Why DCSync Is Critical

DCSync allows password extraction without:

* Interactive Administrator login.
* Memory dumping.
* Local execution.

---

## Event ID 4662

This event is the most important DCSync detection source.

### Investigate

* Replication initiated by user accounts.
* Replication outside Domain Controllers.
* Unusual DRSUAPI requests.

---

## DCSync Investigation Workflow

```text
Replication Event
        │
        ▼
Non-DC Account
        │
        ▼
Directory Changes All Permission
        │
        ▼
Credential Replication
        │
        ▼
Potential Domain Compromise
```

---

## Sigma Rule Concept

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

## Microsoft Defender for Identity

Expected detections include:

* DCSync activity.
* Replication abuse.
* Credential theft.

---

# Pass-the-Hash Detection

## Attack Behavior

Administrator authenticated using NTLM hash through WinRM.

---

## Detection Sources

| Event | Description               |
| ----- | ------------------------- |
| 4624  | Successful Authentication |
| 4648  | Explicit Credentials      |
| 4672  | Privileged Logon          |
| 4688  | PowerShell Execution      |

---

## Indicators

* Administrator WinRM login.
* NTLM authentication.
* PowerShell session.
* Remote workstation.

---

## PowerShell Logging

Enable:

* Script Block Logging.
* Module Logging.
* Transcription.
* AMSI Integration.

---

# Indicators of Compromise Dashboard

## Authentication IOCs

| IOC                           | Description          |
| ----------------------------- | -------------------- |
| Multiple Kerberos AS Requests | Username Enumeration |
| AS-REP Ticket Requests        | Kerberos Abuse       |
| NTLM WinRM Authentication     | Pass-the-Hash        |
| Replication by Backup Account | DCSync               |

---

## Network IOCs

| IOC               | Detection Source       |
| ----------------- | ---------------------- |
| LDAP Queries      | Firewall / Zeek        |
| SMB Enumeration   | Event 5140             |
| WinRM Connections | Network Logs           |
| DRSUAPI Traffic   | Defender / Network IDS |

---

## Host IOCs

| IOC                   | Investigation      |
| --------------------- | ------------------ |
| PowerShell Spawned    | Event 4688         |
| Evil-WinRM Session    | WinRM Logs         |
| SecretsDump Execution | Process Monitoring |

---

# Threat Hunting Playbook

## Hunt 1 — Kerberos Enumeration

Look for:

* High volume Event 4768.
* Many usernames.
* Single source IP.

---

## Hunt 2 — AS-REP Roasting

Look for:

* Accounts without preauthentication.
* Event 4768 anomalies.
* Service account requests.

---

## Hunt 3 — SMB Credential Theft

Look for:

* Backup share access.
* Configuration file downloads.
* Administrative shares.

---

## Hunt 4 — DCSync

Look for:

* Event 4662.
* Replication outside DCs.
* Replication by backup account.

---

## Hunt 5 — Pass-the-Hash

Look for:

* Administrator NTLM authentication.
* WinRM.
* Event 4648.

---

# Microsoft Sentinel Hunting Ideas

Potential hunting queries include monitoring:

* Kerberos Ticket Requests.
* DCSync Events.
* WinRM Authentication.
* SMB Administrative Share Access.
* PowerShell Script Blocks.

---

# Splunk Investigation Workflow

## Investigation Sequence

```text
Kerberos Events
      │
      ▼
Authentication Logs
      │
      ▼
SMB Share Access
      │
      ▼
Directory Replication
      │
      ▼
Administrator Logon
      │
      ▼
PowerShell Execution
```

Analysts correlate these stages into one incident timeline.

---

# Active Directory Threat Intelligence Summary

| Attack                      | Detection Priority           |
| --------------------------- | ---------------------------- |
| Username Enumeration        | Medium                       |
| AS-REP Roasting             | High                         |
| Password Cracking           | Offline (Limited Visibility) |
| SMB Enumeration             | High                         |
| Credential Discovery        | Critical                     |
| DCSync                      | Critical                     |
| Pass-the-Hash               | Critical                     |
| WinRM Administrator Session | Critical                     |

---

# SOC Analyst Investigation Checklist

## Identity Reconnaissance

* [ ] Event 4768 reviewed.
* [ ] Username volume analyzed.
* [ ] Source workstation identified.

---

## Credential Access

* [ ] Roastable accounts identified.
* [ ] SMB access investigated.
* [ ] Credential files reviewed.

---

## Privilege Escalation

* [ ] Event 4662 analyzed.
* [ ] Replication permissions audited.
* [ ] NTDS extraction indicators reviewed.

---

## Lateral Movement

* [ ] WinRM authentication reviewed.
* [ ] Administrator session timeline built.
* [ ] PowerShell logs collected.

---

<div align="center">

## 🔍 Blue Team Visibility Achieved

**Windows Telemetry • MITRE ATT&CK • IOC Correlation • Detection Engineering**

</div>

---

---

# Enterprise Security Hardening & Defensive Recommendations

> *Every attack path demonstrated in this assessment can be disrupted through layered defensive controls.*

The Attacktive Directory room highlights several common Active Directory misconfigurations. This section maps each offensive technique to practical defensive recommendations used in enterprise Windows environments.

Rather than focusing only on detection, the goal is to reduce the attack surface before credential abuse becomes possible.

---

# Active Directory Security Assessment

## Overall Security Posture

| Security Area            | Assessment              |
| ------------------------ | ----------------------- |
| Kerberos Configuration   | Needs Hardening         |
| Service Account Security | High Risk               |
| Password Policy          | Weak                    |
| SMB Permissions          | Sensitive Exposure      |
| Replication Permissions  | Overprivileged          |
| NTLM Authentication      | Legacy Risk             |
| WinRM Access             | Administrative Exposure |

---

## Defense-in-Depth Strategy

```text
Identity Security
      │
      ▼
Authentication Hardening
      │
      ▼
Privilege Management
      │
      ▼
Credential Protection
      │
      ▼
Monitoring & Detection
      │
      ▼
Incident Response
```

---

# Kerberos Security Hardening

## Enable Kerberos Preauthentication

The most important mitigation demonstrated in this room.

### Why It Matters

Preauthentication prevents AS-REP Roasting by requiring proof of password knowledge before authentication material is generated.

---

### Security Recommendation

* Enable Kerberos preauthentication for every user account.
* Audit legacy service accounts.
* Remove exceptions wherever possible.

---

## Audit Roastable Accounts

Regularly identify accounts configured without preauthentication.

### Review Items

* Service Accounts
* Legacy Accounts
* Disabled Accounts
* Backup Accounts

---

## Password Policy Improvements

### Enterprise Recommendations

| Control          | Recommendation           |
| ---------------- | ------------------------ |
| Minimum Length   | 14–16 Characters         |
| Complexity       | Enabled                  |
| Rotation         | Service Account Rotation |
| MFA              | Administrative Accounts  |
| Password History | Enabled                  |

---

### Service Account Best Practices

* Use **Group Managed Service Accounts (gMSA)**.
* Avoid manually managed passwords.
* Rotate credentials automatically.
* Remove interactive logon permissions.

---

<div align="center">

## Kerberos Attack Surface Reduced

</div>

---

# Active Directory Privilege Hardening

## Audit Replication Permissions

The privilege escalation stage succeeded because the backup account possessed replication permissions.

### High-Risk Permissions

| Permission                        | Recommendation          |
| --------------------------------- | ----------------------- |
| Replicating Directory Changes     | Audit Regularly         |
| Replicating Directory Changes All | Restrict                |
| Domain Replication                | Domain Controllers Only |

---

## Privileged Group Review

Review membership of:

* Domain Admins
* Enterprise Admins
* Backup Operators
* Account Operators
* Server Operators

---

## Principle of Least Privilege

Every administrative account should have:

* Minimal permissions.
* Separate privileged identity.
* Dedicated administrative workstation.

---

# SMB Security Hardening

## Protect Sensitive Shares

### Shares That Require Monitoring

| Share    | Recommendation               |
| -------- | ---------------------------- |
| SYSVOL   | Audit Read Access            |
| NETLOGON | Restrict Modifications       |
| ADMIN$   | Administrative Monitoring    |
| Backup   | Encryption + Restricted ACLs |

---

## Backup Security Recommendations

* Encrypt backups.
* Remove credentials.
* Protect configuration files.
* Restrict read permissions.
* Monitor backup downloads.

---

## SMB Monitoring Checklist

* [ ] Event ID 5140 enabled.
* [ ] Event ID 5145 enabled.
* [ ] Administrative shares audited.
* [ ] Backup share alerts configured.

---

<div align="center">

## SMB Credential Exposure Mitigated

</div>

---

# NTLM & Pass-the-Hash Mitigation

## Restrict NTLM Authentication

Pass-the-Hash relies on reusable NTLM authentication material.

### Microsoft Recommendations

* Prefer Kerberos authentication.
* Disable NTLM where possible.
* Restrict outbound NTLM.
* Restrict incoming NTLM.

---

## Enable Credential Guard

Credential Guard protects authentication secrets from theft.

### Benefits

* Protects LSASS secrets.
* Limits credential dumping.
* Reduces Pass-the-Hash exposure.

---

## Enable Windows Defender Credential Protection

Additional protections include:

* Remote Credential Guard.
* Protected Users Group.
* Restricted Admin Mode.

---

# Local Administrator Password Solution (LAPS)

## Why LAPS Matters

LAPS randomizes local administrator passwords across Windows systems.

### Benefits

* Prevents password reuse.
* Reduces lateral movement.
* Protects administrator credentials.

---

## Enterprise LAPS Benefits

| Feature            | Benefit                |
| ------------------ | ---------------------- |
| Random Passwords   | No Credential Reuse    |
| Automatic Rotation | Reduced Exposure       |
| AD Storage         | Centralized Management |

---

<div align="center">

## Lateral Movement Risk Reduced

</div>

---

# Microsoft Defender for Identity Recommendations

## Identity Protection Coverage

Microsoft Defender for Identity can detect many techniques demonstrated during this assessment.

### Detection Categories

| Detection              | Coverage |
| ---------------------- | -------- |
| AS-REP Roasting        | ✅        |
| Kerberoasting          | ✅        |
| DCSync                 | ✅        |
| Pass-the-Hash          | ✅        |
| LDAP Reconnaissance    | ✅        |
| Suspicious Replication | ✅        |

---

## Defender Investigation Workflow

```text
Identity Alert
      │
      ▼
Affected User
      │
      ▼
Authentication Timeline
      │
      ▼
Lateral Movement
      │
      ▼
Credential Theft Investigation
```

---

## Recommended Microsoft Security Features

* Microsoft Defender for Identity
* Defender for Endpoint
* Microsoft Sentinel
* Entra ID Identity Protection
* Attack Surface Reduction Rules

---

# Windows PowerShell Logging Recommendations

## Enable Advanced Logging

| Feature                  | Recommendation |
| ------------------------ | -------------- |
| Script Block Logging     | Enabled        |
| Module Logging           | Enabled        |
| PowerShell Transcription | Enabled        |
| AMSI                     | Enabled        |

---

## Why It Matters

PowerShell was used after administrative access.

Logging provides:

* Command visibility.
* Timeline reconstruction.
* Threat hunting telemetry.

---

# Zero Trust Recommendations

## Identity-Centric Security Model

Zero Trust assumes:

* Identity can be compromised.
* Authentication requires verification.
* Least privilege is enforced continuously.

---

## Zero Trust Controls

| Principle         | Implementation           |
| ----------------- | ------------------------ |
| Verify Explicitly | MFA + Conditional Access |
| Least Privilege   | RBAC + JIT Access        |
| Assume Breach     | Continuous Monitoring    |

---

## Administrative Security Model

* Dedicated Admin Accounts.
* Dedicated Admin Workstations.
* Separate Privileged Sessions.
* JIT Privileged Access.

---

<div align="center">

## Zero Trust Identity Protection

</div>

---

# Security Monitoring Dashboard

## Recommended Telemetry Sources

| Source                | Purpose                      |
| --------------------- | ---------------------------- |
| Windows Security Logs | Authentication Events        |
| Sysmon                | Process & Network Visibility |
| Defender for Endpoint | Endpoint Detection           |
| Defender for Identity | Identity Threat Detection    |
| Sentinel              | SIEM Correlation             |
| Zeek                  | Network Visibility           |

---

## SOC Correlation Workflow

```text
Windows Security Logs
        │
        ▼
Sysmon Events
        │
        ▼
Defender Alerts
        │
        ▼
Microsoft Sentinel
        │
        ▼
Incident Timeline
```

---

# Incident Response Recommendations

## If DCSync Is Detected

Immediate actions:

1. Disable compromised account.
2. Rotate privileged credentials.
3. Reset KRBTGT twice.
4. Audit replication permissions.
5. Investigate lateral movement.
6. Review Domain Controller logs.

---

## If Pass-the-Hash Is Detected

* Reset compromised credentials.
* Invalidate NTLM authentication where possible.
* Review WinRM logs.
* Review PowerShell logs.

---

## If Backup Credentials Leak

* Rotate backup account password.
* Remove plaintext secrets.
* Review backup permissions.
* Audit file access.

---

<div align="center">

## Incident Containment Strategy

</div>

---

# Lessons Learned

## Offensive Security Lessons

This room demonstrates how authentication weaknesses can become more impactful than software vulnerabilities.

### Key Offensive Concepts Learned

* Active Directory reconnaissance.
* Kerberos username enumeration.
* AS-REP Roasting.
* Offline password cracking.
* SMB credential discovery.
* DCSync replication abuse.
* NTDS extraction.
* Pass-the-Hash authentication.
* WinRM administration.

---

## Defensive Security Lessons

### Identity Protection

* Enable Kerberos preauthentication.
* Rotate service account passwords.
* Audit privileged permissions.
* Restrict replication rights.

### Monitoring

* Kerberos authentication monitoring.
* SMB auditing.
* DCSync detection.
* WinRM monitoring.

---

## SOC Analyst Takeaways

The engagement provides excellent practice for:

* MITRE ATT&CK mapping.
* Windows Event Correlation.
* Sigma detection writing.
* Active Directory threat hunting.
* Microsoft Defender investigation workflow.

---

# Skills Demonstrated During This Room

## Red Team

* Active Directory Enumeration
* Kerberos Abuse
* Credential Access
* Password Cracking
* SMB Enumeration
* NTDS Extraction
* Pass-the-Hash
* PowerShell Administration

---

## Blue Team

* Windows Event Analysis
* IOC Identification
* Sigma Rule Concepts
* Detection Engineering
* Identity Threat Monitoring
* Active Directory Hardening
* Incident Response Planning

---

<div align="center">

## Red Team + Blue Team Skill Coverage

</div>

---

# References & Further Reading

## Microsoft Documentation

* Active Directory Domain Services
* Kerberos Authentication
* Windows Event IDs
* Credential Guard
* Microsoft LAPS
* Defender for Identity

---

## MITRE ATT&CK

* Credential Access
* Discovery
* Lateral Movement
* Defense Evasion
* Windows Authentication

---

## Security Research

* Impacket Documentation
* Hashcat Documentation
* Evil-WinRM Documentation
* SigmaHQ Rules
* Microsoft Security Best Practices

---

## Training Platform

**TryHackMe — Attacktive Directory**

This documentation was produced from an authorized educational lab environment.

---

# Repository Resources

| File                                                    | Purpose                                   |
| ------------------------------------------------------- | ----------------------------------------- |
| `README.md`                                             | Repository landing page                   |
| `Resources/notes.md`                                    | Pentesting notes and commands             |
| `Documentation/Attacktive_Directory_documentation.md`   | Full technical report                     |
| `Documentation/Attacktive_Directory_documentation.docx` | Printable documentation                   |
| `docs/index.md`                                         | Premium GitHub Pages website              |
| `docs/assets/images/`                                   | Screenshots used throughout documentation |

---

# Screenshot Reference Index

| Figure      | Image                             |
| ----------- | --------------------------------- |
| Figure 2.1  | `01-nmap-recon.png`               |
| Figure 2.2  | `02-domain-discovery.png`         |
| Figure 3.1  | `03-kerberos-user-enum.png`       |
| Figure 4.1  | `04-asrep-enumeration.png`        |
| Figure 5.1  | `05-hashcat-crack.png`            |
| Figure 6.1  | `06-smb-share-enum.png`           |
| Figure 7.1  | `07-backup-credential-decode.png` |
| Figure 9.1  | `08-ntds-dump.png`                |
| Figure 11.1 | `09-administrator-shell.png`      |
| Figure 13.1 | `10-flag-redacted.png`            |

---

# Portfolio Highlights

## What This Project Demonstrates

* Enterprise Active Directory security assessment.
* Complete identity attack lifecycle.
* Kerberos authentication abuse.
* SMB credential harvesting.
* Active Directory privilege escalation.
* Detection engineering methodology.
* MITRE ATT&CK mapping.
* SOC investigation workflow.
* Security hardening recommendations.

This repository represents both **offensive security methodology** and **defensive detection engineering**.

---

# About This Portfolio

## Cybersecurity Portfolio Series

This project is part of a growing collection of professionally documented cybersecurity labs and CTF walkthroughs covering:

* Active Directory Security
* Windows Privilege Escalation
* Web Application Exploitation
* Cloud Security
* Network Security
* Detection Engineering
* Threat Hunting
* SOC Investigations

Every repository follows the same documentation standard:

* Executive Summary
* Attack Methodology
* Technical Walkthrough
* MITRE ATT&CK Mapping
* Detection Engineering Notes
* Enterprise Hardening
* GitHub Pages Documentation

---

<div align="center">

# 👨‍💻 Author

## **Anurag Revankar**

Cybersecurity • SOC Analyst • Detection Engineering • Active Directory Security • Threat Hunting

---

### 🛡️ Enterprise Active Directory Red Team Case Study

**TryHackMe — Attacktive Directory**

*Professional Cybersecurity Portfolio Documentation*

---

**Thanks for visiting this project ⭐**

If you found this documentation useful, consider starring the repository.

---

> *Built for learning. Documented for professionals. Shared for the cybersecurity community.*

</div>
