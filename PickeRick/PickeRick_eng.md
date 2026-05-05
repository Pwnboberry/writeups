# PickleRick — TryHackMe

**Author:** pwnboberry  
**Date:** May 2026  

---
The goal of the room is to find 3 user flags in the form of ingredients to help Rick regain his human form.  
To do this, you need to hack the web server, gain system access, and escalate privileges to root.

Start the machine and visit the website by IP.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/site.PNG)

We found nothing interesting on the main page, so we checked the Page Source.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/soursecode.png)

Here we discovered a username: R1ckRul3s.

To conduct a deeper analysis and search for hidden directories on the server, we used the following command:
```bash
gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/gobuster.png)

Command output:

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/gobresult.png)

We found a hidden entry named robots.txt. Let's check what is hidden there by navigating to:
```bash
http://10.112.161.78/robots.txt
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/robots.txt.png)

As a result, we found the password for the login page:
```bash
Wubbalubbadubdub
```
Now navigate to the login page:
```bash
http://10.112.161.78/login.php
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/login.png)

We successfully logged in using the discovered password and username, then checked the folders with 'ls'

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/command%20panel.png)

We immediately noticed an interesting file named Sup3rS3cretPickl3Ingred.txt.

Since 'cat' is blocked here, we used 'less' as an alternative to view the file.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/1st.png)

### 1st ingredient found!
<details>
<summary>Question: What is the first ingredient that Rick needs?</summary>

mr. meeseek hair

</details>


Viewing the remaining files told us to look for the other ingredients elsewhere in the system.

Now we move on to the second ingredient.
We listed the root directory (/) with:
```bash
ls -la /
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/homels.png)

The 'home' folder looked promising, so we checked it.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/homels2.png)

Inside, we found the folder 'rick' and assumed it might contain something useful.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/find2sd.png)

We attempted to read the second ingredient file. Since we cannot change directories with cd,
we used the non-interactive command tee to print the result to the panel.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/2st.png)

### 2nd ingredient found!
<details>
<summary>Question: What is the second ingredient in Rick’s potion?</summary>

1 jerry tear

</details>

## Next, the 3rd ingredient
Earlier, when we ran 'ls -la /', we saw the /root directory.
Since we have root privileges, we can list files there.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/root_ls.png)

We then viewed the file containing the data.

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/3rt.png)

### 3rd ingredient found!

<details>
<summary>Question: What is the last and final ingredient?</summary>

fleeb juice

</details>

## Conclusion

The Pickle Rick machine has been fully compromised.
Three ingredients (user flags) were obtained, and root access was confirmed.

### The following vulnerabilities were discovered during the pentest:

- Credential disclosure – the username was found in the source code of the main page, and the password in the publicly accessible /robots.txt file.
- Lack of restrictions in the web shell – the command panel (/portal.php) is accessible without additional authentication and allows system command execution.
- Incorrect sudo configuration – the www-data user can run any command as root without a password (NOPASSWD: ALL), which led to immediate privilege escalation.

## Recommendations
1. Do not store credentials in page source code or publicly accessible files (robots.txt).
2. Restrict access to /portal.php (e.g., by IP address or an additional authentication form).
3. Remove the web shell or completely disable system command execution through the web interface.
4. Fix sudo configuration – remove NOPASSWD: ALL for the www-data user. If specific commands are needed, list them explicitly.
5. Keep software up to date and regularly check the web server configuration for vulnerabilities.

---

## Flags (ingredients)

| Ingredient | Value |
|------------|----------|
| 1 | `mr. meeseek hair` |
| 2 | `1 jerry tear` |
| 3 | `fleeb juice` |

