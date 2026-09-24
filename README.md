# ⚔️ Attacktive Directory — Active Directory CTF Walkthrough

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=flat-square)](https://tryhackme.com/)
[![Focus](https://img.shields.io/badge/Focus-Active%20Directory-blue?style=flat-square)](#)
[![Category](https://img.shields.io/badge/Category-CTF%20Writeup-black?style=flat-square)](#)
[![Documentation](https://img.shields.io/badge/Docs-GitHub%20Pages-brightgreen?style=flat-square)](docs/)

> A portfolio-grade technical write-up documenting an authorized Active Directory attack chain from reconnaissance to administrative access.

## 🔎 What This Lab Covers

This walkthrough demonstrates:

- Network and service reconnaissance with **Nmap**
- Active Directory domain discovery
- **Kerberos** username enumeration
- **AS-REP Roasting**
- Offline password recovery with **Hashcat**
- SMB share enumeration and credential artifact discovery
- Active Directory replication abuse and **NTDS credential extraction**
- **Pass-the-Hash** authentication with Evil-WinRM
- Evidence-based reporting and defensive recommendations

## 🧭 Attack Path

```text
Nmap
  ↓
AD/DC Identification
  ↓
Domain Discovery
  ↓
Kerberos Enumeration
  ↓
AS-REP Roasting
  ↓
Offline Crack
  ↓
SMB Enumeration
  ↓
Backup Credential Artifact
  ↓
Replication Privilege Abuse
  ↓
NTDS Extraction
  ↓
Pass-the-Hash
  ↓
Administrator Shell
```

## 📚 Documentation

The full narrative is available in [`Documentation/Attacktive_Directory_documentation.md`](Documentation/Attacktive_Directory_documentation.md).

The GitHub Pages version is available from [`docs/index.md`](docs/index.md).

Quick reference notes are in [`Resources/notes.md`](Resources/notes.md).

## 🖼️ Evidence

The repository uses a small, numbered evidence set under `docs/assets/images/`. Credential and flag output is redacted in the public portfolio build.

## ⚠️ Disclaimer

This repository documents activity performed in an intentionally vulnerable training environment. The commands and techniques are provided for authorized security testing, education and lab practice. Do not apply them to systems without explicit permission.
