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

* [пробуем туннель "на вкус"](#step4)

* [step5](#step5)

<!--
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

* `Address = 10.0.0.1/24` - IP-адрес сервера в нашей vpn-сети, можно указать несколько адресов через запятую или в новых строчках

* `ListenPort = 514` - UDP-порт, по которому интерфейс сервера со своей стороны устанавливает соединения с клиентами

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

Добавим новый туннель <kbd>ctrl</kbd> <kbd>N</kbd>:

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

* `ListenPort = 514` - UDP-порт сетевого интерфейса клиента, по которому он со своей стороны устанавливает соединение с сервером, может отличаться от сервера

* `Address = 10.0.0.2/24` - IP-адрес клиента в нашей vpn-сети

* `DNS = 1.1.1.1, 8.8.8.8` - IP-адреса DNS-серверов

* `[Peer]` - секция настроек сервера, с которым строим туннель

* `PublicKey = ` (4) - указываем публичный ключ сервера, полученный на [первом шаге](#step1)
    
    ```console
    sudo cat /etc/wireguard/publickey
    ```

* `AllowedIPs = 0.0.0.0/0, ::/0` - весь трафик клиента пойдет через туннель

* `Endpoint = server_public_IP:514` (5) - "белый" IP адрес и UDP-порт для соединения с WireGuard-сервером, например `1.2.3.4:514`

* `PersistentKeepalive = 19` - интервал в секундах, для поддержания туннеля отправкой ping'-а

![Настройки туннеля](02_wireguard_vpn_server_03_2.png "Настройки туннеля")

После сохранения и активации туннеля появится новый сетевой интерфейс, который исчезнет после выключения:

![Виртуальный сетевой интерфейс](02_wireguard_vpn_server_03_3.png "Виртуальный сетевой интерфейс")

## [ <kbd>↑</kbd> ](#up) <a name="step4">[Шаг 4 - Пробный запуск туннеля](#step4)</a>

Итак, все приготовления выполнены, пробуем соединиться! Включаем интерфейс `wg0` на сервере:

```console
adam@my-vps:~$ sudo wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 10.0.0.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens3 -j MASQUERADE
```

Проверим открытые порты:

```console
adam@my-vps:~$ sudo ss -nltup
Netid   State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port   Process
udp     UNCONN   0        0                0.0.0.0:514           0.0.0.0:*
udp     UNCONN   0        0          127.0.0.53%lo:53            0.0.0.0:*       users:(("systemd-resolve",pid=392,fd=12))
udp     UNCONN   0        0                   [::]:514              [::]:*
tcp     LISTEN   0        4096       127.0.0.53%lo:53            0.0.0.0:*       users:(("systemd-resolve",pid=392,fd=13))
tcp     LISTEN   0        128              0.0.0.0:22            0.0.0.0:*       users:(("sshd",pid=1237,fd=3))
tcp     LISTEN   0        128                 [::]:22               [::]:*       users:(("sshd",pid=1237,fd=4))
```

Активируем интерфейс клиента и проверим доступность vpn-IP сервера:

```cmd
ping -t 10.0.0.1

Обмен пакетами с 10.0.0.1 по с 32 байтами данных:
Ответ от 10.0.0.1: число байт=32 время=69мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=69мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=69мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=69мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=68мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=68мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=69мс TTL=64
Ответ от 10.0.0.1: число байт=32 время=68мс TTL=64

Статистика Ping для 10.0.0.1:
    Пакетов: отправлено = 8, получено = 8, потеряно = 0
    (0% потерь)
Приблизительное время приема-передачи в мс:
    Минимальное = 68мсек, Максимальное = 69 мсек, Среднее = 68 мсек
Control-C
^C
```

![Проверка работы туннеля со стороны клиента](02_wireguard_vpn_server_04_1.png "Проверка работы туннеля со стороны клиента")

Чтобы аналогично проверить доступность клиента с сервера, предварительно разрешите в брандмауэре клиента icmp-запросы или отключите его на время проверки:

```console
adam@my-vps:~$ ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=128 time=68.7 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=128 time=69.1 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=128 time=69.0 ms
64 bytes from 10.0.0.2: icmp_seq=4 ttl=128 time=69.1 ms
^C
--- 10.0.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3001ms
rtt min/avg/max/mdev = 68.743/68.984/69.120/0.143 ms
```

Также убедимся, что наш трафик идёт через туннель, выполнив трассировку:

```cmd
TRACERT.EXE -w 500 google.com

Трассировка маршрута к google.com [216.58.211.110]
с максимальным числом прыжков 30:

  1    69 ms     *       68 ms  10.0.0.1
  2    70 ms    70 ms    70 ms  my-host.hosted-by-vdsina.ru [x.x.x.x]
  3    69 ms     *       69 ms  185.8.177.112
  4    70 ms    79 ms    70 ms  185.8.179.33
  5    71 ms     *       73 ms  74.125.119.118
  6    72 ms    73 ms    72 ms  108.170.241.161
  7    71 ms    71 ms     *     108.170.237.45
  8    71 ms    71 ms    71 ms  ams15s32-in-f14.1e100.net [216.58.211.110]

Трассировка завершена.
```
Поздравляем, туннель приготовлен как следует, выключаем интерфейсы с обеих сторон и переходим к следующему шагу - Wireguard как сервис:

```console
adam@my-vps:~$ sudo wg-quick down wg0
[#] wg showconf wg0
[#] ip link delete dev wg0
[#] iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens3 -j MASQUERADE
```

## [ <kbd>↑</kbd> ](#up) <a name="step5">[Шаг 5 - Включаем WireGuard в качестве сервиса](#step5)</a>



<!--
![alt text](image.png "description")
```console
adam@my-vps:~$ 
```
## [ <kbd>↑</kbd> ](#up) <a name="step0">[](#step0)</a>
-->