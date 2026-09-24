# Attacktive Directory — Technical CTF Documentation

> **Platform:** TryHackMe  
> **Lab:** Attacktive Directory  
> **Focus:** Active Directory enumeration, Kerberos abuse, credential discovery, NTDS extraction and pass-the-hash  
> **Environment:** Windows Active Directory / Domain Controller  
> **Author:** Anurag R  
> **Portfolio Project:** Cybersecurity Lab Documentation

---

## 1. Executive Summary

This document records a structured assessment of the **Attacktive Directory** TryHackMe lab from initial network reconnaissance through domain compromise. The workflow demonstrates how small information disclosures can be chained into progressively stronger access:

```text
Network Reconnaissance
        ↓
Active Directory Identification
        ↓
Domain Discovery
        ↓
Kerberos User Enumeration
        ↓
AS-REP Roasting
        ↓
Offline Password Recovery
        ↓
SMB Share Enumeration
        ↓
Backup Credential Discovery
        ↓
Directory Replication Abuse
        ↓
NTDS Credential Extraction
        ↓
Pass-the-Hash
        ↓
Administrative Access
```

The lab highlights the importance of secure Kerberos configuration, least-privilege service accounts, protected backup identities, and controls around directory replication rights.

> **Flag policy:** Flag values are intentionally hidden in this public portfolio version. Screenshots containing flag output are redacted.

---

## 2. Scope & Rules of Engagement

This write-up documents activity performed against the isolated TryHackMe training target only.

| Item | Value |
|---|---|
| Target | `10.64.145.19` |
| Domain | `spookysec.local` |
| Hostname | `AttacktiveDirectory.spookysec.local` |
| Platform | Windows / Active Directory |
| Primary protocols | DNS, LDAP, SMB, Kerberos, WinRM |
| Testing purpose | Education and authorized lab practice |

No real-world systems are implied by this documentation.

---

## 3. Toolset

| Tool | Purpose |
|---|---|
| Nmap | Port and service reconnaissance |
| CrackMapExec / NetExec | SMB and domain discovery |
| Kerbrute | Kerberos user enumeration |
| Impacket GetNPUsers | AS-REP request / roasting workflow |
| Hashcat | Offline password recovery |
| smbclient | SMB share access |
| Base64 decoding utility | Credential artifact decoding |
| Impacket secretsdump | NTDS credential extraction |
| Evil-WinRM | Remote administration shell |

---

## 4. Phase I — Network Reconnaissance

The assessment began with a full TCP scan plus service/version detection.

```bash
nmap -Pn -A -p- -T4 10.64.145.19
```

### Key observations

The scan exposed an unmistakable Active Directory profile. Notable services included:

- **53/tcp** — DNS
- **80/tcp** — Microsoft IIS
- **88/tcp** — Kerberos
- **135/tcp** — MSRPC
- **139/tcp** — NetBIOS
- **389/tcp** — LDAP
- **445/tcp** — SMB
- **636/tcp** — LDAPS
- **3268/tcp** — Global Catalog LDAP
- **3389/tcp** — RDP
- **5985/tcp** — WinRM

Nmap also exposed the Windows host/domain metadata, including the DNS domain `spookysec.local` and domain-controller style service set.

![Nmap reconnaissance](../docs/assets/images/01-nmap-recon.png)

**Security interpretation:** LDAP + Kerberos + SMB + Global Catalog + WinRM strongly suggested that the target was a Windows domain controller and that identity infrastructure should be the primary enumeration path.

---

## 5. Phase II — Domain Discovery

SMB was queried to retrieve domain context:

```bash
crackmapexec smb 10.64.145.19
```

The response identified the Active Directory domain as:

```text
spookysec.local
```

![Domain discovery](../docs/assets/images/02-domain-discovery.png)

### Why this mattered

Knowing the domain allowed subsequent tooling to target Kerberos and LDAP using the correct realm/domain context instead of treating the machine as a standalone Windows host.

---

## 6. Phase III — Kerberos User Enumeration

A supplied username list was tested against the Kerberos service using Kerbrute:

```bash
kerbrute userenum -d spookysec.local --dc 10.64.145.19 users.txt
```

![Kerberos user enumeration](../docs/assets/images/03-kerberos-user-enum.png)

### Observed accounts

The enumeration exposed multiple valid identities, including service, user, administrative and backup-oriented accounts. One of the important findings was the presence of a service account that later proved vulnerable to AS-REP roasting.

> Case variations appeared in the supplied wordlist. For reporting purposes, accounts are normalized to their logical identity rather than treating case variants as distinct principals.

---

## 7. Phase IV — AS-REP Roasting

