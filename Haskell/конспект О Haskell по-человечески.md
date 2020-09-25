# Приготовимся

## stack

Установим stack [отсюда](https://docs.haskellstack.org/en/stable/install_and_upgrade/).

`stack` умеет
+ разворачивать инфраструктуру языка
+ собирать проекты
+ устанавливать библиотеки

глобальная инфраструктура последней стабильной версии в `~/.stack`.
    
    stack setup

новый проект с именем `real`.
    
    stack new real
    cd real
    ~/real$ tree
    .
    ├── ChangeLog.md
    ├── LICENSE
    ├── README.md
    ├── Setup.hs
    ├── app
    │   └── Main.hs         <- главный модуль программы
    ├── package.yaml
    ├── real.cabal          <- сборочный конфиг проекта
    ├── src
    │   └── Lib.hs          <- ещё один модуль
    ├── stack.yaml          <- конфиг stack
    ├── stack.yaml.lock
    └── test
        └── Spec.hs         <- тесты (пока не нужны)
    3 directories, 11 files

сборка

    stack clean && stack build --ghc-options="-Wall -Werror"

скомпилированный файл `real-exe` появится внутри `.stack-work/...`

    ~/real$ stack run
    someFunc

## Модули: знакомство

Проект состоит из модулей, файлов `*.hs` с исходным кодом. Один модуль = один файл. 





## 