# PlayGrounds - практические эксперименты по настройке различных конфигураций серверов

Есть много разных и полезных технологий работы с интернет-трафиком - `stunnel`, `vpn`, `ssh`, etc - которые сложно освоить, играясь с виртуальными машинами на домашнем компьютере.

Практичнее было бы понастраивать "настоящие" сервера, арендованные у реального хостинг-провайдера, например, [здесь](https://vdsina.ru/?partner=yfr2sd6574) :)

Полученные в ходе этих экспериментов опыт и знания собраны здесь и оформлены в виде рецептов.

## Итак, начнём с Ubuntu 20.04

1. [Первичная настройка vps-сервера](https://github.com/mitmih/PlayGrounds/blob/master/VPS/01_ubuntu_20.04_server_-_first_steps.md)

    Подготовка основы для будущего сервера: +`sudo`-пользователь, –`root`-доступ, ssh-ключи и `sshd` сервис, `ufw`.

1. [Запуск WireGuard vpn-сервиса](https://github.com/mitmih/PlayGrounds/blob/master/VPS/02_wireguard_vpn_server.md)

    Настройка собственного wg-сервера, Windows / Linux / Android клиентов.

    Решение проблемы задержки, до нескольких минут, восстановления соединения после перезагрузки сервера.

1. [Запуск Outline Shadowsocks сервиса](https://github.com/mitmih/PlayGrounds/blob/master/VPS/03_outline_shadowsocks_server.md)

    Установка Outline-сервиса на виртуальный сервер.

    Настройка через Outline-manager.

    Подключение Outline-клиента.

1. [Подготовка окружения для обучения разработке на Haskell](https://github.com/mitmih/PlayGrounds/blob/master/Haskell/HowTo_wsl2.md)