The valid-user set was supplied to Impacket's `GetNPUsers.py` to identify accounts without Kerberos preauthentication:

```bash
python3.9 /opt/impacket/examples/GetNPUsers.py spookysec.local/ \
  -dc-ip 10.64.145.19 \
  -usersfile valid_users.txt \
  -request
```

The workflow returned an **AS-REP roastable response for `svc-admin`**.

![AS-REP enumeration](../docs/assets/images/04-asrep-enumeration.png)

### Technical significance

When Kerberos preauthentication is disabled for an account, an attacker can request an AS-REP response without proving knowledge of the password first. The returned material can then be attacked offline.

This is a configuration weakness rather than an interactive exploit against the Kerberos service itself.

---

## 8. Phase V — Offline Hash Recovery

The captured AS-REP material was passed to Hashcat using the Kerberos AS-REP mode:

```bash
hashcat -m 18200 Hash.txt passwords.txt
```

![Hashcat recovery](../docs/assets/images/05-hashcat-crack.png)

The supplied lab wordlist recovered the password for the vulnerable service account.

> **Credential redaction:** The recovered password is intentionally omitted from the public portfolio narrative. It remains represented in the original lab evidence but is not reproduced here.

### Defensive lesson

Service accounts should not be configured in a way that permits unauthenticated AS-REP retrieval. Long, unique passwords and managed service identities materially reduce offline cracking risk.

---

## 9. Phase VI — SMB Share Enumeration

The recovered service-account access was used to enumerate SMB shares:

```bash
netexec smb 10.64.145.19 -u 'svc-admin' -p '<REDACTED>' --shares
```

![SMB share enumeration](../docs/assets/images/06-smb-share-enum.png)

A readable **backup** share stood out as an interesting source of configuration material.

The share was opened with:

```bash
smbclient //10.64.145.19/backup -U 'svc-admin'
```

The share contained a file named `backup_credentials.txt`.

---

## 10. Phase VII — Backup Credential Discovery

The discovered text file contained Base64-encoded data. After decoding, it revealed a second domain credential associated with the backup account.

![Backup credential artifact](../docs/assets/images/07-backup-credential-decode.png)

### Why this was significant

The newly discovered account was not merely another low-privilege identity. The lab environment granted it directory replication privileges capable of exposing directory credential material.

This was the key pivot from user-level access to domain-level credential acquisition.

---

## 11. Phase VIII — Directory Replication Abuse / NTDS Extraction

The backup account was used with Impacket's `secretsdump.py`:

```bash
secretsdump.py spookysec.local/backup:<REDACTED>@10.64.145.19
```

The tool reported that remote operations through one RPC path were denied and then used the **DRSUAPI** method to obtain Active Directory secrets.

![NTDS extraction](../docs/assets/images/08-ntds-dump.png)

### Underlying weakness

The account had directory-replication permissions, including the capability to synchronize directory changes. In a real environment, such rights are highly sensitive because they can enable extraction of password-derived material for domain principals.

### Security lesson

Directory replication permissions should be tightly scoped, audited, and granted only to identities that genuinely require them. A backup account should not automatically imply unrestricted credential extraction capability.

---

## 12. Phase IX — Pass-the-Hash and Administrative Shell

The extracted Administrator NTLM material was then used for a pass-the-hash login through Evil-WinRM:

```bash
evil-winrm -i 10.64.145.19 \
  -u 'Administrator' \
  -H '<REDACTED_NTLM_HASH>'
```

![Administrator remote shell](../docs/assets/images/09-administrator-shell.png)

A `whoami` check confirmed an administrative context.

### Why pass-the-hash worked here

NTLM authentication can, in supported configurations, authenticate a user using the NT hash without requiring the plaintext password. This is why protecting reusable NTLM material is critical even when plaintext passwords are not available.

---

## 13. Phase X — Flag Locations

The completed lab exposed three flag locations during the final stages. The **values themselves are hidden** in this public write-up to reduce direct answer duplication.

| Account / Stage | Location | Public value |
|---|---|---|
| `svc-admin` | `C:\Users\svc-admin\Desktop\user.txt.txt` | `[REDACTED]` |
| `backup` | `C:\Users\backup\Desktop\PrivEsc.txt` | `[REDACTED]` |
| `Administrator` | `C:\Users\Administrator\Desktop\root.txt` | `[REDACTED]` |

![Redacted flag capture](../docs/assets/images/10-flag-redacted.png)

> Screenshots are intentionally redacted where flag output or highly sensitive credential material was visible.

---

## 14. Attack Chain Analysis

The most useful aspect of this room is the dependency chain:

