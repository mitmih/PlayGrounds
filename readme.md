# PlayGrounds - практические эксперименты по настройке различных конфигураций серверов

Есть много разных и полезных технологий работы с интернет-трафиком - `stunnel`, `vpn`, `ssh`, etc - которые сложно освоить, играясь с виртуальными машинами на домашнем компьютере.

Практичнее было бы по-настраивать "настоящие" сервера, арендованные у реального хостинг-провайдера, [вот например тут :-)](https://vdsina.ru/?partner=yfr2sd6574) .

Собственно говоря, здесь приведены, в виде рецептов, знания, добытые в ходе подобных экспериментов.

## Итак, начнём с Ubuntu 20.04

1. [Первичная настройка vps-сервера](https://github.com/mitmih/PlayGrounds/blob/master/VPS/01_ubuntu_20.04_server_-_first_steps.md)

    Подготовка основы для сервера: +`sudo`-пользователь, –`root`, ssh-ключи и служба `sshd`, `ufw`.

2. [Запуск WireGuard VPN-сервера на Ubuntu 20.04](https://github.com/mitmih/PlayGrounds/blob/master/VPS/02_wireguard_vpn_server.md)

    Описана настройка собственного wg-сервера, Windows и Linux клиентов.

    Есть решение проблемы задержки (до нескольких минут) восстановления соединения после перезагрузки сервера.
