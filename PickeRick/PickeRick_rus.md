# PickleRick — TryHackMe

**Author:** pwnboberry  
**Date:** Май 2026  

---
Задание комнаты заключается в поиске 3 флагов пользователя в виде ингредиентов , чтобы помочь Рику вернуть человеческий облик. 
Для этого нужно взломать веб-сервер, получить доступ к системе и повысить права до root.

Запускаем машину и заходим на сайт по айпи

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/site.PNG)

Ничего интересного на главной странице мы не обнаружили, поэтому посмотрим Page Source

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/soursecode.png)

Здесь был обнаружен Username: R1ckRul3s

Чтобы провести более глубокий анализ и поиск скрытых директорий на сервере, мы используем такую команду:
```bash
gobuster dir -u http://IP -w /usr/share/wordlists/dirb/common.txt
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/gobuster.png)

Результат вывода команды:

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/gobresult.png)

Мы обнаружили скрытую папку под названием robots.txt. Проверим что там скрывается, перейдя по адресу:
```bash
http://10.112.161.78/robots.txt
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/robots.txt.png)

В результате был найдет пароль к странице входа в систему: 
```bash
Wubbalubbadubdub
```
Перейдем на страницу входа по следующему адресу:
```bash
http://10.112.161.78/login.php
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/login.png)

Успешно входим в систему по вышенайденному паролю и логину, а также проверяем папки командой ls

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/command%20panel.png)

Сразу замечаем интересный файл под названием Sup3rS3cretPickl3Ingred.txt
Так как 'cat' здесь не работает, используем альтернативу 'less' для просмотра файла

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/1st.png)

1 ингредиент найден!
<details>
mr. meeseek hair
</details>

Просмотр остальных файлов сказал нам искать остальные ингредиенты в других частях системы.

Переходим к поиску второго ингредиента
Смотрим  на список корневой директории (/) командой
```bash
ls -la /
```
![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/homels.png)

Папка 'home' выглядит довольно заманчиво, посмотрим ее

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/homels2.png)

В ней лежит папка rick, и мы делаем вывод, что в ней может быть что-то полезное для нас

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/find2sd.png)

Принимаемся за просмотр файла со вторым ингредиентом. Так как мы не можем перейти в директории командой 'cd'
Мы используем не интерактивную команду 'tee' Она просто выведет резултат нам в панель

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/2st.png)

2 ингредиент найден!
<details>
1 jerry tear
</details>

Ищем далее 3 ингредиент
Ранее в выводе 'ls -la /' мы видели директорию /root
Так как у нас есть права рут, мы можем выполнить просмотр фалов в ней

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/root_ls.png)

Просмотрим файл с 3 ингредиентом

![](https://github.com/Pwnboberry/writeups/blob/main/PickeRick/images/3rt.png)

3 ингредиент найден!
<details>
fleeb juice
</details>

## Заключение

Машина **Pickle Rick** полностью скомпрометирована.  
Получены три ингредиента (флага пользователя) и подтверждён доступ к root.

В ходе пентеста были обнаружены следующие уязвимости:

- **Утечка учётных данных** – логин найден в исходном коде главной страницы, пароль — в публично доступном файле `/robots.txt`.
- **Отсутствие ограничений в веб-шелле** – командная панель (`/portal.php`) доступна без дополнительной аутентификации и позволяет выполнять системные команды.
- **Некорректная конфигурация sudo** – пользователь `www-data` может выполнять любые команды от root без ввода пароля (`NOPASSWD: ALL`), что привело к мгновенному повышению привилегий.

## Рекомендации

1. **Не хранить учётные данные** в исходном коде страниц и публично доступных файлах (`robots.txt`).
2. **Ограничить доступ к `/portal.php`** (например, по IP-адресу или через дополнительную форму аутентификации).
3. **Удалить веб-шелл** или полностью отключить выполнение системных команд через веб-интерфейс.
4. **Настроить sudo** – убрать `NOPASSWD: ALL` для пользователя `www-data`. Если нужны отдельные команды, перечислить их явно.
5. **Регулярно обновлять ПО** и проверять конфигурацию веб-сервера на наличие уязвимостей.

---

## Флаги (ингредиенты)

| Ингредиент | Значение |
|------------|----------|
| 1 | `mr. meeseek hair` |
| 2 | `1 jerry tear` |
| 3 | `fleeb juice` |


