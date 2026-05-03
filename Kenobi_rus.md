# Kenobi — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---

## Задание 1: Развёртывание уязвимой машины

### Nmap сканирование

```bash
nmap -sC -sV 10.114.156.150
```
![](https://writeups/images/nmap_scan.png)

Nmap обнаружил 7 открытых портов на целевой машине.

[!NOTE]- Сколько портов открыто?
7

Задание 2: Перечисление (SMB)
Команда nmap -p 445 --script=smb-enum-shares.nse 10.114.156.150 не сработала должным образом.

https://images/smb_comm.png

Поэтому мы использовали более надёжную альтернативу:

bash
smbclient -L //10.114.156.150 -N
https://images/smbclient_comm.png

[!NOTE]- Сколько шаров найдено?
3

Подключились к анонимной шаре, с помощью ls нашли файл log.txt.

https://images/anonshare.png

Чтобы не усложнять, скачали файл через get:

https://images/get.png

Прочитали log.txt через cat. Анализ выявил информацию об FTP-сервисе, включая порт.

https://images/port21.png

[!NOTE]- Какой порт у FTP?
21

NFS-ресурсы
bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.114.156.150
https://images/nfs.png

[!NOTE]- Какой mount видим?
/var

Задание 3: Эксплуатация ProFTPD
Версия ProFTPD
https://images/proFTPD.png

[!NOTE]- Какая версия?
1.3.5

Поиск эксплойтов
bash
searchsploit proftpd 1.3.5
https://images/searchsploit.png

[!NOTE]- Сколько эксплойтов?
4

Анализ модуля mod_copy
ProFTPD 1.3.5 включает модуль mod_copy, который реализует команды SITE CPFR (копировать из) и SITE CPTO (копировать в).
Эти команды позволяют неаутентифицированному пользователю копировать файлы из любого места файловой системы.

https://images/mod_copy.png

Копирование SSH-ключа Kenobi
https://images/sshcopy.png

[!NOTE]- Флаг пользователя Kenobi
d0b0f3f53b6caa532a83915e19224899

Задание 4: Повышение привилегий через SUID
Поиск SUID-файлов
bash
find / -perm -u=s -type f 2>/dev/null
https://images/search_suid.png

[!NOTE]- Какой файл выделяется?
/usr/bin/menu

Анализ бинарного файла
Запустили бинарник:

https://images/bin.png

[!NOTE]- Сколько опций появляется?
3

Эксплуатация — манипуляция PATH
https://images/PATH.png