### 14.1 Identity discovery

Network services revealed that the target was an AD controller. SMB then disclosed the domain name, enabling Kerberos-aware enumeration.

### 14.2 Kerberos weakness

Username enumeration identified candidate principals and exposed a service account that did not enforce preauthentication.

### 14.3 Offline credential recovery

AS-REP data could be attacked offline. This converted an authentication configuration issue into a reusable service-account credential.

### 14.4 Trust expansion through SMB

The service account had access to a backup share containing another credential artifact. This demonstrated why secret material should never be stored in readable shares without strong access controls.

### 14.5 Privileged replication rights

The backup identity possessed replication rights that allowed acquisition of directory credential material. This became the decisive privilege-escalation step.

### 14.6 Domain compromise

The Administrator NTLM material enabled a pass-the-hash remote session, completing the lab's escalation chain.

---

## 15. Mitigation Recommendations

### Kerberos

- Require Kerberos preauthentication for normal user and service accounts.
- Audit accounts with `DONT_REQ_PREAUTH` / equivalent configuration.
- Prefer managed service identities where practical.
- Enforce strong, unique service-account secrets.

### SMB and file shares

- Apply least privilege to shares.
- Remove credential files from general-purpose shares.
- Monitor unusual access to backup repositories.
- Disable legacy protocols that are not required.

### Active Directory replication

- Minimize accounts with replication permissions.
- Periodically review `Replicating Directory Changes` and related rights.
- Alert on replication requests from unexpected hosts or accounts.
- Treat backup identities as privileged infrastructure accounts.

### NTLM

- Reduce NTLM usage where feasible.
- Protect privileged accounts from credential exposure.
- Use modern Windows authentication controls and administrative tiering.
- Monitor pass-the-hash indicators and abnormal remote administration.

---

## 16. Detection Opportunities

Useful telemetry to investigate includes:

| Activity | Example signal |
|---|---|
| Kerberos AS-REP abuse | Unexpected AS-REP requests for user/service accounts |
| SMB share discovery | Enumeration followed by access to backup-oriented shares |
| Credential-file access | Reads of files containing encoded credentials |
| Replication abuse | Directory replication operations from unusual principals |
| NTLM pass-the-hash | Remote logons where NTLM material is reused without normal password entry |
| Evil-WinRM | Unexpected WinRM sessions into administrative endpoints |

---

## 17. Skills Demonstrated

**Reconnaissance:** Nmap service mapping and AD fingerprinting.  
**Windows Security:** Kerberos, LDAP, SMB, NTLM and WinRM concepts.  
**Enumeration:** Domain and username discovery.  
**Credential Attacks:** AS-REP roasting and offline password recovery.  
**Post-Exploitation:** SMB artifact collection, directory replication abuse and pass-the-hash.  
**Reporting:** Evidence-driven attack-chain documentation and remediation mapping.

---

## 18. Lessons Learned

This lab reinforced a central Active Directory security principle: **individual weaknesses become substantially more dangerous when chained**.

A configuration issue in Kerberos enabled credential recovery. Credential reuse or poor secret storage enabled access to a backup account. Excessive replication permissions exposed directory secrets. Finally, reusable NTLM material enabled administrative remote access.

The practical defensive objective is therefore not only to patch isolated weaknesses, but to prevent the **chain** from forming.

---

## 19. Evidence Index

| No. | Evidence | File |
|---|---|---|
| 01 | Full TCP/service reconnaissance | `01-nmap-recon.png` |
| 02 | SMB/domain discovery | `02-domain-discovery.png` |
| 03 | Kerberos user enumeration | `03-kerberos-user-enum.png` |
| 04 | AS-REP enumeration | `04-asrep-enumeration.png` |
| 05 | Offline hash recovery | `05-hashcat-crack.png` |
| 06 | SMB share enumeration | `06-smb-share-enum.png` |
| 07 | Backup credential artifact | `07-backup-credential-decode.png` |
| 08 | NTDS/DRSUAPI extraction | `08-ntds-dump.png` |
| 09 | Administrator shell | `09-administrator-shell.png` |
| 10 | Redacted flag evidence | `10-flag-redacted.png` |

---

## 20. Conclusion

Attacktive Directory is a compact demonstration of an end-to-end Active Directory compromise path. The important takeaway is not any single command, but the sequence of trust transitions:

**Discovery → Identity Enumeration → Kerberos Weakness → Credential Recovery → Share Access → Replication Privilege → NTDS Extraction → Pass-the-Hash.**

The public portfolio version intentionally preserves the methodology and security reasoning while omitting reusable secrets and flag values.
