

Kenobi — TryHackMe
pwnboberry
## Май 2026
Задание 1: Развёртывание уязвимой машины
Nmap сканирование
nmap -sC -sV 10.114.156.150
## (images/nmap_scan.png)

Nmap обнаружил 7 открытых портов на целевой машине.
Вопрос: Scan the machine with nmap, how many ports are open?
[!NOTE]- Ответ
## 7
Задание 2:Перечисление (SMB)
Команда nmap -p 445 --script=smb-enum-shares.nse 10.114.156.150 не сработала
должным образом
## (images/smb_comm.png)

Поэтому мы использовали более надёжную альтернативу: smbclient -L //10.114.156.150
## -N
## (images/smbclient_comm.png)

Вопрос: Using the nmap command above, how many shares have been found?
[!NOTE]- Ответ
## 3

Подключились к анонимной шаре и с помощью ls и нашли файл log.txt.
## (images/anonshare.png)

Чтобы не усложнять жизнь сложными командами, скачали файл через get
## (images/get.png)

После получения файла log.txt его содержимое было прочитано с помощью cat. Анализ
выявил информацию об FTP-сервисе, включая порт, на котором он работает.
## (images/port21.png)

Вопрос: What port is FTP running on?
[!NOTE]- Ответ
## 21

Для перечисления NFS-ресурсов использовали команду:
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.114.156.150
## (images/nfs.png)

Вопрос: What mount can we see?
[!NOTE]- Ответ
## /var

Задание 3: : Эксплуатация ProFTPD

Определение версии ProFTPD
(images/proFTPD.png)

Вопрос: What is the version?
[!NOTE]- Ответ
## 1.3.5

Поиск эксплойтов через searchsploit
## (images/searchsploit.png)

Вопрос: How many exploits are there for the ProFTPd running?
[!NOTE]- Ответ
## 4

Анализ модуля mod_copy
ProFTPD версии 1.3.5 включает модуль mod_copy, который реализует команды SITE
CPFR (копировать из) и SITE CPTO (копировать в). Эти команды позволяют
неаутентифицированному пользователю копировать файлы из любого места
файловой системы в выбранное место назначения. Аутентификация не требуется.
## (images/mod_copy.png)

Копирование SSH-ключа Kenobi
## (images/sshcopy.png)

Вопрос: What is Kenobi's user flag?
[!NOTE]- Ответ
d0b0f3f53b6caa532a83915e19224899

Задание 4: Повышение привилегий через SUID
Поиск SUID-файлов
find / -perm -u=s -type f 2>/dev/null
## (images/search_suid.png)

Вопрос: What file looks particularly out of the ordinary?
[!NOTE]- Ответ
## /usr/bin/menu


Анализ бинарного файла
Запустили бинарник:
## (images/bin.png)

Вопрос: Run the binary, how many options appear?
[!NOTE]- Ответ
## 3

Эксплуатация — манипуляция PATH
(images/PATH.png)



