# [ <kbd>←</kbd> PlayGrounds ReadMe](https://github.com/mitmih/PlayGrounds/blob/master/readme.md) <a name="up">[](#up)</a>

# [ <kbd>↑</kbd> ](#up) <a name="h1">[Запуск vpn-сервиса WireGuard на базе Ubuntu 20.04](#h1)</a>

VPN - Virtual Private Network - Виртуальная Частная Сеть - множество технологий, с помощью которых вы можете поверх публичной сети, как правило -Интернет, построить свою *логическую (виртуальную)* сеть.

Наверное, вы уже столкнулись с одной из таких технологий, когда провайдер организовал вам доступ в Сеть: очень часто абонентов подключают по технологии PPPoE или L2TP.

Работа в VPN, в зависимости от качества каналов связи конечно, не будет для вас заметно отличаться от работы в домашней или офисной *физической* сети.

Технологии VPN применяются и крупными компаниями и простыми пользователями с очень разными целями. Например:

* корпорации таким образом связывают между собой локальные сети своих филиалов

* также это хороший способ организовать безопасно подкючить к корпоративной сети удалённых сотрудников в условиях, например, гостиничного WiFi

* обычне пользователи могут с помощью VPN [сэкономить при покупках](https://habr.com/ru/company/hidemy_name/blog/404017/)

* или покупать товары в тех зарубежных магазинах, которые не продают, если заходить на их сайты напрямую

Самые простые способы - бесплатные VPN-сервисы - часто оказываются и самыми ненадёжными в плане конфеденциальности, [зарабатывая на ваших личных данных.](https://habr.com/ru/company/hidemy_name/blog/450416/)

Более сложный, и в тоже время надёжный способ заключается в организации собственного WireGuard / Outline / OpenVPN сервиса, который позволит перенаправить через себя весь наш трафик, попутно выполняя его шифрование.

[WireGuard](https://www.wireguard.com/performance/#results) - новый и успешно развивающийся проект, производительный и простой в настройке. Рецепт приготовления сервиса по шагам:

* [ставим WireGuard, готовим ключи](#step1)

* [готовим виртуальный сетевой интерфейс, нарезаем клиентские настройки ломтиками](#step2)

* [разогреваем Windows-клиента](#step3)

<!-- * [step4](#step4)

* [step5](#step5)

* [step6](#step6)

* [step7](#step7) -->


## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 1 - Устанавливаем WireGuard, разбираемся с серверными ключами шифрования](#step1)</a>

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


## [ <kbd>↑</kbd> ](#up) <a name="step2">[Шаг 2 - конфигурация виртуального сетевого интерфейса wg-сервера](#step2)</a>

Откроем в редакторе конфиг `wg0` - виртуального сетевого интерфейса:

```console
adam@my-vps:~$ sudo nano /etc/wireguard/wg0.conf
```

и добавим необходимые настройки

```properties
[Interface]
PrivateKey = server private key
Address = 10.0.0.1/24
ListenPort = 514
SaveConfig = true

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE

[Peer]
AllowedIPs = 10.0.0.2/32
# PublicKey = client public key
```

Где

* `[Interface]` - секция, отвечающая за настройки интерфейса `wg0`

* `Address = 10.0.0.1/24` - IP-адрес сервера в нашей vpn-сети, через `,` можно указать несколько адресов

* `ListenPort = 514` - порт, на который сервер принимает клиентские подключения

* `SaveConfig = true` - сохранять в конфиг текущие настройки `wg0` при выключении - позволяет серверу подстраиваться под клиентов

* `PostUp = ...` и `PostDown = ...` - команды, которые выполняются при включении и выключении интерфейса `wg0`

* `ens3` - сетевой интерфейс, который по умолчанию использует сервер для выхода в Интернет, можно узнать командой:

    ```console
    adam@my-vps:~$ ip -o -4 route show to default | awk '{print $5}'
    ```

* `[Peer]` - клиентские настройки, у каждого клиента своя отдельная секция `[Peer]`

* `AllowedIPs = 10.0.0.2/32` - IP-адрес клиента в vpn-сети

* `# PublicKey = client public key` - запишем сюда открытый ключ клиента, когда настроим на нём WireGuard и получим пару ключей, а пока оставим эту строку закомментированной

Наша цель - пропускать через WireGuard весь трафик, *от* и *к* клиенту. Поэтому нужно, чтобы работал NAT.

Отредактируем файл `/etc/sysctl.conf`, разрешив перенаправление IP трафика:

```console
adam@my-vps:~$ sudo nano /etc/sysctl.conf
```

```properties
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Проверим изменения:

```console
adam@my-vps:~$ sudo sysctl -p
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```

<!--     
    # https://forum.keenetic.net/topic/7868-перенаправление-траффика-в-wg0/
    # net.ipv4.tcp_fwmark_accept=1
    # net.ipv6.conf.default.forwarding=1
    # net.ipv6.conf.all.forwarding=1
 -->

У нас включен брандмауэр `ufw` - разрешим UDP-трафик на 514 порту:

```console
adam@my-vps:~$ sudo ufw allow 514/udp
Rule added
Rule added (v6)
```


## [ <kbd>↑</kbd> ](#up) <a name="step3">[Шаг 3 - Настройка клиента в ОС Windows](#step3)</a>

Скачиваем клиент [с официального сайта](https://download.wireguard.com/windows-client/wireguard-amd64-0.1.0.msi) и устанавливаем.

Запустим и добавим новый туннель <kbd>ctrl</kbd> <kbd>N</kbd>:

![Нажмите ctrl+N для добавления нового туннеля](02_wireguard_vpn_server_03_1.png "Добавление нового туннеля")

Настроим его следующим образом:

```properties
[Interface]
PrivateKey = client private key
ListenPort = 514
Address = 10.0.0.2/24
DNS = 1.1.1.1, 8.8.8.8

[Peer]
PublicKey = server public key
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = server public ip:514
PersistentKeepalive = 19
```

где

* `Name: ` (1) - название клиентского виртуальный сетевой интерфейс

* `Public Key :` (2) - открытый ключ клиента, скопируйте его в буфер обмена и пропишите в конфиге интерфейса `wg0` на сервере, секция `[Peer]`:

    ```console
    adam@my-vps:~$ sudo nano /etc/wireguard/wg0.conf
    ```

    ```properties
    [Peer]
    AllowedIPs = 10.0.0.2/32
    PublicKey = client public key <- (2)
    ```

* `[Interface]` - секция настроек виртуального интерфейса

* `PrivateKey = ` (3) - закрытый, т.е. *секретный* ключ клиента

* `ListenPort = 514`

* `Address = 10.0.0.2/24`

* `DNS = 1.1.1.1, 8.8.8.8`

* `[Peer]` - секция настроек сервера, к которму будем подключаться

* `PublicKey = ` (4) - указываем публичный ключ сервера, полученный на [первом шаге](#step1)
    
    ```console
    sudo cat /etc/wireguard/publickey
    ```

* `AllowedIPs = 0.0.0.0/0, ::/0` - 

* `Endpoint = server_public_IP:514` (5) - 

* `PersistentKeepalive = 19` - 

![Настройки туннеля](02_wireguard_vpn_server_03_2.png "Настройки туннеля")

После сохранения и активации туннеля появится новый сетевой интерфейс, который исчезнет после выключения:

![Виртуальный сетевой интерфейс](02_wireguard_vpn_server_03_3.png "Виртуальный сетевой интерфейс")

## [ <kbd>↑</kbd> ](#up) <a name="step0">[Шаг 4 - Проверка работы туннеля](#step0)</a>

Мы готовы попробовать 







wg-quick up wg0
wg-quick down wg0

<!--
![alt text](image.png "description")
```console
adam@my-vps:~$ 
```
## [ <kbd>↑</kbd> ](#up) <a name="step0">[](#step0)</a>
-->
