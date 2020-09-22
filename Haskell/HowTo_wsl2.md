# [ <kbd>←</kbd> PlayGrounds ReadMe](https://github.com/mitmih/PlayGrounds/blob/master/readme.md) <a name="up">[](#up)</a>

Данный рецепт позволяет настроить окружение для разработки на Haskell по схеме Windows 10 (VS Code) + Ubuntu под WSL2 (инфраструктура и проекты).

* [готовим Ubuntu 20.04 на WSL2 консольно](#step1)

* [ставим stack](#step2)

* [заводим Haskell IDE Engine (HIE)](#step3)

* [#step3](#step3)

* [#step4](#step4)


## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 1 - Подготовка Windows Subsystem Linux 2, установка Ubuntu 20.04](#step1)</a>

[Оригинал](https://docs.microsoft.com/en-us/windows/wsl/install-win10) руководства по быстрому старту WSL2 на Windows 10.

1. Запускаем терминал `PowerShell` с правами администратора:
    
    <kbd>Win</kbd>, `PowerShell`, <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>Enter</kbd> 

1. Включаем `Windows Subsystem for Linux`:

    ```PowerShell
    PS > Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
    ```

1. Обновляем платформу до WSL 2:
    > wsl2 доступен для Windows 10 версии 1903 сборка 18362 или выше
    
    Для просмотра текущей версии ОС выполняем <kbd>Win</kbd> <kbd>R</kbd> команду `winver`. При необходимости обновляем ОС.

1. Включаем `Virtual Machine`:

    ```PowerShell
    PS > Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName VirtualMachinePlatform
    ```

1. Проверяем компоненты:
    ```PowerShell
    PS > Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -in @('Microsoft-Windows-Subsystem-Linux', 'VirtualMachinePlatform')}
    
    FeatureName : Microsoft-Windows-Subsystem-Linux
    State       : Enabled

    FeatureName : VirtualMachinePlatform
    State       : Enabled
    ```

1. Скачиваем и устанавливаем пакет Linux kernel update:
    ```PowerShell
    PS > curl.exe -L -o $env:USERPROFILE\Downloads\wsl_update_x64.msi https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
    
    PS > & $env:USERPROFILE\Downloads\wsl_update_x64.msi
    ```

1. Включаем использование 2-й версиии wsl по умолчанию:
    ```PowerShell
    PS > wsl --set-default-version 2
    ```

1. Скачиваем и устанавливаем Ubuntu 20.04:
    ```PowerShell
    PS > curl.exe -L -o $env:USERPROFILE\Downloads\ubuntu-20-04.appx https://aka.ms/wslubuntu2004
    PS > Add-AppxPackage $env:USERPROFILE\Downloads\ubuntu-20-04.appx
    ```

    > Ссылки доступных для скачивания дистрибутивов Linux можно посмотреть на [официальной странице](https://docs.microsoft.com/en-us/windows/wsl/install-manual).

1. Запускаем, в первый раз нужно указать пользователя и задать ему пароль:
    
    > набираем `ubuntu`, нажимаем <kbd>Tab</kbd>
    > ```PowerShell
    > PS > ubuntu2004.exe
    > ```
    
    > Отменить развёртывание можно командами:
    > ```PowerShell
    > PS > wsl -l -v
    > NAME            STATE           VERSION
    > * Ubuntu-20.04    Running         2
    > PS > wsl -t Ubuntu-20.04
    > PS > wsl -l -v
    > NAME            STATE           VERSION
    > * Ubuntu-20.04    Stopped         2
    > PS > Get-AppxPackage *ubuntu* | Remove-AppxPackage
    > ```

1. И обновляем дистрибутив:
    ```console
    ~$ sudo apt update && sudo apt upgrade -y
    ```


## [ <kbd>↑</kbd> ](#up) <a name="step2">[Шаг 2 - Подготовка Haskell на стороне Ubuntu](#step2)</a>
На основе [дополнительной информации по среде разработки](https://rizzoma.com/topic/c27faf1cfa188c1120f59af4c35e6099/0_b_9n8n_96jab/) из чата `Learn Haskell with FSD`.

1. Устанавливаем [stack](https://docs.haskellstack.org/en/stable/README/):
    
    ```console
    ~$ curl -sSL https://get.haskellstack.org/ | sh
    ```

1. Обращаем внимание на предупреждение про `~/.local/bin` - заводим нужную папку:
    

    ```console
    ~$ mkdir -p ~/.local/bin
    ```
    По умолчанию профиль пользователя `~/.profile` уже настроен на добавление к `PATH` такой папки.

1. Перезайдём в консоль, чтобы изменения вступили в силу:
    ```console
    ~$ logout
    PS > ubuntu2004.exe
    ~$ stack path --local-bin
    /home/wsl2/.local/bin
    ```

1. Разворачиваем Haskell-инфраструктуру в своём профиле:
    
    ```console
    ~$ stack setup
    ```

1. Проверяем установку `stack` сборкой тестового проекта:

    ```console
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
    ```console
    ~$ stack exec stack -- --version
    Version 2.3.3, Git revision cb44d51bed48b723a5deb08c3348c0b3ccfc437e x86_64 hpack-0.33.0
    ```

1. Устанавливаем Cabal и Haskell через Stack:
    
    ```console
    ~$ stack install cabal-install
    ~$ cabal update
    ```
    
    > У меня в процессе установки выскочила ошибка из несоответствия доступной версии и указанной в конфиге stack`а:
    > ```console
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
    > ```console
    > ~$ nano ~/.stack/global-project/stack.yaml
    > ```
    > 
    > И добавляем, как рекомендовано, соответствующую поправку
    > ```console
    > extra-deps:
    > - Cabal-3.2.0.0@sha256:d0d7a1f405f25d0000f5ddef684838bc264842304fd4e7f80ca92b997b710874,27320
    > ```
    > после этого повторяем `stack install cabal-install`

1.
    ```
    ~$ stack install shake
    ~$ stack exec -- shake --demo
    ~$ sudo apt install -y libicu-dev libncurses-dev libgmp-dev zlib1g-dev
    ```

1. Ставим из исходников интерфейс для IDE [Haskell IDE Engine](https://github.com/haskell/haskell-ide-engine#installation-from-source):
    
    <!-- sudo apt install libicu-dev libncurses-dev libgmp-dev zlib1g-dev -y -->
    
    <!-- ~$ git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules -->
    ```console
    ~$ git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules
    ~$ cd haskell-ide-engine
    ~/haskell-ide-engine$ stack ./install.hs help
    ~/haskell-ide-engine$ stack clean && stack ./install.hs hie -s
    ```

1. Для работы подсказок ставим движок поиска по документации [Hoogle](https://github.com/ndmitchell/hoogle/blob/master/docs/Install.md), к которму обращается HIE:
    ```console
    ~$ stack install hoogle
    ~$ echo >> ~/.ghci ':def hoogle \x -> return ~$ ":!hoogle " ++ x'
    ~$ hoogle generate
    ~$ stack haddock --hoogle
    ```

<!--
## [ <kbd>↑</kbd> ](#up) <a name="step3">[Шаг 3](#step3)</a>
## [ <kbd>↑</kbd> ](#up) <a name="step4">[Шаг 4](#step4)</a>
<details>
<summary>

```console
```
</summary>

```console
```
</details>

ghcup System requirements
Install the following distro packages: build-essential curl libffi-dev libffi6 libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5
sudo apt install build-essential curl libffi-dev libffi6 libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5

https://medium.com/@remisa.yousefvand/setup-haskell-development-environment-on-ubuntu-64c0f29f2b

git clone --recursive https://github.com/haskell/haskell-ide-engine

1. Устанавливаем, по желанию, [ghcup](https://www.haskell.org/ghcup/) - упрощает установку различный версий ghc:

    ```console
    ~$ curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
    ~$ ghcup list
    ```
-->