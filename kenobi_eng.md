# Kenobi — TryHackMe

**Author:** pwnboberry  
**Date:** May 2026  

---

## Task 1: Deploy the vulnerable machine

### Nmap scan

```bash
nmap -sC -sV 10.114.156.150
```
![](https://github.com/Pwnboberry/writeups/blob/main/images/nmap_scan.png)

Nmap scan revealed 7 open ports on the target machine.

<details>
<summary>Question: Scan the machine with nmap, how many ports are open?</summary>

7

</details>

## Task 2:Enumeration (SMB)
### The command did not work as expected
```
bash nmap -p 445 --script=smb-enum-shares.nse 10.114.156.150 
```

![](https://github.com/Pwnboberry/writeups/blob/main/images/smb_comm.png)

so we will use a more reliable alternative:
```
bash smbclient -L //10.114.156.150 -N
```

![](https://github.com/Pwnboberry/writeups/blob/main/images/smbclient_comm.png)

<details>
<summary>Question: Using the nmap command above, how many shares have been found?</summary>

3

</details>

Connected to the anonymous share and used ls to find the log.txt file.

![](https://github.com/Pwnboberry/writeups/blob/main/images/anonshare.png)

To keep things simple and avoid complicated commands, we downloaded the file
using the get command.

![](https://github.com/Pwnboberry/writeups/blob/main/images/get.png)

Once log.txt was retrieved, its contents were examined using the cat command.
The analysis revealed information regarding the FTP service, including the port on
which it is running.

![](https://github.com/Pwnboberry/writeups/blob/main/images/port21.png)

<details>
<summary>Question: What port is FTP running on?</summary>

21

</details>

Was used to enumerate NFS exportable filesystems.
```
bash nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.114.156.150
```

![](https://github.com/Pwnboberry/writeups/blob/main/images/nfs.png)

<details>
<summary>Question: What mount can we see?</summary>

/var

</details>

## Task 3: ProFTPD Exploitation
### ProFTPD Version Detection

![](https://github.com/Pwnboberry/writeups/blob/main/images/proFTPD.png)

<details>
<summary>Question: What is the version?</summary>

1.3.5

</details>

Exploit search using searchsploit
```
bash searchsploit proftpd 1.3.5
```

![](https://github.com/Pwnboberry/writeups/blob/main/images/searchsploit.png)

<details>
<summary>Question: How many exploits are there for the ProFTPd running?</summary>

4

</details>

ProFTPD version 1.3.5 includes the mod_copy module, which implements the SITE
CPFR (copy from) and SITE CPTO (copy to) commands. These commands allow an
unauthenticated user to copy files from any location on the filesystem to a
destination of their choice. No authentication is required.

![](https://github.com/Pwnboberry/writeups/blob/main/images/mod_copy.png)

Copying Kenobi's SSH key

![](https://github.com/Pwnboberry/writeups/blob/main/images/sshcopy.png)

![](https://github.com/Pwnboberry/writeups/blob/main/images/user.txt.png)

<details>
<summary>Question: What is Kenobi's user flag?</summary>

d0b0f3f53b6caa532a83915e19224899

</details>

## Task 4: SUID Privilege Escalation
### Finding SUID Binaries
```
bash find / -perm -u=s -type f 2>/dev/null
```

![](https://github.com/Pwnboberry/writeups/blob/main/images/search_suid.png)

<details>
<summary>Question: What file looks particularly out of the ordinary?</summary>

/usr/bin/menu

</details>

Analysing the Binary
Running the binary:

![](https://github.com/Pwnboberry/writeups/blob/main/images/bin.png)

<details>
<summary>Question: Run the binary, how many options appear?</summary>

3

</details>


Exploitation – PATH Manipulation

![](https://github.com/Pwnboberry/writeups/blob/main/images/PATH.png)

we have entered root and can now search for root flags 

![](https://github.com/Pwnboberry/writeups/blob/main/images/root_flag.png)

<details>
<summary>Question: What is the root flag (/root/root.txt)?</summary>

177b3cd8562289f37382721c28381f02

</details>

---
## Summary

The **Kenobi** machine has been fully compromised.  
User and root flags were obtained.

Key vulnerabilities identified:

- Anonymous access to SMB and NFS
- Outdated ProFTPD version (1.3.5) with the `mod_copy` module
- SUID binary `/usr/bin/menu`
- Unsafe `PATH` variable manipulation
