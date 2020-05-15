# [ <kbd>←</kbd> PlayGrounds ReadMe](https://github.com/mitmih/PlayGrounds/blob/master/readme.md) <a name="up">[](#up)</a>

# [ <kbd>↑</kbd> ](#up) <a name="h1">[Первичная настройка сервера на базе Ubuntu 20.04](#h1)</a>

Изначально, сразу после запуска новый сервер на Ubuntu 20.04 будет доступен пользователю `root` через `ssh` по паролю. Это не безопасно по следующим причинам:

* `root` является администратором и имеет самые широкие права в системе - можно случайно что-нибудь поломать, поэтому для повседневной работы лучше использовать ограниченную учётную запись, включая, при необходимости, повышенный доступ

* использование паролей для входа через ssh означает вероятность успешной атаки подбора пароля, а значит и взлома вашего сервера - ssh-ключи в этом плане намного безопаснее

В этом руководстве мы пошагово настроим vps-сервер "с нуля":

1. [сгенерируем пару (открытый / закрытый) ключей по алгоритму `Ed25519` и запустим новый сервер с `root`-доступом по открытому ключу](#step1)

2. [добавим нового пользователя `adam` и сделаем его `sudo`-пользователем](#step2)

3. [включим брандмауэр `ufw` и разрешим работу `OpenSSH`](#step3)

4. [настроим `adam`'у ssh-доступ по ключу](#step4)

5. [обезопасим сервис `sshd`](#step5)


## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 1 - Подготовка SSH-ключей, установка виртуального сервера](#step1)</a>

### Подготовка ключей в ОС Windows

Воспользуемся программой [`PuTTY Key Generator`](https://puttygen.com/download.php?val=46 "32-bit release 0.73")

![PuTTY Key Generator 32bit Release 0.73](01_ubuntu_20.04_server_-_first_steps_01_1.png "PuTTY Key Generator 32-bit release 0.73")

1. выбираем алгоритм `Ed25519`

2. нажимаем `Generate`

3. указываем описание (рекомендуется)

4. защищаем закрытый ключ парольной фразой (рекомендуется)

5. сохраняем закрытый ключ в файл `vdsina.ru.ppk`

6. выделяем и копируем в буфер обмена текст открытого ключа

7. добавляем текст открытого ключа через панель хостинга на вкладке [`SSH-ключи`](https://cp.vdsina.ru/sshkey/list)

### Подготовка виртуального сервера

Добавляем новый / переустанавливаем существующий сервер через хостинг-панель. Для доступа **обязательно** указываем ssh-ключ, добавленный в п.7.

Воспользуемся консольной программой [`plink.exe`](https://the.earth.li/~sgtatham/putty/latest/w32/plink.exe) для подключения к нашему серверу с использованием файла закрытого ssh-ключа:

```cmd
start "ssh" PLINK.EXE -ssh v000000.hosted-by-vdsina.ru -i vdsina.ru.ppk
```

где `v000000.hosted-by-vdsina.ru` - имя нашего виртуального сервера.

В дальнейших примерах будет использоваться `my-vps` вместо `v000000`.

1. сохраним в кэш открытый ключ нашего сервера

2. входим под `root`

3. указываем парольную фразу, которую мы установили на закрытую часть ключа в п.4

![PLINK 32-bit release 0.73](01_ubuntu_20.04_server_-_first_steps_01_2.png "подключение к серверу через PLINK (32-bit release 0.73)")

Поздравляем! `root@v000000:~#` - приглашение bash, оболочки по умолчанию в Ubuntu 20.04 - означает успешный вход на сервер. Теперь можно приступать к его настройке.

P.S. Вместо консольной `plink.exe` для ssh-подключений можно, например, воспользоваться GUI-утилитой [`KiTTY`](http://www.9bis.net/kitty/files/kitty_nocompress.exe), которая не требует установки и имеет множество различных настроек.


## [ <kbd>↑</kbd> ](#up) <a name="step2">[Шаг 2 - Добавляем нового пользователя](#step2)</a>

Первым делом обновляем список источников пакетов и сами пакеты:

```console
root@my-vps:~# apt update && apt upgrade -y
```

Пока мы вошли в систему под `root` - суперпользователем - но, в дальнейшем, всю работу на сервере будем выполнять из-под ограниченной учётной записи `adam`, в том числе и требующую прав администратора ОС.

Для этого выполним команду:

```console
root@my-vps:~$ adduser adam
Adding user `adam' ...
Adding new group `adam' (1000) ...
Adding new user `adam' (1000) with group `adam' ...
Creating home directory `/home/adam' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for adam
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
root@my-vps:~$ 
```

В процессе нужно будет установить пароль и ответить ещё на несколько вопросов.

Далее внесём его в группу `sudo`, что позволит запускать команды с повышенными привелегиями:

```console
root@my-vps:~$ usermod -aG sudo adam
```

Проверим работу новой учётной записи, подменив `root`'-а на `adam`'-а:

```console
root@my-vps:~$ su - adam
adam@my-vps:~$ whoami
adam
adam@my-vps:~$ sudo whoami
root
```

В случае ошибки, можно удалить пользователя вместе с его домашним каталогом и начать заново:

```console
root@my-vps:~$ deluser --remove-home adam
```


## [ <kbd>↑</kbd> ](#up) <a name="step3">[Шаг 3 - Включаем брандмауэр](#step3)</a>

Проверим регистрацию приложения `OpenSSH`:

```console
root@my-vps:~$ ufw app list
```

Разрешим приложению работу через брандмауэр:

```console
root@my-vps:~$ ufw allow OpenSSH
```

включим брандмауэр:

```console
root@my-vps:~$ ufw enable
```

и проверим текущее состояние:

```console
root@my-vps:~$ ufw status
```

<details>
<summary>
Как добавить программу в список `ufw`:
</summary>

> "Приложения" / "ufw app" - файлы `conf/ini`-синтаксиса в каталоге `/etc/ufw/applications.d/` с описанием и портами/протоколами, необходимыми для работы программы.
> 
> Например, добавим в список новое приложение `WireGuard`:
> 
> ```console
> root@my-vps:~$ nano /etc/ufw/applications.d/wireguard-server
> ```
> 
> содержание файла:
> 
> ```properties
> [WireGuard]
> title=WireGuard VPN service
> description=modern fast kernel-level vpn service
> ports=514/udp
> ```
> 
> Перезагрузим брандмауэр, чтобы перечитать список приложений
> 
> ```console
> root@my-vps:~$ ufw reload
> ```

</details>


## [ <kbd>↑</kbd> ](#up) <a name="step4">[Шаг 4 - включаем ssh-доступ для нового пользователя](#step4)</a>

Когда мы запустили сервер, определив ssh-ключ для доступа, открытый ключ был записан в домашней папке `root`-пользователя  в файле `/root/.ssh/authorized_keys`, посмотреть который можно так:

```console
root@my-vps:~$ cat ~/.ssh/authorized_keys
```

Самый простой способ организовать ssh-доступ для `adam`'-а - скопировать, сохранив права доступа, структуру `/root/.ssh` в его домашний каталог, используюя `rsync`:

```console
root@my-vps:~$ rsync --archive --chown=adam:adam /root/.ssh /home/adam
```

> **Примечание:** команда `rsync` по-разному обрабатывает источники и приемники с завершающим слэшем и без завершающего слэша. При использовании команды убедитесь, что исходный каталог `/root/.ssh` не содержит завершающий слэш (убедитесь, что вы не используете `/root/.ssh/`).
>
> Если вы случайно добавили завершающий слэш в команду, `rsync` скопирует *содержимое* каталога `/root/.ssh/` вместо его *структуры* в домашний каталог пользователя.
>
> Файлы будут храниться в неправильном месте, и `OpenSSH` не сможет их найти и использовать.

Проверьте вход на сервер под учётной записью `adam`. Если все ok, можно двигаться дальше и отключить вход по паролям и вход для `root`.

<details>
<summary>
P.S. Вместо использования готовых файлов `root`'-а можно пойти другим, более безопасным путём и сгенерировать новую пару ключей:
</summary>

```console
root@my-vps:~$ su - adam
adam@my-vps:~$ ssh-keygen -o -a 100 -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/adam/.ssh/id_ed25519):
Created directory '/home/adam/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/adam/.ssh/id_ed25519
Your public key has been saved in /home/adam/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:w5GRExrDAWAWHw0Br4aPWc8yymPJm5meDWdYDh4+G90 adam@my-vps.hosted-by-vdsina.ru
The key's randomart image is:
+--[ED25519 256]--+
|  *+=*+.oo       |
| o o .o+oo       |
|    o . o.       |
| . .   . .       |
|.o+.    S        |
|o*Bo.    .       |
|+B==oE           |
|.*&o             |
|+Xo.             |
+----[SHA256]-----+
```

Теперь добавим публичную часть нового ключа в список разрешённых ключей:

```console
adam@my-vps:~$ ssh-copy-id adam@my-vps.hosted-by-vdsina.ru
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/adam/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
adam@my-vps.hosted-by-vdsina.ru's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'adam@my-vps.hosted-by-vdsina.ru'"
and check to make sure that only the key(s) you wanted were added.
```

> Чтобы ключ был добавлен успешно, потребуется вводить пароль пользователя, то есть должна работать *аутентификация по паролю*. Проверьте параметр и исправьте ~~no~~ на yes
> 
> ```console
> adam@my-vps:~$ grep -i wordauth /etc/ssh/sshd_config
> PasswordAuthentication no
> ```

и ограничим доступ к файлам (любого кроме `root`-а)

```console
adam@my-vps:~$ chmod 600 ~/.ssh/{authorized_keys, id_ed25519.pub}
```

Теперь сохраните этот *закрытый* - `id_ed25519` - ключ пользователя `adam` на свой ПК, с которого вы собираетесь подключаться к серверу по `ssh`:

* либо зайдите на сервер через [WinSCP](https://winscp.net/download/WinSCP-5.17.5-Portable.zip) и скопируйте себе файл ключа

* либо выведите его в консоль и скопируйте в буфер обмена:

```console
adam@my-vps:~$ cat ~/.ssh/id_ed25519
```
Серверу нужен только публичный ключ пользователя, записанный в `~/.ssh/authorized_keys`. Закрытый ключ используется ssh-клиентом, с помощью которого вы подключаетесь, поэтому его необходимо удалить с сервера:

```console
adam@my-vps:~$ rm -f ~/.ssh/id_ed25519
```
</details>


## [ <kbd>↑</kbd> ](#up) <a name="step5">[Шаг 5 - Настройка службы `sshd`](#step5)</a>

Дальнейшие настройки выполним, войдя на сервер под пользователем `adam`, которого мы добавили на 2-ом шаге.

> Если, при запуске программ под `sudo`, вы столкнулись с ошибкой `sudo: unable to resolve host my-vps.local: Name or service not known`
> 
> Добавьте my-vps.local в `/etc/hosts`
> 
> ```console
> adam@my-vps:~$ printf "\n127.0.1.1 $(cat /etc/hostname)\n" | sudo tee -a /etc/hosts
> 
> 127.0.1.1 my-vps.local
> ```
> 
> Проверьте файл `/etc/nsswitch.conf` на наличие строки (`files` на первом месте):
> 
> ```console
> adam@my-vps:~$ cat /etc/nsswitch.conf | grep -i host
> hosts:          files dns
> ```
> 
> Перезапустите службу
> 
> ```console
> adam@my-vps:~$ sudo systemctl restart systemd-resolved.service
> ```

Сначала, на всякий случай, сделаем резервную копию оригинального файла конфига:

```console
adam@my-vps:~$ sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

и уберём у неё доступ на запись:

```console
adam@my-vps:~$ sudo chmod a-w /etc/ssh/sshd_config.backup
```

Отредактируем текущий конфиг:

```console
adam@my-vps:~$ sudo nano /etc/ssh/sshd_config
```

(используйте комбинацию `alt + shift + 3`, чтобы включить в редакторе `nano` отображение номеров строк)

```properties
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
```

Проверим изменения относительно оригинального конфига:

```console
diff -W 80 -y --suppress-common-lines /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
PermitRootLogin yes                   | PermitRootLogin no
#PubkeyAuthentication yes             | PubkeyAuthentication yes
#PasswordAuthentication yes           | PasswordAuthentication no
#PermitEmptyPasswords no              | PermitEmptyPasswords no
```

И посмотрим *текущие* настройки

```console
adam@my-vps:~$ grep -ine '^[[:alpha:]]' /etc/ssh/sshd_config
13:Include /etc/ssh/sshd_config.d/*.conf
34:PermitRootLogin no
39:PubkeyAuthentication yes
58:PasswordAuthentication no
59:PermitEmptyPasswords no
63:ChallengeResponseAuthentication no
86:UsePAM yes
91:X11Forwarding yes
95:PrintMotd no
113:AcceptEnv LANG LC_*
116:Subsystem   sftp    /usr/lib/openssh/sftp-server
```

Перезапустим службу `sshd`:

```console
adam@my-vps:~$ sudo systemctl restart sshd.service
```

Поздравляем, вы настроили хорошую основу для вашего нового сервера! Теперь он готов для дальнейшей установки необходимых сервисов и приложений.

P.S. не забывайте регулярно обновлять ваш сервер

```console
adam@my-vps:~$ sudo apt update && sudo apt upgrade -y
```

## [ <kbd>↑</kbd> ](#up)