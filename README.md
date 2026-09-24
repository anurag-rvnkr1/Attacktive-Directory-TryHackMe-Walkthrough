# 👻 Attacktive Directory — TryHackMe Walkthrough

<div align="center">

# 🛡️ Attacktive Directory

### Enterprise Active Directory Attack Path • Kerberos Abuse • Pass-the-Hash • Domain Compromise

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Attacktive_Directory-red?style=for-the-badge\&logo=tryhackme)](https://tryhackme.com/)
[![Active Directory](https://img.shields.io/badge/Windows-Active_Directory-0078D6?style=for-the-badge\&logo=windows)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)]
[![Writeup](https://img.shields.io/badge/Writeup-Portfolio_Grade-6f42c1?style=for-the-badge\&logo=github)]
[![Blue Team](https://img.shields.io/badge/Blue_Team-Detection_Notes-0EA5E9?style=for-the-badge)]
[![Red Team](https://img.shields.io/badge/Red_Team-AD_Enumeration-red?style=for-the-badge)]

**Professional cybersecurity walkthrough documenting a complete compromise of a Windows Active Directory domain through Kerberos enumeration, AS-REP Roasting, SMB credential discovery, DCSync abuse, NTDS extraction, and Pass-the-Hash authentication.**

---

### 👨‍💻 Author

**Anurag Revankar**

Cybersecurity | SOC Analyst | Active Directory Security | Detection Engineering

</div>

---

## 📌 Overview

**Attacktive Directory** is a beginner-friendly Active Directory room on **TryHackMe** that introduces one of the most important attack paths in enterprise Windows environments.

This walkthrough demonstrates a realistic Red Team methodology against an Active Directory Domain Controller, progressing from initial reconnaissance to full Domain Administrator access while documenting each phase professionally for cybersecurity portfolio purposes.

Unlike a simple CTF write-up, this repository focuses on:

* Enterprise attack methodology.
* Active Directory internals.
* Kerberos abuse techniques.
* Security impact and defensive recommendations.
* Professional documentation suitable for GitHub Pages.

---

## 🧠 Skills Demonstrated

| Area                      | Techniques Covered                      |
| ------------------------- | --------------------------------------- |
| Reconnaissance            | Nmap Service Enumeration, SMB Discovery |
| Active Directory          | Domain Enumeration, LDAP Identification |
| Kerberos                  | Username Enumeration, AS-REP Roasting   |
| Password Attacks          | Offline Hash Cracking using Hashcat     |
| SMB Enumeration           | Share Discovery and Credential Hunting  |
| Credential Access         | Base64 Credential Recovery              |
| Privilege Escalation      | DCSync / NTDS Extraction                |
| Lateral Movement          | Pass-the-Hash Authentication            |
| Windows Post Exploitation | Evil-WinRM Remote PowerShell            |
| Reporting                 | Professional Incident Documentation     |

---

# 🗺️ Attack Path Overview

> Complete attack chain documented throughout this walkthrough.

```text
Internet Access
      │
      ▼
Nmap Enumeration
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
SMB Share Enumeration
      │
      ▼
Credential Discovery
      │
      ▼
Backup Account Compromise
      │
      ▼
DCSync / NTDS Extraction
      │
      ▼
Administrator NTLM Hash
      │
      ▼
Pass-the-Hash (Evil-WinRM)
      │
      ▼
Domain Administrator Access
```

---

# ⚙️ Lab Information

| Property            | Value                               |
| ------------------- | ----------------------------------- |
| Platform            | TryHackMe                           |
| Machine             | Attacktive Directory                |
| Operating System    | Windows Server 2019                 |
| Environment         | Active Directory Domain Controller  |
| Domain              | `spookysec.local`                   |
| Objective           | Obtain User, PrivEsc and Root Flags |
| Documentation Style | Portfolio / Educational             |

---

# 🛠️ Tools Used

| Tool                   | Purpose                       |
| ---------------------- | ----------------------------- |
| Nmap                   | Network reconnaissance        |
| CrackMapExec / NetExec | SMB & Domain Enumeration      |
| Kerbrute               | Kerberos Username Enumeration |
| Impacket GetNPUsers    | AS-REP Roasting               |
| Hashcat                | Offline Password Cracking     |
| SMBClient              | SMB Share Access              |
| CyberChef              | Base64 Credential Decoding    |
| Impacket SecretsDump   | NTDS / DCSync Extraction      |
| Evil-WinRM             | Pass-the-Hash Remote Shell    |

---

# 📸 Walkthrough Preview

> Screenshots have been recreated and organized for portfolio presentation.

| Step                           | Screenshot                                    |
| ------------------------------ | --------------------------------------------- |
| Initial Nmap Enumeration       | `Screenshots/01-nmap-recon.png`               |
| Domain Discovery               | `Screenshots/02-domain-discovery.png`         |
| Kerberos User Enumeration      | `Screenshots/03-kerberos-user-enum.png`       |
| AS-REP Roasting                | `Screenshots/04-asrep-enumeration.png`        |
| Password Cracking              | `Screenshots/05-hashcat-crack.png`            |
| SMB Enumeration                | `Screenshots/06-smb-share-enum.png`           |
| Credential Recovery            | `Screenshots/07-backup-credential-decode.png` |
| NTDS Extraction                | `Screenshots/08-ntds-dump.png`                |
| Administrator Shell            | `Screenshots/09-administrator-shell.png`      |
| Flag Verification *(Redacted)* | `Screenshots/10-flag-redacted.png`            |

---

# 🔬 Key Attack Techniques Explained

## Kerberos Username Enumeration

Validated domain usernames without requiring credentials.

**Security Concept**

* Kerberos authentication leaks username validity.
* Used for attack surface discovery.

**MITRE ATT&CK**

`T1589` • Gather Victim Identity Information

---

## AS-REP Roasting

Requested Kerberos AS-REP tickets from accounts without Kerberos preauthentication.

**Impact**

* Offline password cracking.
* No authentication required.

**MITRE ATT&CK**

`T1558.004`

---

## Offline Password Cracking

Recovered credentials using GPU-assisted offline cracking.

**Why It Works**

Weak passwords + AS-REP hash exposure.

**MITRE ATT&CK**

`T1110.002`

---

## SMB Share Enumeration

Authenticated enumeration of network shares using compromised credentials.

**Findings**

* Backup share.
* Sensitive credential file.

**MITRE ATT&CK**

`T1021.002`

---

## DCSync / NTDS Extraction

Abused replication permissions assigned to the Backup account.

**Security Concept**

Replicating Directory Changes permissions allow retrieval of NTDS password hashes.

**MITRE ATT&CK**

`T1003.006`

---

## Pass-the-Hash

Authenticated to WinRM using Administrator NTLM hash without the plaintext password.

**MITRE ATT&CK**

`T1550.002`

---

# 🧱 Repository Structure

```text
Attacktive-Directory-TryHackMe-Walkthrough/
│
├── README.md
├── SECURITY.md
├── LICENSE
│
├── Screenshots/
│   ├── 01-nmap-recon.png
│   ├── 02-domain-discovery.png
│   ├── 03-kerberos-user-enum.png
│   ├── 04-asrep-enumeration.png
│   ├── 05-hashcat-crack.png
│   ├── 06-smb-share-enum.png
│   ├── 07-backup-credential-decode.png
│   ├── 08-ntds-dump.png
│   ├── 09-administrator-shell.png
│   └── 10-flag-redacted.png
│
├── Resources/
│   └── notes.md
│
├── Documentation/
│   ├── Attacktive_Directory_documentation.md
│   └── Attacktive_Directory_documentation.docx
│
└── docs/
    ├── index.md
    └── assets/
        ├── css/
        └── images/
```

---

# 📚 Documentation Included

This repository contains multiple documentation formats.

| File                                                    | Description                                                    |
| ------------------------------------------------------- | -------------------------------------------------------------- |
| `README.md`                                             | Professional GitHub landing page.                              |
| `Resources/notes.md`                                    | Pentesting notes, commands, MITRE mapping, defensive insights. |
| `Documentation/Attacktive_Directory_documentation.md`   | Detailed technical walkthrough with explanations.              |
| `Documentation/Attacktive_Directory_documentation.docx` | Recruiter-friendly printable report.                           |
| `docs/index.md`                                         | GitHub Pages premium portfolio documentation.                  |

---

# 🛡️ Detection & Defensive Notes

| Attack               | Defensive Control                             |
| -------------------- | --------------------------------------------- |
| Username Enumeration | Kerberos auditing, account lockout monitoring |
| AS-REP Roasting      | Enforce Kerberos Preauthentication            |
| Weak Passwords       | Password Policy + MFA                         |
| SMB Enumeration      | Least Privilege Share Permissions             |
| DCSync Abuse         | Audit Replication Permissions                 |
| Pass-the-Hash        | Credential Guard, NTLM Restrictions, LAPS     |

---

# 📖 MITRE ATT&CK Coverage

| Tactic             | Technique                    |
| ------------------ | ---------------------------- |
| Discovery          | T1087, T1018                 |
| Credential Access  | T1558.004, T1110.002         |
| Lateral Movement   | T1021.002                    |
| Credential Dumping | T1003.006                    |
| Defense Evasion    | T1550.002                    |
| Persistence        | Replication Abuse Discussion |

---

# 📑 Learning Outcomes

After completing this room, you'll understand:

* Windows Active Directory architecture.
* Kerberos authentication workflow.
* AS-REP Roasting attack lifecycle.
* SMB authentication and share enumeration.
* DCSync attack mechanics.
* NTDS password extraction.
* Pass-the-Hash authentication.
* Remote PowerShell administration using Evil-WinRM.
* Defensive mitigations against common AD attacks.

---

# ⚠️ Ethical Use Disclaimer

This repository is provided **strictly for educational purposes**.

The techniques demonstrated were performed inside the authorized **TryHackMe** laboratory environment.

Do **NOT** use these techniques against systems without explicit authorization.

---

# 🌐 GitHub Pages Portfolio

A premium GitHub Pages site accompanies this repository.

## Portfolio Features

* Hacker-style Jekyll documentation.
* Interactive attack timeline.
* MITRE ATT&CK mapping.
* Detection engineering notes.
* Professional screenshots.
* Responsive cyber-themed styling.

> Visit the **GitHub Pages** deployment from the repository after enabling Pages.

---

# 🤝 Connect With Me

**Anurag Revankar**

Cybersecurity • SOC • Threat Detection • Active Directory Security • TryHackMe Portfolio

If you found this documentation useful, consider ⭐ starring the repository.

---

<div align="center">

### 💀 Red Team Methodology • Blue Team Insights • Professional Cybersecurity Documentation

**Attacktive Directory — Complete Active Directory Attack Chain Documentation**

Made with ❤️ for cybersecurity learning and portfolio development.

</div>
