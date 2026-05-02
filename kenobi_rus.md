

Kenobi — TryHackMe
pwnboberry
## Май 2026
Задание 1: Развёртывание уязвимой машины
Nmap сканирование
nmap -sC -sV 10.114.156.150

Nmap обнаружил 7 открытых портов на целевой машине.
Вопрос: Scan the machine with nmap, how many ports are open?
## Ответ: 7
Задание 2:Перечисление (SMB)
Команда nmap -p 445 --script=smb-enum-shares.nse 10.114.156.150 не сработала
должным образом

Поэтому мы использовали более надёжную альтернативу: smbclient -L
## //10.114.156.150 -N

Вопрос: Using the nmap command above, how many shares have been found?
## Ответ: 3

Подключились к анонимной шаре и с помощью ls и нашли файл log.txt.

Чтобы не усложнять жизнь сложными командами, скачали файл через get


После получения файла log.txt его содержимое было прочитано с помощью cat.
Анализ выявил информацию об FTP-сервисе, включая порт, на котором он
работает.


Вопрос: What port is FTP running on?
## Ответ: 21

Для перечисления NFS-ресурсов использовали команду:
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.114.156.150

Вопрос: What mount can we see?
## Ответ: /var







Задание 3: : Эксплуатация ProFTPD
Определение версии ProFTPD

Вопрос: What is the version?
## Ответ: 1.3.5

Поиск эксплойтов через searchsploit

Вопрос: How many exploits are there for the ProFTPd running?
## Ответ: 4

Анализ модуля mod_copy
ProFTPD версии 1.3.5 включает модуль mod_copy, который реализует команды
SITE CPFR (копировать из) и SITE CPTO (копировать в). Эти команды
позволяют неаутентифицированному пользователю копировать файлы из
любого места файловой системы в выбранное место назначения.
Аутентификация не требуется.




Копирование SSH-ключа Kenobi


Вопрос: What is Kenobi's user flag?
Ответ: d0b0f3f53b6caa532a83915e19224899

Задание 4: Повышение привилегий через SUID
Поиск SUID-файлов
find / -perm -u=s -type f 2>/dev/null

Вопрос: What file looks particularly out of the ordinary?
## Ответ: /usr/bin/menu





Анализ бинарного файла
Запустили бинарник:

Вопрос: Run the binary, how many options appear?
## Ответ: 3
Эксплуатация — манипуляция PATH

Мы получили доступ от root и можем искать root-флаги

Вопрос: What is the root flag (/root/root.txt)?
## Ответ: 177b3cd8562289f37382721c28381f02