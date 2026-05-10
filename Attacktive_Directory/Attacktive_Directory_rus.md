# Attacktive Directory — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---
Задача комнаты Attacktive Directory —  скомпрометировать учётные записи домена, получить хеши паролей
и достичь полного контроля над контроллером домена.

В ходе работы были выполнены:
- разведка и перечисление сервисов;
- enumeration пользователей Active Directory;
- атака AS-REP Roasting;
- получение учётных данных через SMB;
- дамп NTDS.DIT;
- Pass-the-Hash-аутентификация через WinRM.

Задачи 1,2 нужно выполнить самостоятельно (подключение к openvpn и включение машины) 

### Задача 3: Добро пожаловать в Актив Директори

### Справка:

nmap — сканирует порты (открыт/закрыт, версия сервиса).

enum4linux — перечисляет данные через эти порты (пользователи, шары, политики).

<details>
<summary>Вопрос: Какой инструмент позволит нам перечислить порт 139/445?</summary>

enum4linux

</details>

Испольузем nmap для того, чтобы быстрее и легче узнать имя домена

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/nmap.png)

В ходе сканирования были обнаружены сервисы Active Directory, включая:

- Kerberos
- LDAP
- SMB
- DNS

Также узнаем сетевое доменное имя

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/thm-adm.png)

Также удалось определить:

доменное имя: spookysec.local;
NetBIOS-имя: THM-AD.


<details>
<summary>Вопрос: Какой несуществующий домен верхнего уровня (TLD) часто используют люди для своего домена Active Directory?</summary>

.local

</details>

### Задача 4: Перечисление пользователей с помощью Kerberos

Для этого использовался инструмент Kerbrute, позволяющий определять существующие учётные записи без необходимости аутентификации.
```bash
kerbrute usernum –d имя домена –dc айпи userlist.txt (взяли с сайта и перенесли на кали)
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/kerbrute.png)

В результате enumeration были обнаружены несколько доменных пользователей, включая:

- svc-admin
- backup

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/kerbrnames.png)

### Задача 5: Эксплуатация с использованием Kerberos

Аккаунт svc-admin оказался особенно интересным, поскольку для него была отключена Kerberos pre-authentication.

Так как для учётной записи svc-admin не требовалась предварительная Kerberos-аутентификация, появилась возможность выполнить атаку AS-REP Roasting и получить Kerberos-хэш пользователя без знания пароля.

Для получения хэша использовался Impacket GetNPUsers.
Также сохраняем файл hash.txt

```bash
python3 GetNPUsers.py имя домена/имя пользователя –no-pass –dc-ip айпи 
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

Полученный хэш был сохранён и передан в Hashcat:

```bash
hashcat –m 18200 –a 0 имя файла /usr/share/wordlists/rockyou.txt
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/passwadm.png)

Результат:

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/5.png)

В результате удалось получить пароль пользователя svc-admin.

### Задача 6: Вернемся к основам перечисления

После получения доменных учётных данных было выполнено перечисление SMB-шар.
```bash
smbclient –L айпи –U домен/имя пользователя
```
![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/smb1.png)

<details>
<summary>Есть одна конкретная шара, к которой у нас есть доступ и которая содержит текстовый файл. Какая это шара?</summary>

Backup

</details>

Подключаемся к backup через smbclient и скачиваем файл себе

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/smb2.png)

Файл содержал Base64-строку:

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/backuptxt.png)

Для декодирования использовалась команда:
```
echo 'YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw' | base64 -d
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/base64.png)

<details>
<summary>После декодирования содержимого файла, каково полное содержимое?</summary>

backup@spookysec.local:backup2517860

</details>

### Задача 7: Повышение привилегий домена

Используя учётную запись backup, удалось выполнить дамп NTDS.DIT через DRSUAPI с помощью инструмента secretsdump из пакета Impacket.
Команда: 

```bash
python3 secretsdump.py имя домена/backup@айпи
```

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/secretsdump.png)

<details>
<summary>Каков NTLM-хэш администратора?</summary>

0e0363213e37b94221497260b0bcb4fc

</details>

Для получения shell-доступа к контроллеру домена использовалась техника Pass-the-Hash через Evil-WinRM.

![](https://github.com/Pwnboberry/writeups/blob/main/Attacktive_Directory/images/enter.png)

### Задача 8: Панель отправки флагов Flag
После успешной аутентификации был получен доступ от имени Administrator, что подтвердило полную компрометацию Active Directory.

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
