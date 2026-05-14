# Attacktive Directory — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---
The goal of the Attacktive Directory room is to compromise domain user accounts, obtain password hashes, and achieve full control over the Domain Controller.

The following steps were performed during the engagement:

- Reconnaissance and service enumeration
- Active Directory user enumeration
- AS-REP Roasting attack
- Credential harvesting via SMB
- NTDS.DIT dumping
- Pass-the-Hash authentication via WinRM

Tasks 1 and 2 must be completed independently (OpenVPN connection and starting the machine).

### Task 3: Welcome to Active Directory

### Reference:

nmap — scans ports (open/closed, service version).

enum4linux — enumerates data through these ports (users, shares, policies)

<details>
<summary>Question: What tool will allow us to enumerate port 139/445?</summary>

enum4linux

</details>

We used nmap to quickly discover the domain name.

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/nmap.png)

The scan revealed Active Directory services, including:

- Kerberos
- LDAP
- SMB
- DNS

We also obtained the network domain name.

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/thm-adm.png)

The following information was identified:

- Domain name: spookysec.local
- NetBIOS name: THM-AD

<details>
<summary>Question: What invalid TLD do people commonly use for their Active Directory Domain?</summary>

.local

</details>

### Task 4: User Enumeration with Kerberos

For this task, we used Kerbrute, a tool that identifies valid user accounts without requiring authentication.
```bash
kerbrute usernum -d <domain> -dc <IP> userlist.txt 
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/kerbrute.png)

The enumeration revealed several domain users, including:

- svc-admin
- backup

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/kerbrnames.png)

### Task 5: Exploitation Using Kerberos

The svc-admin account was particularly interesting because Kerberos pre‑authentication was disabled for this user.

Because no pre‑authentication was required, we performed an AS-REP Roasting attack and obtained a Kerberos hash for the user without needing the password.

We used Impacket GetNPUsers to retrieve the hash and saved it as hash.txt.

```bash
python3 GetNPUsers.py <domain>/<username> –no-pass –dc-ip <IP> 
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/admhash.png)

<details>
<summary>Question: Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC?</summary>

Kerberos 5 AS-REP etype 23

</details>

<details>
<summary>Question: What mode is the hash?</summary>

18200

</details>

The obtained hash was saved and passed to Hashcat:

```bash
hashcat –m 18200 -a 0 <file_name> /usr/share/wordlists/rockyou.txt
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/passwadm.png)

Result:

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/5.png)

We successfully recovered the password for the svc-admin account.

### Task 6: Back to Enumeration Basics

After obtaining domain credentials, we performed SMB share enumeration.
```bash
smbclient -L <IP> -U <domain>/<username>
```
![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/smb1.png)

<details>
<summary>Question: There is one particular share that we have access to that contains a text file. Which share is it?</summary>

Backup

</details>

We connected to the backup share via smbclient and downloaded the file.

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/smb2.png)

The file contained a Base64-encoded string:

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/backuptxt.png)

We decoded it using:
```
echo 'YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw' | base64 -d
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/base64.png)

<details>
<summary>Question: Decoding the contents of the file, what is the full contents?</summary>

backup@spookysec.local:backup2517860

</details>

### Task 7: Domain Privilege Escalation

Using the backup account credentials, we successfully dumped NTDS.DIT via the DRSUAPI method using secretsdump from the Impacket suite.
Command:

```bash
python3 secretsdump.py <domain>/backup@<IP>
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/secretsdump.png)

<details>
<summary>Question: What is the Administrators NTLM hash?</summary>

0e0363213e37b94221497260b0bcb4fc

</details>

To obtain shell access to the Domain Controller, we used the Pass-the-Hash technique with Evil-WinRM.

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/enter.png)

### Task 8: Flag Submission Panel
After successful authentication, we gained access as Administrator, confirming full Active Directory compromise.

Administrator Flag

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/1.png)

svc-admin Flag

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/2.png)

backup Flag

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/3.png)

## Conclusion

### Flags confirming system compromise were obtained
During the test, the Active Directory infrastructure was successfully compromised through an AS-REP Roasting attack, followed by SMB resource enumeration and the acquisition of backup account credentials. The use of DRSUAPI allowed the extraction of NTLM hashes for all domain users, including the Administrator, leading to full Domain Controller compromise.

## Recommendations

1. **For Kerberos:** Disable the DONT_REQUIRE_PREAUTH flag for all user accounts (except service accounts) to prevent AS-REP Roasting attacks.
2. **For SMB:**  Do not store sensitive data (passwords, keys) in public SMB shares. Restrict share access according to the principle of least privilege.
3. **For Pass-the-Hash:**  Implement multi‑factor authentication and limit the use of NTLM authentication in the domain. Monitor for suspicious WinRM connections.
4. **For Password Policy:** Enforce strong password complexity (length, special characters) and regular password changes.

---

## Flags

|    Account     | Flag |
|----------------|------|
| `svc-admin` | `TryHackMe{K3rb3r0s_Pr3_4uth}` |
| `backup` | `TryHackMe{B4ckM3UpSc0tty!}` |
| `Administrator` (root) | `TryHackMe{4ctiveD1rectoryM4st3r}` |
