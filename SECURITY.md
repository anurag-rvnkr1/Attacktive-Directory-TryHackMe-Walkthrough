# Security Policy

<div align="center">

# 🛡️ Security Policy

### Attacktive Directory — TryHackMe Walkthrough

**Enterprise Active Directory Security • Responsible Disclosure • Educational Research**

![Security](https://img.shields.io/badge/Security-Policy-2EA043?style=for-the-badge\&logo=github)
![Responsible Disclosure](https://img.shields.io/badge/Responsible-Disclosure-blue?style=for-the-badge)
![Educational Use](https://img.shields.io/badge/Educational-Use-green?style=for-the-badge)
![MITRE ATT\&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red?style=for-the-badge)

</div>

---

## Security Commitment

This repository documents a complete **Active Directory security assessment** performed inside the authorized **TryHackMe Attacktive Directory** laboratory environment.

The project is intended for:

* Cybersecurity education.
* Defensive security research.
* Active Directory learning.
* SOC analyst training.
* Detection engineering practice.
* Penetration testing methodology.

No techniques demonstrated in this repository should be executed against systems without **explicit written authorization**.

---

# Supported Versions

Security updates apply to the latest portfolio documentation.

| Version                              | Supported             |
| ------------------------------------ | --------------------- |
| **v2.x (Current Portfolio Edition)** | ✅ Yes                 |
| v1.x                                 | ⚠️ Documentation only |
| Older revisions                      | ❌ No                  |

The **main branch** always contains the maintained version of this documentation.

---

# Responsible Disclosure Policy

If you discover a security issue within this repository itself (such as accidentally exposed credentials, secrets, hashes, or sensitive files), please report it responsibly.

### Please report

* Accidentally committed secrets.
* API keys.
* Tokens.
* Passwords.
* Private certificates.
* NTLM hashes.
* Kerberos tickets.
* Sensitive personal information.

### Please do not report

* Challenge flags from TryHackMe.
* Intended walkthrough content.
* Educational attack techniques.
* MITRE ATT&CK references.

---

# Reporting a Security Issue

If you identify sensitive information exposed in this repository:

1. **Do not** publicly disclose the information in GitHub Issues.
2. Open a **private security report** if available.
3. If private reporting is unavailable, contact the repository owner directly before public disclosure.
4. Allow reasonable time for remediation before publishing findings.

Include:

* File name.
* Description of the issue.
* Location of the exposed content.
* Steps to reproduce (if applicable).

---

# Repository Security Scope

This repository contains only **educational documentation**.

### Included

* Markdown documentation.
* Diagrams.
* Screenshots.
* MITRE ATT&CK mappings.
* Detection engineering notes.
* Sigma rule examples.
* Windows Event IDs.
* Blue Team recommendations.

### Never Included

* Real enterprise credentials.
* Live Active Directory credentials.
* Production passwords.
* Active NTLM hashes.
* Valid Kerberos tickets.
* Secrets from real environments.

---

# Sensitive Information Handling

Every public artifact follows a strict sanitization policy.

## Challenge Flags

Challenge flags are intentionally **redacted**.

Example:

```text
THM{************************}
```

## Passwords

Passwords recovered during the lab are **never published**.

Example:

```text
Password: <REDACTED_PASSWORD>
```

## NTLM Hashes

NTLM authentication material is removed.

Example:

```text
Administrator NTLM
<REDACTED_HASH>
```

## Kerberos Tickets

AS-REP tickets are shortened and sanitized.

Example:

```text
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:<REDACTED_HASH>
```

---

# Educational Usage Policy

This repository may be used for:

* Learning Active Directory attacks.
* Studying Kerberos authentication.
* Practicing detection engineering.
* Preparing for cybersecurity interviews.
* SOC analyst learning.
* Blue Team training.

### Prohibited Uses

* Unauthorized penetration testing.
* Credential attacks against real systems.
* Password cracking outside authorized labs.
* Lateral movement in production environments.
* Active Directory exploitation without permission.

---

# Threat Model

The walkthrough demonstrates the following ATT&CK techniques in an **authorized lab**.

| ATT&CK Technique | Purpose                   |
| ---------------- | ------------------------- |
| T1589            | Username Enumeration      |
| T1087            | Account Discovery         |
| T1558.004        | AS-REP Roasting           |
| T1110.002        | Offline Password Cracking |
| T1552.001        | Credentials in Files      |
| T1003.006        | DCSync / NTDS Extraction  |
| T1550.002        | Pass-the-Hash             |
| T1021            | Remote Services (WinRM)   |

The documentation exists to explain these techniques—not encourage misuse.

---

# Secure Documentation Practices

All screenshots included in this repository have been sanitized.

### Sanitization Checklist

* [x] Flags removed.
* [x] Passwords hidden.
* [x] NTLM hashes hidden.
* [x] Kerberos tickets shortened.
* [x] IP addresses generalized where appropriate.
* [x] Sensitive identifiers removed.

---

# Detection Engineering Notice

Sigma rules and detection logic included in this repository are **educational examples**.

They are intended to demonstrate:

* Windows Event correlation.
* MITRE ATT&CK mapping.
* SOC investigation workflow.
* Threat hunting concepts.

They should be validated and customized before production deployment.

---

# Third-Party Tools Referenced

This documentation references several open-source security tools.

| Tool       | Purpose                              |
| ---------- | ------------------------------------ |
| Nmap       | Network Enumeration                  |
| Kerbrute   | Kerberos Username Enumeration        |
| Impacket   | Kerberos & Active Directory Research |
| Hashcat    | Offline Password Cracking            |
| SMBClient  | SMB Enumeration                      |
| Evil-WinRM | PowerShell Remoting                  |
| CyberChef  | Encoding/Decoding Analysis           |

All credit belongs to the respective maintainers of these projects.

---

# Defensive Security Recommendations

Organizations can reduce the attack surface demonstrated in this room by implementing:

## Identity Security

* Kerberos Preauthentication
* MFA for privileged users
* Group Managed Service Accounts (gMSA)

## Active Directory Security

* Least Privilege Administration
* Audit Replication Permissions
* Protected Users Group
* Privileged Access Workstations

## Credential Protection

* Microsoft LAPS
* Credential Guard
* NTLM Restrictions
* Password Rotation Policies

## Monitoring

* Microsoft Defender for Identity
* Microsoft Sentinel
* Sysmon
* Windows Security Event Monitoring
* PowerShell Logging

---

# Compliance With GitHub Secret Scanning

This repository is maintained to remain compatible with GitHub Secret Scanning.

### Repository Policy

* No secrets committed.
* No API tokens committed.
* No SSH private keys.
* No cloud credentials.
* No production certificates.
* No OAuth tokens.

If GitHub Secret Scanning reports a finding, it should be treated as a high-priority issue.

---

# Disclaimer

This repository demonstrates offensive security techniques **only inside an authorized educational environment**.

The repository owner does **not** authorize or encourage the use of these techniques against systems without explicit permission.

Users are responsible for ensuring their activities comply with:

* Local laws.
* Organizational policies.
* Responsible disclosure practices.
* Ethical hacking guidelines.

---

# Contact

For repository security concerns related to this documentation:

**Repository Maintainer**

**Anurag Ravikumar**

Cybersecurity • Active Directory Security • Detection Engineering • SOC Analyst

Please use responsible disclosure practices when reporting sensitive information.

---

<div align="center">

### 🛡️ Security Through Responsible Research

**Attacktive Directory — Enterprise Active Directory Security Walkthrough**

Educational • Ethical • Responsible

</div>
