# [ <kbd>←</kbd> PlayGrounds ReadMe](https://github.com/mitmih/PlayGrounds/blob/master/readme.md) <a name="up">[](#up)</a>

# [ <kbd>↑</kbd> ](#up) <a name="h1">[Запуск VPN-сервера WireGuard на базе Ubuntu 20.04](#h1)</a>

VPN - Virtual Private Network - Виртуальная Частная Сеть - множество технологий, с помощью которых вы можете поверх публичной сети, как правило -Интернет, построить свою *логическую (виртуальную)* сеть.

Наверное, вы уже столкнулись с одной из таких технологий, когда провайдер организовал вам доступ в Сеть: очень часто абонентов подключают по технологии PPPoE или L2TP.

Работа в VPN, в зависимости от качества каналов связи конечно, не будет для вас заметно отличаться от работы в домашней или офисной *физической* сети.

Технологии VPN применяются и крупными компаниями и простыми пользователями с очень разными целями. Например:

* корпорации таким образом связывают между собой локальные сети своих филиалов

* также это хороший способ организовать безопасно подкючить к корпоративной сети удалённых сотрудников в условиях, например, гостиничного WiFi

* обычне пользователи могут с помощью VPN [сэкономить при покупках](https://habr.com/ru/company/hidemy_name/blog/404017/)

* или покупать товары в тех зарубежных магазинах, которые не продают, если заходить на их сайты напрямую

Самые простые способы - бесплатные VPN-сервисы - часто оказываются и самыми ненадёжными в плане конфеденциальности, [зарабатывая на ваших личных данных.](https://habr.com/ru/company/hidemy_name/blog/450416/)

Более сложный, и в тоже время надёжный способ заключается в организации собственного WireGuard / Outline / OpenVPN сервиса.

Данная статья рассматривает настройку [WireGuard](https://www.wireguard.com/performance/#results) - достаточно новый и активно развивающийся проект, быстрый в работе, простой в настройке.

## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 1 - Установка и первичная настройка](#step1)</a>

Основой для WireGuard-сервиса будет приготовленный по [этому рецепту](https://github.com/mitmih/PlayGrounds/blob/master/VPS/01_ubuntu_20.04_server_-_first_steps.md) сервер.

WireGuard можно [собрать из исходников](https://www.wireguard.com/compilation/) или установить готовый пакет из репозитория через пакетный менеджер:

```console
adam@my-vps:~$ sudo apt install wireguard -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  wireguard-tools
The following NEW packages will be installed:
  wireguard wireguard-tools
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 85.3 kB of archives.
After this operation, 341 kB of additional disk space will be used.
Get:1 http://mirror.nl.leaseweb.net/ubuntu focal/universe amd64 wireguard-tools amd64 1.0.20200319-1ubuntu1 [82.4 kB]
Get:2 http://mirror.nl.leaseweb.net/ubuntu focal/universe amd64 wireguard all 1.0.20200319-1ubuntu1 [2,912 B]
Fetched 85.3 kB in 0s (1,124 kB/s)
Selecting previously unselected package wireguard-tools.
(Reading database ... 101648 files and directories currently installed.)
Preparing to unpack .../wireguard-tools_1.0.20200319-1ubuntu1_amd64.deb ...
Unpacking wireguard-tools (1.0.20200319-1ubuntu1) ...
Selecting previously unselected package wireguard.
Preparing to unpack .../wireguard_1.0.20200319-1ubuntu1_all.deb ...
Unpacking wireguard (1.0.20200319-1ubuntu1) ...
Setting up wireguard-tools (1.0.20200319-1ubuntu1) ...
Setting up wireguard (1.0.20200319-1ubuntu1) ...
Processing triggers for man-db (2.9.1-1) ...
```

Далее генерируем пару, закрытый и открытый, ключей для нашего сервера:

```console
adam@my-vps:~$ (umask 077 && printf "[Interface]\nPrivateKey = " | sudo tee /etc/wireguard/wg0.conf > /dev/null)
adam@my-vps:~$ wg genkey | sudo tee -a /etc/wireguard/wg0.conf | wg pubkey | sudo tee /etc/wireguard/publickey
yMXfyyT9m3fkoUTnW/mOMaq/WspQoOZyL02oUftwDkI=
```

Здесь

* `/etc/wireguard/wg0.conf` - конфиг виртуального сетевого интерфейса, который будет шифровать и обрабатывать трафик между сервером и клиентами, также здесь записан закрытый, т.е. *секретный* ключ сервера в формате `base64`

    ```console
    adam@my-vps:~$ sudo cat /etc/wireguard/wg0.conf
    [Interface]
    PrivateKey = cCCJ27hpJvdIVmCAvVk8FRAt9caNR3zpH5eeUoL3uHg=
    ```

* `/etc/wireguard/publickey` - публичный ключ сервера, который прописывается на клиентах

    ```console
    adam@my-vps:~$ sudo cat /etc/wireguard/publickey
    yMXfyyT9m3fkoUTnW/mOMaq/WspQoOZyL02oUftwDkI=
    ```

Естесственно, ваша пара ключей будет другой, поэтому дальше в примерах конфигов сервера или клиентов вместо ключей будут указаны псевдонимы:

* `server public key` - открытый ключ сервера

* `server private key` - закрытый (секретный) ключ сервера

* `client public key` - открытый ключ клиента

* `client private key` - закрытый (секретный) ключ клиента


## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 2 - редактирование конфига](#step1)</a>





adam@my-vps:~$ 
=
## [ <kbd>↑</kbd> ](#up) <a name="step1">[](#step1)</a>