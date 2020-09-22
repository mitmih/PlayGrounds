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
    PS C:\Users\user> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
    ```

1. Обновляем платформу до WSL 2:
    > wsl2 доступен для Windows 10 версии 1903 сборка 18362 или выше
    
    Для просмотра текущей версии ОС выполняем <kbd>Win</kbd> <kbd>R</kbd> команду `winver`. При необходимости обновляем ОС.

1. Включаем `Virtual Machine`:

    ```PowerShell
    PS C:\Users\user> Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName VirtualMachinePlatform
    ```

1. Проверяем компоненты:
    ```PowerShell
    PS C:\Users\user> Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -in @('Microsoft-Windows-Subsystem-Linux', 'VirtualMachinePlatform')}
    
    FeatureName : Microsoft-Windows-Subsystem-Linux
    State       : Enabled

    FeatureName : VirtualMachinePlatform
    State       : Enabled
    ```

1. Скачиваем и устанавливаем пакет Linux kernel update:
    ```PowerShell
    PS C:\Users\user> curl.exe -L -o $env:USERPROFILE\Downloads\wsl_update_x64.msi https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
    
    PS C:\Users\user> & $env:USERPROFILE\Downloads\wsl_update_x64.msi
    ```

1. Включаем использование 2-й версиии wsl по умолчанию:
    ```PowerShell
    PS C:\Users\user> wsl --set-default-version 2
    ```

1. Скачиваем и устанавливаем Ubuntu 20.04:
    ```PowerShell
    PS C:\Users\user> curl.exe -L -o $env:USERPROFILE\Downloads\ubuntu-20-04.appx https://aka.ms/wslubuntu2004
    PS C:\Users\user> Add-AppxPackage $env:USERPROFILE\Downloads\ubuntu-20-04.appx
    ```

    > Ссылки доступных для скачивания дистрибутивов Linux можно посмотреть на [официальной странице](https://docs.microsoft.com/en-us/windows/wsl/install-manual).

1. Запускаем, в первый раз нужно указать пользователя и задать ему пароль:
    
    > набираем `ubuntu`, нажимаем <kbd>Tab</kbd>
    
    ```PowerShell
    PS C:\Users\user> ubuntu2004.exe
    ```

1. И обновляем дистрибутив:
    ```console
    wsl2@w10m2:~$ sudo apt update && sudo apt upgrade -y
    ```


## [ <kbd>↑</kbd> ](#up) <a name="step2">[Шаг 2 - Подготовка Haskell на стороне Ubuntu](#step2)</a>
На основе [дополнительной информации по среде разработки](https://rizzoma.com/topic/c27faf1cfa188c1120f59af4c35e6099/0_b_9n8n_96jab/) из чата `Learn Haskell with FSD`.

1. Устанавливаем [stack](https://docs.haskellstack.org/en/stable/README/):
    
    ```console
    wsl2@w10m2:~$ curl -sSL https://get.haskellstack.org/ | sh
    ```

1. Обращаем внимание на предупреждение про `~/.local/bin` - заводим нужную папку:
    

    ```console
    wsl2@w10m2:~$ mkdir -p ~/.local/bin
    ```
    По умолчанию профиль пользователя `~/.profile` уже настроен на добавление к `PATH` такой папки.

1. Перезайдём в консоль, чтобы изменения вступили в силу:
    ```console
    wsl2@w10m2:~$ logout
    PS C:\Users\user> ubuntu2004.exe
    wsl2@w10m2:~$ stack path --local-bin
    /home/wsl2/.local/bin
    ```

1. Разворачиваем Haskell-инфраструктуру в своём профиле:
    
    ```console
    wsl2@w10m2:~$ wsl2@w10m2:~$ stack setup
    ```

1. Проверяем установку `stack` сборкой тестового проекта:

    ```console
    wsl2@w10m2:~$ stack new test
    wsl2@w10m2:~$ cd ./test/
    wsl2@w10m2:~/test$ stack build
    wsl2@w10m2:~/test$ stack exec test-exe
    someFunc
    wsl2@w10m2:~/test$ stack install
    wsl2@w10m2:~/test$ cd ..
    wsl2@w10m2:~$ test-exe
    someFunc
    ```

1. Смотрим версию `stack`:
    ```console
    wsl2@w10m2:~$ stack exec stack -- --version
    Version 2.3.3, Git revision cb44d51bed48b723a5deb08c3348c0b3ccfc437e x86_64 hpack-0.33.0
    ```

1. Устанавливаем [ghcup](https://www.haskell.org/ghcup/) - упрощает установку определенных версий ghc:

    ```console
    wsl2@w10m2:~$ curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
    wsl2@w10m2:~$ ghcup list
    ```

1. Ставим из исходников интерфейс для IDE [Haskell IDE Engine](https://github.com/haskell/haskell-ide-engine#installation-from-source):
    
    <!-- sudo apt install libicu-dev libncurses-dev libgmp-dev zlib1g-dev -y -->
    
    ```console
    wsl2@w10m2:~$ git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules
    wsl2@w10m2:~$ cd haskell-ide-engine
    wsl2@w10m2:~/haskell-ide-engine$ stack ./install.hs help
    wsl2@w10m2:~/haskell-ide-engine$ stack clean && stack ./install.hs hie -q
    ```

1. Для работы подсказок ставим движок поиска по документации [Hoogle](https://github.com/ndmitchell/hoogle/blob/master/docs/Install.md), к которму обращается HIE:
    ```console
    wsl2@w10m2:~$ stack install hoogle
    wsl2@w10m2:~$ echo >> ~/.ghci ':def hoogle \x -> return $ ":!hoogle " ++ x'
    wsl2@w10m2:~$ hoogle generate
    wsl2@w10m2:~$ stack haddock --hoogle
    ```

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


