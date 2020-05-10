# PlayGrounds - тренировки в настройке различных конфигураций серверов на практике

"Игровые площадки" показались мне неплохим способом освоить на практике `stunnel`, `vpn`, `ssh`, прочие интересные и полезные технологии работы с интернет-трафиком.

Практика происходила на арендованных [у хостинга vdsina.ru](https://vdsina.ru/?partner=yfr2sd6574) виртуальных серверах.

## Итак, начнём с Ubuntu 20.04

1. [Первичная настройка vps-сервера](https://github.com/mitmih/PlayGrounds/blob/master/VPS/01_ubuntu_20.04_server_-_first_steps.md)

    Подготовка основы для сервера: ➕`sudo`-пользователь, ➖`root`, ssh-ключи и служба `sshd`, `ufw`.

2. [Запуск WireGuard VPN-сервера на Ubuntu 20.04](https://github.com/mitmih/PlayGrounds/blob/master/VPS/02_wireguard_vpn_server.md)

    Рабочий способ настройки собственного wg-сервера на примере Ubuntu, а также Windows и Linux клиентов.