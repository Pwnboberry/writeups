# Kenobi — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---

## Задание 1: Развёртывание уязвимой машины

### Nmap сканирование

```bash
nmap -sC -sV 10.114.156.150
```
![](https://github.com/Pwnboberry/writeups/blob/main/images/nmap_scan.png)

Nmap обнаружил 7 открытых портов на целевой машине.

<details>
<summary>Вопрос: Scan the machine with nmap, how many ports are open?</summary>

7

</details>

## Задание 2: Перечисление (SMB)
### Команда 
```
bash nmap -p 445 --script=smb-enum-shares.nse 10.114.156.150 
```
не сработала должным образом.

![](https://github.com/Pwnboberry/writeups/blob/main/images/smb_comm.png)

Поэтому мы использовали более надёжную альтернативу:
```bash smbclient -L //10.114.156.150 -N ```

![](https://github.com/Pwnboberry/writeups/blob/main/images/smbclient_comm.png)

<details>
<summary>Вопрос: Using the nmap command above, how many shares have been found?</summary>

3

</details>

Подключились к анонимной шаре, с помощью ls нашли файл log.txt.

![](https://github.com/Pwnboberry/writeups/blob/main/images/anonshare.png)

Чтобы не усложнять жизнь сложными командами, скачали файл через get

![](https://github.com/Pwnboberry/writeups/blob/main/images/get.png)

Прочитали log.txt через cat. Анализ выявил информацию об FTP-сервисе, включая порт.

![](https://github.com/Pwnboberry/writeups/blob/main/images/port21.png)

<details>
<summary>Вопрос: What port is FTP running on?</summary>

21

</details>

Для перечисления NFS-ресурсов использовали команду: 
```bash nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.114.156.150 ```

![](https://github.com/Pwnboberry/writeups/blob/main/images/nfs.png)

<details>
<summary>Вопрос: What mount can we see?</summary>

/var

</details>

## Задание 3: Эксплуатация ProFTPD
### Определение версии ProFTPD 

![](https://github.com/Pwnboberry/writeups/blob/main/images/proFTPD.png)

<details>
<summary>Вопрос: What is the version?</summary>

1.3.5

</details>

Поиск эксплойтов через searchsploit
```bash searchsploit proftpd 1.3.5```

![](https://github.com/Pwnboberry/writeups/blob/main/images/searchsploit.png)

<details>
<summary>Вопрос: How many exploits are there for the ProFTPd running?</summary>

4

</details>

Анализ модуля mod_copy
ProFTPD 1.3.5 включает модуль mod_copy, который реализует команды SITE CPFR (копировать из) и SITE CPTO (копировать в).
Эти команды позволяют неаутентифицированному пользователю копировать файлы из любого места файловой системы.

![](https://github.com/Pwnboberry/writeups/blob/main/images/mod_copy.png)

Копирование SSH-ключа Kenobi

![](https://github.com/Pwnboberry/writeups/blob/main/images/sshcopy.png)

![](https://github.com/Pwnboberry/writeups/blob/main/images/user.txt.png)

<details>
<summary>Вопрос: What is Kenobi's user flag?</summary>

d0b0f3f53b6caa532a83915e19224899

</details>

## Задание 4: Повышение привилегий через SUID
### Поиск SUID-файлов
```bash find / -perm -u=s -type f 2>/dev/null```

![](https://github.com/Pwnboberry/writeups/blob/main/images/search_suid.png)

<details>
<summary>Вопрос: What file looks particularly out of the ordinary?</summary>

/usr/bin/menu

</details>

Анализ бинарного файла
Запустили бинарник:

![](https://github.com/Pwnboberry/writeups/blob/main/images/bin.png)

<details>
<summary>Вопрос: Run the binary, how many options appear?</summary>

3

</details>


Эксплуатация — манипуляция PATH

![](https://github.com/Pwnboberry/writeups/blob/main/images/PATH.png)

Мы получили доступ от root и можем искать root-флаги 

![](https://github.com/Pwnboberry/writeups/blob/main/images/root_flag.png)

<details>
<summary>Вопрос: What is the root flag (/root/root.txt)?</summary>

177b3cd8562289f37382721c28381f02

</details>

---
## Итог

Машина **Kenobi** полностью скомпрометирована.  
Получены флаги пользователя и root.  

Основные уязвимости:
- Анонимный доступ к SMB и NFS
- Устаревшая версия ProFTPD (1.3.5) с модулем `mod_copy`
- SUID-бинар `/usr/bin/menu`
- Небезопасная переменная `PATH`
