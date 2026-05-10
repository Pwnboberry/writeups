# Attacktive Directory — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---
Задача комнаты Attacktive Directory —  скомпрометировать учётные записи домена, получить хеши паролей 
и достичь полного контроля над контроллером домена.

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
