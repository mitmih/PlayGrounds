# [ <kbd>←</kbd> PlayGrounds ReadMe](https://github.com/mitmih/PlayGrounds/blob/master/readme.md) <a name="up">[](#up)</a>

Данный рецепт позволяет настроить окружение для разработки на Haskell по схеме Windows 10 (VS Code) + Ubuntu под WSL2 (инфраструктура и проекты).

* [готовим Ubuntu 20.04 на WSL2 консольно](#step1)

* [ставим stack](#step2)

* [расширяем VS Code](#step3)

* [HIE не заводится, что делать?](#step3err)

<!-- * [#step4](#step4) -->

<!-- * [#step5](#step5) -->


## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 1 - Подготовка Windows Subsystem Linux 2, установка Ubuntu 20.04](#step1)</a>

[Оригинал](https://docs.microsoft.com/en-us/windows/wsl/install-win10) руководства по быстрому старту WSL2 на Windows 10.

1. Запускаем терминал `PowerShell` с правами администратора:
    
    <kbd>Win</kbd>, `PowerShell`, <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>Enter</kbd> 

1. Включаем `Windows Subsystem for Linux`:

    ```PowerShell
    PS> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
    ```

1. Обновляем платформу до WSL 2:
    > wsl2 доступен для Windows 10 версии 1903 сборка 18362 или выше
    
    Для просмотра текущей версии ОС выполняем <kbd>Win</kbd> <kbd>R</kbd> команду `winver`. При необходимости обновляем ОС.

1. Включаем `Virtual Machine`:

    ```PowerShell
    PS> Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName VirtualMachinePlatform
    ```

1. Проверяем компоненты:
    ```PowerShell
    PS> Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -in @('Microsoft-Windows-Subsystem-Linux', 'VirtualMachinePlatform')}
    
    FeatureName : Microsoft-Windows-Subsystem-Linux
    State       : Enabled

    FeatureName : VirtualMachinePlatform
    State       : Enabled
    ```

1. Скачиваем и устанавливаем пакет Linux kernel update:
    ```PowerShell
    PS> curl.exe -L -o $env:USERPROFILE\Downloads\wsl_update_x64.msi https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
    
    PS> & $env:USERPROFILE\Downloads\wsl_update_x64.msi
    ```

1. Включаем использование 2-й версиии wsl по умолчанию:
    ```PowerShell
    PS> wsl --set-default-version 2
    ```

1. Скачиваем и устанавливаем Ubuntu 20.04:
    ```PowerShell
    PS> curl.exe -L -o $env:USERPROFILE\Downloads\ubuntu-20-04.appx https://aka.ms/wslubuntu2004
    PS> Add-AppxPackage $env:USERPROFILE\Downloads\ubuntu-20-04.appx
    ```

    > Ссылки доступных для скачивания дистрибутивов Linux можно посмотреть на [официальной странице](https://docs.microsoft.com/en-us/windows/wsl/install-manual).

1. Запускаем, в первый раз нужно указать пользователя и задать ему пароль:
    
    ```PowerShell
    # набираем `ubuntu`, нажимаем Tab
    PS> ubuntu2004.exe
    ```
    
    > Отменить развёртывание можно командами:
    > ```PowerShell
    > PS> wsl -l -v
    >   NAME            STATE           VERSION
    > * Ubuntu-20.04    Running         2
    > PS> wsl --terminate Ubuntu-20.04
    > PS> wsl --unregister Ubuntu-20.04
    > Отмена регистрации...
    > PS> wsl -l -v
    > Нет установленных дистрибутивов подсистемы Windows для Linux.
    > Дистрибутивы можно установить из Microsoft Store:
    > https://aka.ms/wslstore
    > ```
    > 
    > Полезные команды по управлению дистрибутивами:
    > ```PowerShell
    > # перезапуск дистрибутива
    > wsl --terminate ubuntu ; wsl --distribution ubuntu --user wsl2
    > 
    > # бэкап дистрибутива
    > wsl --export ubuntu D:\ubuntu.tar
    > 
    > # удаление дистрибутива
    > wsl --unregister ubuntu
    > 
    > # импорт дистрибутива
    > wsl --import ubuntu C:\ubuntu D:\ubuntu.tar
    > ```

1. И обновляем дистрибутив:
    ```shell
    ~$ sudo apt update && sudo apt upgrade -y
    ```
    > для листинга директорий в виде дерева пригодится утилита `tree`
    > ```shell
    > ~$ sudo apt install tree
    > ```


## [ <kbd>↑</kbd> ](#up) <a name="step2">[Шаг 2 - Подготовка Haskell в Ubuntu - установка `stack`](#step2)</a>
На основе [дополнительной информации по среде разработки](https://rizzoma.com/topic/c27faf1cfa188c1120f59af4c35e6099/0_b_9n8n_96jab/) из чата `Learn Haskell with FSD`.

1. Устанавливаем, [ghcup](https://www.haskell.org/ghcup/) - понадобится для корректной работы vscode-расширения `Integrated Haskell Shell`, также упрощает установку различный версий `ghc`:

    Сначала установим необходимые пакеты
    ```shell
    ~$ sudo apt install build-essential curl libffi-dev libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5 zlib1g-dev libicu-dev
    ```
    <!-- ~$ sudo apt install -y build-essential curl libffi-dev libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5 -->
    Потом сам `ghcup`
    ```shell
    ~$ curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
    ~$ ghcup list
    ```
    > Если `ghcup list` не срабатывает, нужно перезайти, чтобы изменения переменных окружения вступили в силу (см. шаг 2 п. 2)

1. Устанавливаем [stack](https://docs.haskellstack.org/en/stable/README/):
    
    ```shell
    ~$ curl -sSL https://get.haskellstack.org/ | sh
    ```
    > Также, автоматически, будут установлены зависимые deb-пакеты.

1. Обращаем внимание на предупреждение:
    ```
    WARNING: '/home/wsl2/.local/bin' is not on your PATH.
        For best results, please add it to the beginning of PATH in your profile.
    ```
    По умолчанию профиль пользователя `~/.profile` уже настроен на добавление такой папки к `PATH`, но её пока не существует, исправляем:
    ```shell
    ~$ mkdir -p ~/.local/bin
    ```
    
    Перезайдём в консоль, чтобы изменения вступили в силу:
    ```shell
    ~$ logout
    ```
    ```PowerShell
    PS> wsl --terminate Ubuntu-20.04 ; wsl --distribution Ubuntu-20.04
    ```

1. Разворачиваем Haskell-инфраструктуру в своём профиле:
    
    ```shell
    ~$ stack setup
    ```

1. Проверяем, что `stack` находится в переменной `PATH`:
    ```shell
    ~$ stack path --local-bin
    /home/wsl2/.local/bin
    ```

1. И установку `stack` сборкой тестового проекта:

    ```shell
    ~$ stack new test
    ~$ cd ./test/
    ~/test$ stack build
    ~/test$ stack exec test-exe
    someFunc
    ~/test$ stack install
    ~/test$ cd ..
    ~$ test-exe
    someFunc
    ```

1. Смотрим версию `stack`:
    ```shell
    ~$ stack exec stack -- --version
    Version 2.3.3, Git revision cb44d51bed48b723a5deb08c3348c0b3ccfc437e x86_64 hpack-0.33.0
    ```

## [ <kbd>↑</kbd> ](#up) <a name="step3">[Шаг 3 - Настройка работы плагинов в Ubuntu и в VS Code](#step3)</a>

> Для разработки в Ubuntu через WSL2 достаточно установленного по умолчанию расширения `Remote - WSL`
> 
> Если необходимо подключаться к контейнеру или к отдельной машине по SSH, то можно установить сразу набор из трёх расширений `Remote Development`

 Запускаем VS Code:
```shell
# новый проект ex
$ stack new ex
# ...
# открытие проекта в VS Code
$ code ~/ex
Installing VS Code Server for x64 (58bb7b2331731bf72587010e943852e13e6fd3cf)
Downloading: 100%
Unpacking: 100%
Unpacked 2341 files and folders to /home/wsl2/.vscode-server/bin/58bb7b2331731bf72587010e943852e13e6fd3cf.
```

> Все настройки сервера VS Code, в т.ч. и установленные плагины, хранятся в профиле пользователя:
> ```shell
> ~$ tree -L 2 .vscode-server/
> .vscode-server/
> ├── bin
> │   └── 58bb7b2331731bf72587010e943852e13e6fd3cf
> ├── data
> │   ├── CachedExtensionVSIXs
> │   ├── Machine
> │   ├── User
> │   ├── logs
> │   └── machineid
> └── extensions
>     ├── eriksik2.vscode-ghci-0.0.2
>     ├── haskell.haskell-1.1.0
>     ├── jcanero.hoogle-vscode-0.0.7
>     ├── justusadam.language-haskell-3.3.0
>     ├── lunaryorn.hlint-0.5.1
>     └── phoityne.phoityne-vscode-0.0.25
>```

Установка плагинов происходит по <kbd>F1</kbd> + команда `extensions: Install Extensions`.

1. Плагин [Haskell](https://marketplace.visualstudio.com/items?itemName=haskell.haskell) добавляет поддержку языка в VS Code:
    * диагностические сообщения об ошибках и предупреждениях от GHC
    * информация о типах и документация при наведении курсора
    * переход к определениям функций
    * подсветка рекомендаций
    * автодополнение кода
    * и другие возможности
    
    <details><summary>сервер Haskell Language Server</summary>
    </details>
    <details><summary>сервер Haskell IDE Engine</summary>
    Ставим из исходников интерфейс для IDE [Haskell IDE Engine](https://github.com/haskell/haskell-ide-engine#installation-from-source):
    
    ```shell
    ~$ git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules
    ~$ cd haskell-ide-engine
    ~/haskell-ide-engine$ stack ./install.hs help
    ~/haskell-ide-engine$ stack clean && stack ./install.hs latest -q
    ```
    <!-- stack clean && stack ./install.hs hie-8.8.3 -s -j1 -->
    
    <a name="step3err">[](#step3err)<details><summary>Что делать с ошибками?</summary>
     
    В процессе установки `hie` могут возникать различные ошибки:
    
    1. `ConnectionTimeout` - ошибка при скачивании пакета, например:
        ```shell
        HttpExceptionRequest Request {
        host                 = "casa.fpcomplete.com"
        port                 = 443
        secure               = True
        requestHeaders       = []
        path                 = "/v1/pull"
        queryString          = ""
        method               = "POST"
        proxy                = Nothing
        rawBody              = False
        redirectCount        = 10
        responseTimeout      = ResponseTimeoutDefault
        requestVersion       = HTTP/1.1
        }
        ConnectionTimeout
        Progress 27/56
        ```
        
        В процессе скачивания пакетов на различных этапах не удаётся скачать какой-либо пакет и сборка завершается ошибкой. Облегчает, но полностью не избавляет, либо включение VPN, либо запуск в однопоточном режиме `stack ./install.hs hie -q -j1`, либо их комбинация.
        
        **Решение**: повторный запуск сборки `stack clean && stack ./install.hs hie -q` до тех пор, пока все пакеты не будут скачаны.
    
    1. `/usr/bin/ld.gold: error: cannot find -ltinfo` - ошибка компоновщика для ELF файлов, например:
        ```shell
        ghc-lib-parser-ex  > /usr/bin/ld.gold: error: cannot find -ltinfo
        ghc-lib-parser-ex  > collect2: error: ld returned 1 exit status
        ghc-lib-parser-ex  > `gcc' failed in phase `Linker'. (Exit code: 1)
        ghc-exactprint     > /usr/bin/ld.gold: error: cannot find -ltinfo
        ghc-exactprint     > collect2: error: ld returned 1 exit status
        ghc-exactprint     > `gcc' failed in phase `Linker'. (Exit code: 1)
        ```
        Компоновщик `ld.gold` не может найти библиотеку `libtinfo.so` ни по одному из путей `ld --verbose | grep SEARCH_DIR`. Для решения нужно провести небольшое расследование.
        
        Посмотрим подробнее результат поиска при компоновке:
        ```shell
        $ ld.gold -ltinfo --verbose && rm a.out
        ld.gold: Attempt to open //lib/x86_64-linux-gnu/libtinfo.so failed
        ld.gold: Attempt to open //lib/x86_64-linux-gnu/libtinfo.a failed
        ld.gold: Attempt to open //usr/lib/x86_64-linux-gnu/libtinfo.so failed
        ld.gold: Attempt to open //usr/lib/x86_64-linux-gnu/libtinfo.a failed
        ld.gold: Attempt to open //lib/libtinfo.so failed
        ld.gold: Attempt to open //lib/libtinfo.a failed
        ld.gold: Attempt to open //usr/lib/libtinfo.so failed
        ld.gold: Attempt to open //usr/lib/libtinfo.a failed
        ld.gold: error: cannot find -ltinfo
        ld.gold: Opened new descriptor 3 for "a.out"
        ```
        
        Не хватает символической ссылки `/usr/lib/x86_64-linux-gnu/libtinfo.so`. Поищем нужные файлы:
        ```shell
        $ sudo find /lib /usr/lib -name *libtinfo.*
        /usr/lib/x86_64-linux-gnu/libtinfo.so.6.2
        /usr/lib/x86_64-linux-gnu/libtinfo.so.6
        ```
        
        Найдено два файла. Посмотрим, что это за файлы, чтобы понять, на какой из них нужно дать ссылку:
        ```shell
        $ ls -la /usr/lib/x86_64-linux-gnu/*tinfo*so*
        lrwxrwxrwx 1 root root     15 Feb 26  2020 /usr/lib/x86_64-linux-gnu/libtinfo.so.6 -> libtinfo.so.6.2
        -rw-r--r-- 1 root root 192032 Feb 26  2020 /usr/lib/x86_64-linux-gnu/libtinfo.so.6.2
        ```
        
        **Решение 1** раз оригиналом является файл `libtinfo.so.6.2`, а `libtinfo.so.6` - лишь ссылкой, сделаем ещё одну подобную ссылку на оригинал:
        ```shell
        ~$ cd /usr/lib/x86_64-linux-gnu/
        /usr/lib/x86_64-linux-gnu$ sudo ln -s /usr/lib/x86_64-linux-gnu/libtinfo.so.6.2 libtinfo.so
        /usr/lib/x86_64-linux-gnu$ ls -la *libtinfo*
        lrwxrwxrwx 1 root root     41 Sep 22 23:30 libtinfo.so -> /usr/lib/x86_64-linux-gnu/libtinfo.so.6.2
        lrwxrwxrwx 1 root root     15 Feb 26  2020 libtinfo.so.6 -> libtinfo.so.6.2
        -rw-r--r-- 1 root root 192032 Feb 26  2020 libtinfo.so.6.2
        /usr/lib/x86_64-linux-gnu$ cd ~
        ~$ ld.gold -ltinfo --verbose
        ld.gold: Opened new descriptor 3 for "//lib/x86_64-linux-gnu/libtinfo.so"
        ld.gold: Attempt to open //lib/x86_64-linux-gnu/libtinfo.so succeeded
        ld.gold: Unlocking file "//lib/x86_64-linux-gnu/libtinfo.so"
        ld.gold: Released descriptor 3 for "//lib/x86_64-linux-gnu/libtinfo.so"
        ld.gold: Opened new descriptor 4 for "a.out"
        ```
        
        **Решение 2** ~~узнаём, [в какой пакет входит библиотека](https://packages.debian.org/search?suite=buster&arch=any&searchon=contents&keywords=libtinfo.so.6) и ставим его `$ sudo apt install -y libtinfo6`~~ не работает, такой пакет уже стоит.
    </details>
    
    Смотрим версию HIE
    ```shell
    ~/haskell-ide-engine$ hie --version
    ```
    </details>
    
    Для работы подсказок ставим движок поиска по документации [Hoogle](https://github.com/ndmitchell/hoogle/blob/master/docs/Install.md), к которму обращается HIE:
    ```shell
    ~/haskell-ide-engine$ cd ~
    ~$ stack install hoogle
    ```
    <!-- ~$ echo >> ~/.ghci ':def hoogle \x -> return ~$ ":!hoogle " ++ x' -->
    > Если видим ошибки `ConnectionTimeout` то перезапускаем установку `stack install hoogle`

    Генерируем индекс
    ```shell
    ~$ hoogle generate
    ```
    <!-- ~$ stack haddock --hoogle -->

    Проверяем версию `hoogle`
    
    ```shell
    $ hoogle -V
    ```
    
    Устанавливаем плагин в VS Code, в настройках "Haskell > Language Server Variant" выбираем HIE.







1. [hlint](https://marketplace.visualstudio.com/items?itemName=lunaryorn.hlint)

1. [hoogle-vscode](https://marketplace.visualstudio.com/items?itemName=jcanero.hoogle-vscode)

1. [Integrated Haskell Shell](https://marketplace.visualstudio.com/items?itemName=eriksik2.vscode-ghci)

1. [Haskell GHCi Debug Adapter Phoityne](https://marketplace.visualstudio.com/items?itemName=phoityne.phoityne-vscode)

<!-- 1. Для плагина отладки
    ```shell
    ~$ stack install phoityne-vscode haskell-dap ghci-dap haskell-debug-adapter
    ``` -->







<!--
## [ <kbd>↑</kbd> ](#up) <a name="step3">[Шаг 3](#step3)</a>
## [ <kbd>↑</kbd> ](#up) <a name="step4">[Шаг 4](#step4)</a>
<details>
<summary>

```shell
```
</summary>

```shell
```
</details>


https://medium.com/@remisa.yousefvand/setup-haskell-development-environment-on-ubuntu-64c0f29f2b



    1. Устанавливаем Cabal и Haskell через Stack:
    
    ```shell
    ~$ stack install cabal-install
    ~$ cabal update
    ```
    
    > У меня в процессе установки выскочила ошибка из несоответствия доступной версии и указанной в конфиге stack`а:
    > ```shell
    > Error: While constructing the build plan, the following exceptions were encountered:
    > 
    > In the dependencies for cabal-install-3.2.0.0:
    >     Cabal-3.0.1.0 from stack configuration does not match ==3.2.*  (latest matching version is 3.2.0.0)
    > needed since cabal-install is a build target.
    > 
    > Some different approaches to resolving this:
    > 
    > * Set 'allow-newer: true' in /home/wsl2/.stack/config.yaml to ignore all version constraints and build anyway.
    > 
    > * Recommended action: try adding the following to your extra-deps in /home/wsl2/.stack/global-project/stack.yaml:
    > 
    > - Cabal-3.2.0.0@sha256:d0d7a1f405f25d0000f5ddef684838bc264842304fd4e7f80ca92b997b710874,27320
    > ```
    > 
    > Отркрываем конфиг:
    > ```shell
    > ~$ nano ~/.stack/global-project/stack.yaml
    > ```
    > 
    > И добавляем, как рекомендовано, соответствующую поправку
    > ```shell
    > extra-deps:
    > - Cabal-3.2.0.0@sha256:d0d7a1f405f25d0000f5ddef684838bc264842304fd4e7f80ca92b997b710874,27320
    > ```
    > после этого повторяем `stack install cabal-install`

choco list --local-only
choco uninstall haskell-dev --remove-dependencies
choco list --local-only
-->