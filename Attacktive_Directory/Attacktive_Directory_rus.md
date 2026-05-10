# Attacktive Directory — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---
Задача комнаты Attacktive Directory —  скомпрометировать учётные записи домена, получить хеши паролей 
и достичь полного контроля над контроллером домена.

Задачи 1,2 нужно выполнить самостоятельно (подключение к openvpn и включение машины) 

### Задача 3: Добро пожаловать в Актив Директори

Справка:
nmap — сканирует порты (открыт/закрыт, версия сервиса).

enum4linux — перечисляет данные через эти порты (пользователи, шары, политики).

<details>
<summary>Вопрос: Какой инструмент позволит нам перечислить порт 139/445?</summary>

enum4linux

</details>

Но испольузем nmap для того, чтобы быстрее и легче узнать имя домена

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/nmap.png)

Результат: Домен Spookysec.local

Также узнаем сетевое доменное имя

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/thm-adm.png)

<details>
<summary>Вопрос: Какое сетевое имя NetBIOS-домена у машины?</summary>

THM-AD

</details>


<details>
<summary>Вопрос: Какой несуществующий домен верхнего уровня (TLD) часто используют люди для своего домена Active Directory?</summary>

.local

</details>

### Задача 4: Перечисление пользователей с помощью Kerberos

Используем команду для выяснения учетных записей пользователей:
```bash
kerbrute usernum –d имя домена –dc айпи userlist.txt (взяли с сайта и перенесли на кали)
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/kerbrute.png)

Нашли несколько примечательных имен

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/kerbrnames.png)

<details>
<summary>Вопрос: Какая команда в Kerbrute позволяет перечислять действительные имена пользователей?</summary>

Userenum

</details>

<details>
<summary>Вопрос: Какая примечательная учётная запись была обнаружена? (Она должна броситься в глаза)</summary>

svc-admin

</details>

<details>
<summary>Вопрос: Какая другая примечательная учётная запись была обнаружена?</summary>

Backup

</details>

### Задача 5: Эксплуатация с использованием Kerberos

<details>
<summary>У нас есть две учётные записи, у которых мы потенциально можем запросить билет. С какой учётной записи вы можете запросить билет без пароля?</summary>

svc-admin

</details>

Получаем хэш админа атакой  as rep и сохраняем в файл hash.txt командой:

```bash
Python 3 GetNPUsers.py имя домена/имя пользователя –no-pass –dc-ip айпи 
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/admhash.png)

<details>
<summary>Заглянув на страницу Hashcat Examples Wiki, какой тип хэша Kerberos мы получили от KDC? (Укажите полное имя)</summary>

Kerberos 5 AS-REP etype 23

</details>

<details>
<summary>Какой режим (mode) у этого хэша?</summary>

18200

</details>

Расшифровываем хэш админа командой:

```bash
Hashcat –m 18200 –a 0 имя файла /usr/share/wordlists/rockyou.txt
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/passwadm.png)

Результат:

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/5.png)

<details>
<summary>Теперь взломайте хэш с предоставленным модифицированным списком паролей. Каков пароль учётной записи пользователя?</summary>

management2005

</details>

### Задача 6: Вернемся к основам перечисления

<details>
<summary>Какую утилиту мы можем использовать для отображения удалённых SMB-шар?</summary>

Smbclient

</details>

<details>
<summary>Какой параметр (опция) выводит список шар?</summary>

-L

</details>

Смотрим в какие директории мы можем зайти
Команда:
```bash
Smbclient –L айпи –U домен/имя пользователя
```
![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/smb1.png)

<details>
<summary>Сколько удалённых шар отображает сервер?</summary>

6

</details>

<details>
<summary>Есть одна конкретная шара, к которой у нас есть доступ и которая содержит текстовый файл. Какая это шара?</summary>

Backup

</details>

Подключаемся к backup через smbclient и скачиваем файл себе

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/smb2.png)

Вывод файла backup_credentials.txt

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/backuptxt.png)

<details>
<summary>Каково содержимое файла?</summary>

YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw

</details>

Расшифровываем файл через ```bash echo ‘хэш’ | base64 -d ```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/base64.png)

<details>
<summary>После декодирования содержимого файла, каково полное содержимое?</summary>

backup@spookysec.local:backup2517860

</details>

### Задача 7: Повышение привилегий домена

<details>
<summary>Какой метод позволил нам сделать дамп NTDS.DIT?</summary>

DRSUAPI

</details>

Получаем хэши всех пользователей через утилиту secretsdump
Команда: 

```bash
Python3 secretsdump.py имя домена/backup@айпи
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/secretsdump.png)

<details>
<summary>Каков NTLM-хэш администратора?</summary>

0e0363213e37b94221497260b0bcb4fc

</details>

<details>
<summary>Какой метод атаки позволяет нам аутентифицироваться как пользователь без пароля?</summary>

Pass The Hash

</details>

<details>
<summary>При использовании инструмента Evil-WinRM, какая опция (параметр) позволяет нам использовать хэш?</summary>

-H

</details>

Заходим в учетную запись администратора через 

```bash
Evil-winrm –u Administrator –H хэш –i айпи
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/enter.png)

### Задача 8: Панель отправки флагов Flag

Флаг Administrator

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/1.png)

Флаг svc-admin

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/2.png)

Флаг backup

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/3.png)

## Заключение

### Все цели тестирования достигнуты. Получены флаги, подтверждающие компрометацию системы.

## Рекомендации по устранению уязвимостей

1. **For Kerberos:** Отключить флаг `DONT_REQUIRE_PREAUTH` для всех пользовательских учётных записей (кроме сервисных). Это предотвратит атаки AS-REP Roasting.
2. **For SMB:** Не хранить конфиденциальные данные (пароли, ключи) в общедоступных SMB-шарах. Ограничить доступ к шарам по принципу минимальных привилегий.
3. **For Pass-the-Hash:** Использовать многофакторную аутентификацию и ограничить использование NTLM-аутентификации в домене. Внедрить мониторинг подозрительных подключений по протоколу WinRM.
4. **For Политика паролей:** Установить требование использования сложных паролей (длина, специальные символы) и регулярную их смену.

---

## Флаги

| Учётная запись | Флаг |
|----------------|------|
| `svc-admin` | `TryHackMe{K3rb3r0s_Pr3_4uth}` |
| `backup` | `TryHackMe{B4ckM3UpSc0tty!}` |
| `Administrator` (root) | `TryHackMe{4ctiveD1rectoryM4st3r}` |
