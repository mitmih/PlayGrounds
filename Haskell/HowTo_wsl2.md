# [ <kbd>←</kbd> PlayGrounds ReadMe](https://github.com/mitmih/PlayGrounds/blob/master/readme.md) <a name="up">[](#up)</a>

Данный рецепт позволяет настроить окружение для разработки на Haskell по схеме Windows 10 (VS Code) + Ubuntu под WSL2 (инфраструктура и проекты).

* [готовим Ubuntu 20.04 на WSL2 консольно](#step1)

* [ставим stack](#step2)

* [заводим Haskell IDE Engine (HIE)](#step3)

* [#step4](#step4)

* [#step5](#step5)


## [ <kbd>↑</kbd> ](#up) <a name="step1">[Шаг 1 - Подготовка Windows Subsystem Linux 2, установка Ubuntu 20.04](#step1)</a>

[Оригинал](https://docs.microsoft.com/en-us/windows/wsl/install-win10) руководства по быстрому старту WSL2 на Windows 10.

1. Запускаем терминал `PowerShell` с правами администратора
    
    <kbd>Win</kbd>, `PowerShell`, <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>Enter</kbd> 

1. Включаем `Windows Subsystem for Linux`

    ```PowerShell
    PS C:\Users\user> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
    ```

1. Обновляем платформу до WSL 2
    > wsl2 доступен для Windows 10 версии 1903 сборка 18362 или выше
    
    Для просмотра текущей версии ОС выполняем <kbd>Win</kbd> <kbd>R</kbd> команду `winver`. При необходимости обновляем ОС.

1. Включаем `Virtual Machine`

    ```PowerShell
    PS C:\Users\user> Enable-WindowsOptionalFeature -Online -NoRestart -FeatureName VirtualMachinePlatform
    ```

1. Проверяем компоненты
    ```PowerShell
    PS C:\Users\user> Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -in @('Microsoft-Windows-Subsystem-Linux', 'VirtualMachinePlatform')}
    
    FeatureName : Microsoft-Windows-Subsystem-Linux
    State       : Enabled

    FeatureName : VirtualMachinePlatform
    State       : Enabled
    ```

1. Скачиваем и устанавливаем пакет Linux kernel update
    ```PowerShell
    PS C:\Users\user> curl.exe -L -o $env:USERPROFILE\Downloads\wsl_update_x64.msi https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
    
    PS C:\Users\user> & $env:USERPROFILE\Downloads\wsl_update_x64.msi
    ```

1. Включаем использование 2-й версиии wsl по умолчанию
    ```PowerShell
    PS C:\Users\user> wsl --set-default-version 2
    ```

1. Скачиваем и устанавливаем Ubuntu 20.04
    ```PowerShell
    PS C:\Users\user> curl.exe -L -o $env:USERPROFILE\Downloads\ubuntu-20-04.appx https://aka.ms/wslubuntu2004
    PS C:\Users\user> Add-AppxPackage $env:USERPROFILE\Downloads\ubuntu-20-04.appx
    ```

    > Ссылки доступных для скачивания дистрибутивов Linux можно посмотреть на [официальной странице](https://docs.microsoft.com/en-us/windows/wsl/install-manual).

1. Запускаем, в первый раз нужно указать пользователя и задать ему пароль
    
    > набираем `ubuntu`, нажимаем <kbd>Tab</kbd>
    <details>
    <summary>
    
    ```PowerShell
    PS C:\Users\user> ubuntu2004.exe
    ```
    </summary>

    ```PowerShell
    Installing, this may take a few minutes...
    Please create a default UNIX user account. The username does not need to match your Windows username.
    For more information visit: https://aka.ms/wslusers
    Enter new UNIX username: wsl2
    New password:
    Retype new password:
    passwd: password updated successfully
    Installation successful!
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.

    Welcome to Ubuntu 20.04 LTS (GNU/Linux 4.19.128-microsoft-standard x86_64)

    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage

    System information as of Mon Sep 21 23:06:18 MSK 2020

    System load:  0.06               Processes:             8
    Usage of /:   0.4% of 250.98GB   Users logged in:       0
    Memory usage: 0%                 IPv4 address for eth0: 172.31.41.186
    Swap usage:   0%

    0 updates can be installed immediately.
    0 of these updates are security updates.


    The list of available updates is more than a week old.
    To check for new updates run: sudo apt update


    This message is shown once once a day. To disable it please create the
    /home/wsl2/.hushlogin file.
    ```
    </details>

1. И обновляем дистрибутив
    ```console
    wsl2@w10m2:~$ sudo apt update && sudo apt upgrade -y
    ```


## [ <kbd>↑</kbd> ](#up) <a name="step2">[Шаг 2 - установка / настройка / проверка `The Haskell Tool Stack`](#step2)</a>
На основе [дополнительной информации по среде разработки](https://rizzoma.com/topic/c27faf1cfa188c1120f59af4c35e6099/0_b_9n8n_96jab/) из чата `Learn Haskell with FSD`.

1. Устанавливаем [stack](https://docs.haskellstack.org/en/stable/README/)


    <details>
    <summary>
    
    ```console
    wsl2@w10m2:~$ sudo curl -sSL https://get.haskellstack.org/ | sh
    ```
    </summary>

    ```console
    Detected Linux distribution: ubuntu

    Installing dependencies...


    About to use 'sudo' to run the following command as root:
        apt-get install -y g++ gcc libc6-dev libffi-dev libgmp-dev make zlib1g-dev
    in order to install required system dependencies.

    [sudo] password for wsl2:
    Reading package lists... Done
    Building dependency tree
    Reading state information... Done
    The following additional packages will be installed:
    binutils binutils-common binutils-x86-64-linux-gnu cpp cpp-9 g++-9 gcc-9 gcc-9-base libasan5 libatomic1 libbinutils
    libc-dev-bin libcc1-0 libcrypt-dev libctf-nobfd0 libctf0 libgcc-9-dev libgmpxx4ldbl libgomp1 libisl22 libitm1
    liblsan0 libmpc3 libquadmath0 libstdc++-9-dev libtsan0 libubsan1 linux-libc-dev manpages-dev
    Suggested packages:
    binutils-doc cpp-doc gcc-9-locales g++-multilib g++-9-multilib gcc-9-doc gcc-multilib autoconf automake libtool flex
    bison gdb gcc-doc gcc-9-multilib glibc-doc gmp-doc libgmp10-doc libmpfr-dev libstdc++-9-doc make-doc
    The following NEW packages will be installed:
    binutils binutils-common binutils-x86-64-linux-gnu cpp cpp-9 g++ g++-9 gcc gcc-9 gcc-9-base libasan5 libatomic1
    libbinutils libc-dev-bin libc6-dev libcc1-0 libcrypt-dev libctf-nobfd0 libctf0 libffi-dev libgcc-9-dev libgmp-dev
    libgmpxx4ldbl libgomp1 libisl22 libitm1 liblsan0 libmpc3 libquadmath0 libstdc++-9-dev libtsan0 libubsan1
    linux-libc-dev make manpages-dev zlib1g-dev
    0 upgraded, 36 newly installed, 0 to remove and 0 not upgraded.
    Need to get 39.4 MB of archives.
    After this operation, 172 MB of additional disk space will be used.
    Get:1 http://archive.ubuntu.com/ubuntu focal/main amd64 binutils-common amd64 2.34-6ubuntu1 [207 kB]
    Get:2 http://archive.ubuntu.com/ubuntu focal/main amd64 libbinutils amd64 2.34-6ubuntu1 [474 kB]
    Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 libctf-nobfd0 amd64 2.34-6ubuntu1 [47.0 kB]
    Get:4 http://archive.ubuntu.com/ubuntu focal/main amd64 libctf0 amd64 2.34-6ubuntu1 [46.6 kB]
    Get:5 http://archive.ubuntu.com/ubuntu focal/main amd64 binutils-x86-64-linux-gnu amd64 2.34-6ubuntu1 [1614 kB]
    Get:6 http://archive.ubuntu.com/ubuntu focal/main amd64 binutils amd64 2.34-6ubuntu1 [3376 B]
    Get:7 http://archive.ubuntu.com/ubuntu focal/main amd64 gcc-9-base amd64 9.3.0-10ubuntu2 [19.3 kB]
    Get:8 http://archive.ubuntu.com/ubuntu focal/main amd64 libisl22 amd64 0.22.1-1 [592 kB]
    Get:9 http://archive.ubuntu.com/ubuntu focal/main amd64 libmpc3 amd64 1.1.0-1 [40.8 kB]
    Get:10 http://archive.ubuntu.com/ubuntu focal/main amd64 cpp-9 amd64 9.3.0-10ubuntu2 [7491 kB]
    Get:11 http://archive.ubuntu.com/ubuntu focal/main amd64 cpp amd64 4:9.3.0-1ubuntu2 [27.6 kB]
    Get:12 http://archive.ubuntu.com/ubuntu focal/main amd64 libcc1-0 amd64 10-20200411-0ubuntu1 [41.1 kB]
    Get:13 http://archive.ubuntu.com/ubuntu focal/main amd64 libgomp1 amd64 10-20200411-0ubuntu1 [101 kB]
    Get:14 http://archive.ubuntu.com/ubuntu focal/main amd64 libitm1 amd64 10-20200411-0ubuntu1 [26.3 kB]
    Get:15 http://archive.ubuntu.com/ubuntu focal/main amd64 libatomic1 amd64 10-20200411-0ubuntu1 [9284 B]
    Get:16 http://archive.ubuntu.com/ubuntu focal/main amd64 libasan5 amd64 9.3.0-10ubuntu2 [395 kB]
    Get:17 http://archive.ubuntu.com/ubuntu focal/main amd64 liblsan0 amd64 10-20200411-0ubuntu1 [144 kB]
    Get:18 http://archive.ubuntu.com/ubuntu focal/main amd64 libtsan0 amd64 10-20200411-0ubuntu1 [319 kB]
    Get:19 http://archive.ubuntu.com/ubuntu focal/main amd64 libubsan1 amd64 10-20200411-0ubuntu1 [136 kB]
    Get:20 http://archive.ubuntu.com/ubuntu focal/main amd64 libquadmath0 amd64 10-20200411-0ubuntu1 [146 kB]
    Get:21 http://archive.ubuntu.com/ubuntu focal/main amd64 libgcc-9-dev amd64 9.3.0-10ubuntu2 [2359 kB]
    Get:22 http://archive.ubuntu.com/ubuntu focal/main amd64 gcc-9 amd64 9.3.0-10ubuntu2 [8234 kB]
    Get:23 http://archive.ubuntu.com/ubuntu focal/main amd64 gcc amd64 4:9.3.0-1ubuntu2 [5208 B]
    Get:24 http://archive.ubuntu.com/ubuntu focal/main amd64 libc-dev-bin amd64 2.31-0ubuntu9 [71.8 kB]
    Get:25 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 linux-libc-dev amd64 5.4.0-48.52 [1138 kB]
    Get:26 http://archive.ubuntu.com/ubuntu focal/main amd64 libcrypt-dev amd64 1:4.4.10-10ubuntu4 [104 kB]
    Get:27 http://archive.ubuntu.com/ubuntu focal/main amd64 libc6-dev amd64 2.31-0ubuntu9 [2520 kB]
    Get:28 http://archive.ubuntu.com/ubuntu focal/main amd64 libstdc++-9-dev amd64 9.3.0-10ubuntu2 [1711 kB]
    Get:29 http://archive.ubuntu.com/ubuntu focal/main amd64 g++-9 amd64 9.3.0-10ubuntu2 [8404 kB]
    Get:30 http://archive.ubuntu.com/ubuntu focal/main amd64 g++ amd64 4:9.3.0-1ubuntu2 [1604 B]
    Get:31 http://archive.ubuntu.com/ubuntu focal/main amd64 libgmpxx4ldbl amd64 2:6.2.0+dfsg-4 [9128 B]
    Get:32 http://archive.ubuntu.com/ubuntu focal/main amd64 libgmp-dev amd64 2:6.2.0+dfsg-4 [320 kB]
    Get:33 http://archive.ubuntu.com/ubuntu focal/main amd64 make amd64 4.2.1-1.2 [162 kB]
    Get:34 http://archive.ubuntu.com/ubuntu focal/main amd64 manpages-dev all 5.05-1 [2266 kB]
    Get:35 http://archive.ubuntu.com/ubuntu focal/main amd64 zlib1g-dev amd64 1:1.2.11.dfsg-2ubuntu1 [156 kB]
    Get:36 http://archive.ubuntu.com/ubuntu focal/main amd64 libffi-dev amd64 3.3-4 [57.0 kB]
    Fetched 39.4 MB in 4s (9530 kB/s)
    Extracting templates from packages: 100%
    Selecting previously unselected package binutils-common:amd64.
    (Reading database ... 31933 files and directories currently installed.)
    Preparing to unpack .../00-binutils-common_2.34-6ubuntu1_amd64.deb ...
    Unpacking binutils-common:amd64 (2.34-6ubuntu1) ...
    Selecting previously unselected package libbinutils:amd64.
    Preparing to unpack .../01-libbinutils_2.34-6ubuntu1_amd64.deb ...
    Unpacking libbinutils:amd64 (2.34-6ubuntu1) ...
    Selecting previously unselected package libctf-nobfd0:amd64.
    Preparing to unpack .../02-libctf-nobfd0_2.34-6ubuntu1_amd64.deb ...
    Unpacking libctf-nobfd0:amd64 (2.34-6ubuntu1) ...
    Selecting previously unselected package libctf0:amd64.
    Preparing to unpack .../03-libctf0_2.34-6ubuntu1_amd64.deb ...
    Unpacking libctf0:amd64 (2.34-6ubuntu1) ...
    Selecting previously unselected package binutils-x86-64-linux-gnu.
    Preparing to unpack .../04-binutils-x86-64-linux-gnu_2.34-6ubuntu1_amd64.deb ...
    Unpacking binutils-x86-64-linux-gnu (2.34-6ubuntu1) ...
    Selecting previously unselected package binutils.
    Preparing to unpack .../05-binutils_2.34-6ubuntu1_amd64.deb ...
    Unpacking binutils (2.34-6ubuntu1) ...
    Selecting previously unselected package gcc-9-base:amd64.
    Preparing to unpack .../06-gcc-9-base_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking gcc-9-base:amd64 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package libisl22:amd64.
    Preparing to unpack .../07-libisl22_0.22.1-1_amd64.deb ...
    Unpacking libisl22:amd64 (0.22.1-1) ...
    Selecting previously unselected package libmpc3:amd64.
    Preparing to unpack .../08-libmpc3_1.1.0-1_amd64.deb ...
    Unpacking libmpc3:amd64 (1.1.0-1) ...
    Selecting previously unselected package cpp-9.
    Preparing to unpack .../09-cpp-9_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking cpp-9 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package cpp.
    Preparing to unpack .../10-cpp_4%3a9.3.0-1ubuntu2_amd64.deb ...
    Unpacking cpp (4:9.3.0-1ubuntu2) ...
    Selecting previously unselected package libcc1-0:amd64.
    Preparing to unpack .../11-libcc1-0_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libcc1-0:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libgomp1:amd64.
    Preparing to unpack .../12-libgomp1_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libgomp1:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libitm1:amd64.
    Preparing to unpack .../13-libitm1_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libitm1:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libatomic1:amd64.
    Preparing to unpack .../14-libatomic1_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libatomic1:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libasan5:amd64.
    Preparing to unpack .../15-libasan5_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking libasan5:amd64 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package liblsan0:amd64.
    Preparing to unpack .../16-liblsan0_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking liblsan0:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libtsan0:amd64.
    Preparing to unpack .../17-libtsan0_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libtsan0:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libubsan1:amd64.
    Preparing to unpack .../18-libubsan1_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libubsan1:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libquadmath0:amd64.
    Preparing to unpack .../19-libquadmath0_10-20200411-0ubuntu1_amd64.deb ...
    Unpacking libquadmath0:amd64 (10-20200411-0ubuntu1) ...
    Selecting previously unselected package libgcc-9-dev:amd64.
    Preparing to unpack .../20-libgcc-9-dev_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking libgcc-9-dev:amd64 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package gcc-9.
    Preparing to unpack .../21-gcc-9_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking gcc-9 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package gcc.
    Preparing to unpack .../22-gcc_4%3a9.3.0-1ubuntu2_amd64.deb ...
    Unpacking gcc (4:9.3.0-1ubuntu2) ...
    Selecting previously unselected package libc-dev-bin.
    Preparing to unpack .../23-libc-dev-bin_2.31-0ubuntu9_amd64.deb ...
    Unpacking libc-dev-bin (2.31-0ubuntu9) ...
    Selecting previously unselected package linux-libc-dev:amd64.
    Preparing to unpack .../24-linux-libc-dev_5.4.0-48.52_amd64.deb ...
    Unpacking linux-libc-dev:amd64 (5.4.0-48.52) ...
    Selecting previously unselected package libcrypt-dev:amd64.
    Preparing to unpack .../25-libcrypt-dev_1%3a4.4.10-10ubuntu4_amd64.deb ...
    Unpacking libcrypt-dev:amd64 (1:4.4.10-10ubuntu4) ...
    Selecting previously unselected package libc6-dev:amd64.
    Preparing to unpack .../26-libc6-dev_2.31-0ubuntu9_amd64.deb ...
    Unpacking libc6-dev:amd64 (2.31-0ubuntu9) ...
    Selecting previously unselected package libstdc++-9-dev:amd64.
    Preparing to unpack .../27-libstdc++-9-dev_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking libstdc++-9-dev:amd64 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package g++-9.
    Preparing to unpack .../28-g++-9_9.3.0-10ubuntu2_amd64.deb ...
    Unpacking g++-9 (9.3.0-10ubuntu2) ...
    Selecting previously unselected package g++.
    Preparing to unpack .../29-g++_4%3a9.3.0-1ubuntu2_amd64.deb ...
    Unpacking g++ (4:9.3.0-1ubuntu2) ...
    Selecting previously unselected package libgmpxx4ldbl:amd64.
    Preparing to unpack .../30-libgmpxx4ldbl_2%3a6.2.0+dfsg-4_amd64.deb ...
    Unpacking libgmpxx4ldbl:amd64 (2:6.2.0+dfsg-4) ...
    Selecting previously unselected package libgmp-dev:amd64.
    Preparing to unpack .../31-libgmp-dev_2%3a6.2.0+dfsg-4_amd64.deb ...
    Unpacking libgmp-dev:amd64 (2:6.2.0+dfsg-4) ...
    Selecting previously unselected package make.
    Preparing to unpack .../32-make_4.2.1-1.2_amd64.deb ...
    Unpacking make (4.2.1-1.2) ...
    Selecting previously unselected package manpages-dev.
    Preparing to unpack .../33-manpages-dev_5.05-1_all.deb ...
    Unpacking manpages-dev (5.05-1) ...
    Selecting previously unselected package zlib1g-dev:amd64.
    Preparing to unpack .../34-zlib1g-dev_1%3a1.2.11.dfsg-2ubuntu1_amd64.deb ...
    Unpacking zlib1g-dev:amd64 (1:1.2.11.dfsg-2ubuntu1) ...
    Selecting previously unselected package libffi-dev:amd64.
    Preparing to unpack .../35-libffi-dev_3.3-4_amd64.deb ...
    Unpacking libffi-dev:amd64 (3.3-4) ...
    Setting up manpages-dev (5.05-1) ...
    Setting up binutils-common:amd64 (2.34-6ubuntu1) ...
    Setting up linux-libc-dev:amd64 (5.4.0-48.52) ...
    Setting up libctf-nobfd0:amd64 (2.34-6ubuntu1) ...
    Setting up libgomp1:amd64 (10-20200411-0ubuntu1) ...
    Setting up libffi-dev:amd64 (3.3-4) ...
    Setting up libgmpxx4ldbl:amd64 (2:6.2.0+dfsg-4) ...
    Setting up make (4.2.1-1.2) ...
    Setting up libquadmath0:amd64 (10-20200411-0ubuntu1) ...
    Setting up libmpc3:amd64 (1.1.0-1) ...
    Setting up libatomic1:amd64 (10-20200411-0ubuntu1) ...
    Setting up libubsan1:amd64 (10-20200411-0ubuntu1) ...
    Setting up libcrypt-dev:amd64 (1:4.4.10-10ubuntu4) ...
    Setting up libisl22:amd64 (0.22.1-1) ...
    Setting up libbinutils:amd64 (2.34-6ubuntu1) ...
    Setting up libc-dev-bin (2.31-0ubuntu9) ...
    Setting up libcc1-0:amd64 (10-20200411-0ubuntu1) ...
    Setting up liblsan0:amd64 (10-20200411-0ubuntu1) ...
    Setting up libitm1:amd64 (10-20200411-0ubuntu1) ...
    Setting up gcc-9-base:amd64 (9.3.0-10ubuntu2) ...
    Setting up libtsan0:amd64 (10-20200411-0ubuntu1) ...
    Setting up libctf0:amd64 (2.34-6ubuntu1) ...
    Setting up libgmp-dev:amd64 (2:6.2.0+dfsg-4) ...
    Setting up libasan5:amd64 (9.3.0-10ubuntu2) ...
    Setting up cpp-9 (9.3.0-10ubuntu2) ...
    Setting up libc6-dev:amd64 (2.31-0ubuntu9) ...
    Setting up binutils-x86-64-linux-gnu (2.34-6ubuntu1) ...
    Setting up binutils (2.34-6ubuntu1) ...
    Setting up libgcc-9-dev:amd64 (9.3.0-10ubuntu2) ...
    Setting up zlib1g-dev:amd64 (1:1.2.11.dfsg-2ubuntu1) ...
    Setting up cpp (4:9.3.0-1ubuntu2) ...
    Setting up gcc-9 (9.3.0-10ubuntu2) ...
    Setting up libstdc++-9-dev:amd64 (9.3.0-10ubuntu2) ...
    Setting up gcc (4:9.3.0-1ubuntu2) ...
    Setting up g++-9 (9.3.0-10ubuntu2) ...
    Setting up g++ (4:9.3.0-1ubuntu2) ...
    update-alternatives: using /usr/bin/g++ to provide /usr/bin/c++ (c++) in auto mode
    Processing triggers for libc-bin (2.31-0ubuntu9) ...
    Processing triggers for man-db (2.9.1-1) ...
    Processing triggers for install-info (6.7.0.dfsg.2-5) ...

    Using generic bindist...

    % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
    0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    100   655  100   655    0     0    624      0  0:00:01  0:00:01 --:--:--  639k
    100 13.8M  100 13.8M    0     0  2716k      0  0:00:05  0:00:05 --:--:-- 4590k
    Installing Stack to: /usr/local/bin/stack...

    About to use 'sudo' to run the following command as root:
        install -c -o 0 -g 0 -m 0755 /tmp/tmp.rxXLjYdeAm/stack /usr/local/bin
    in order to copy 'stack' to the destination directory.

    [sudo] password for wsl2:

    -------------------------------------------------------------------------------

    Stack has been installed to: /usr/local/bin/stack

    WARNING: '/home/wsl2/.local/bin' is not on your PATH.
        For best results, please add it to the beginning of PATH in your profile.
    ```
    </details>

1. Обращаем внимание на предупреждение про `~/.local/bin` и делаем нужную папку
    <details>
    <summary>
    по умолчанию профиль пользователя уже настроен на добавление этой папки к переменной `PATH`

    ```console
    wsl2@w10m2:~$ mkdir -p ~/.local/bin
    ```
    </summary>
    
    ```console
    wsl2@w10m2:~$ cat ~/.profile
    # ~/.profile: executed by the command interpreter for login shells.
    # This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
    # exists.
    # see /usr/share/doc/bash/examples/startup-files for examples.
    # the files are located in the bash-doc package.

    # the default umask is set in /etc/profile; for setting the umask
    # for ssh logins, install and configure the libpam-umask package.
    #umask 022

    # if running bash
    if [ -n "$BASH_VERSION" ]; then
        # include .bashrc if it exists
        if [ -f "$HOME/.bashrc" ]; then
            . "$HOME/.bashrc"
        fi
    fi

    # set PATH so it includes user's private bin if it exists
    if [ -d "$HOME/bin" ] ; then
        PATH="$HOME/bin:$PATH"
    fi

    # set PATH so it includes user's private bin if it exists
    if [ -d "$HOME/.local/bin" ] ; then
        PATH="$HOME/.local/bin:$PATH"
    fi
    ```
    </details>

1. Перезайдём в консоль, чтобы изменения вступили в силу
    ```console
    wsl2@w10m2:~$ logout
    PS C:\Users\user> ubuntu2004.exe
    wsl2@w10m2:~$ stack path --local-bin
    /home/wsl2/.local/bin
    ```

1. Разворачиваем Haskell-инфраструктуру в своём профиле
    <details>
    <summary>
    
    ```console
    wsl2@w10m2:~$ wsl2@w10m2:~$ stack setup
    ```
    </summary>
    
    ```console
    Writing implicit global project config file to: /home/wsl2/.stack/global-project/stack.yaml
    Note: You can change the snapshot via the resolver field there.
    Using latest snapshot resolver: lts-16.15
    Preparing to install GHC (tinfo6) to an isolated location.
    This will not interfere with any system-level installation.
    Downloaded ghc-tinfo6-8.8.4.
    Installed GHC.
    stack will use a sandboxed GHC it installed
    For more information on paths, see 'stack path' and 'stack exec env'
    To use this GHC and packages outside of a project, consider using:
    stack ghc, stack ghci, stack runghc, or stack exec
    ```
    </details>

1. Проверяем установку сборкой тестового проекта
    <details>
    <summary>

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
    </summary>

    ```console
    wsl2@w10m2:~$ stack new test
    Downloading template "new-template" to create project "test" in test/ ...

    The following parameters were needed by the template but not provided: author-name
    You can provide them in /home/wsl2/.stack/config.yaml, like this:
    templates:
    params:
        author-name: value
    Or you can pass each one as parameters like this:
    stack new test new-template -p "author-name:value"


    The following parameters were needed by the template but not provided: author-email, author-name, category, copyright, github-username
    You can provide them in /home/wsl2/.stack/config.yaml, like this:
    templates:
    params:
        author-email: value
        author-name: value
        category: value
        copyright: value
        github-username: value
    Or you can pass each one as parameters like this:
    stack new test new-template -p "author-email:value" -p "author-name:value" -p "category:value" -p "copyright:value" -p "github-username:value"

    Looking for .cabal or package.yaml files to use to init the project.
    Using cabal packages:
    - test/

    Selecting the best among 18 snapshots...

    * Matches lts-16.15

    Selected resolver: lts-16.15
    Initialising configuration using resolver: lts-16.15
    Total number of user packages considered: 1
    Writing configuration to file: test/stack.yaml
    All done.
    /home/wsl2/.stack/templates/new-template.hsfiles:    3.72 KiB downloaded...
    wsl2@w10m2:~$ cd ./test/
    wsl2@w10m2:~/test$ stack build
    [1 of 2] Compiling Main             ( /home/wsl2/.stack/setup-exe-src/setup-mPHDZzAJ.hs, /home/wsl2/.stack/setup-exe-src/setup-mPHDZzAJ.o )
    [2 of 2] Compiling StackSetupShim   ( /home/wsl2/.stack/setup-exe-src/setup-shim-mPHDZzAJ.hs, /home/wsl2/.stack/setup-exe-src/setup-shim-mPHDZzAJ.o )
    Linking /home/wsl2/.stack/setup-exe-cache/x86_64-linux-tinfo6/tmp-Cabal-simple_mPHDZzAJ_3.0.1.0_ghc-8.8.4 ...
    Building all executables for `test' once. After a successful build of all of them, only specified executables will be rebuilt.
    test> configure (lib + exe)
    Configuring test-0.1.0.0...
    test> build (lib + exe)
    Preprocessing library for test-0.1.0.0..
    Building library for test-0.1.0.0..
    [1 of 2] Compiling Lib
    [2 of 2] Compiling Paths_test
    Preprocessing executable 'test-exe' for test-0.1.0.0..
    Building executable 'test-exe' for test-0.1.0.0..
    [1 of 2] Compiling Main
    [2 of 2] Compiling Paths_test
    Linking .stack-work/dist/x86_64-linux-tinfo6/Cabal-3.0.1.0/build/test-exe/test-exe ...
    test> copy/register
    Installing library in /home/wsl2/test/.stack-work/install/x86_64-linux-tinfo6/c733b47589def700198e64ab41cee71169ccfacbb8a95176edcfff30ed0174e6/8.8.4/lib/x86_64-linux-ghc-8.8.4/test-0.1.0.0-7rsN3rTO0kgD6zYZVJXKz
    Installing executable test-exe in /home/wsl2/test/.stack-work/install/x86_64-linux-tinfo6/c733b47589def700198e64ab41cee71169ccfacbb8a95176edcfff30ed0174e6/8.8.4/bin
    Registering library for test-0.1.0.0..
    wsl2@w10m2:~/test$ stack exec test-exe
    someFunc
    wsl2@w10m2:~/test$ stack install
    Copying from /home/wsl2/test/.stack-work/install/x86_64-linux-tinfo6/c733b47589def700198e64ab41cee71169ccfacbb8a95176edcfff30ed0174e6/8.8.4/bin/test-exe to /home/wsl2/.local/bin/test-exe

    Copied executables to /home/wsl2/.local/bin:
    - test-exe
    wsl2@w10m2:~/test$ cd ..
    wsl2@w10m2:~$ test-exe
    someFunc
    ```
    </details>

1. Проверка версии `stack`'а
    ```console
    wsl2@w10m2:~$ stack exec stack -- --version
    Version 2.3.3, Git revision cb44d51bed48b723a5deb08c3348c0b3ccfc437e x86_64 hpack-0.33.0
    ```

## [ <kbd>↑</kbd> ](#up) <a name="step3">[Шаг 3 - Haskell IDE Engine (HIE)](#step3)</a>

1. Ставим непосредственно движок HIE [из исходников](https://github.com/haskell/haskell-ide-engine#installation-from-source)
    <details>
    <summary>
    
    ```console
    wsl2@w10m2:~$ git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules
    wsl2@w10m2:~$ cd haskell-ide-engine
    wsl2@w10m2:~$ stack ./install.hs help
    wsl2@w10m2:~$ stack ./install.hs hie -q
    ```
    </summary>

    ```console
    wsl2@w10m2:~/haskell-ide-engine$ stack ./install.hs help
    rts-1.0: Warning: .:464:1: The field "hugs-options" is deprecated. hugs isn't supported anymore
    [1 of 2] Compiling Main             ( /home/wsl2/.stack/setup-exe-src/setup-mPHDZzAJ.hs, /home/wsl2/.stack/setup-exe-src/setup-mPHDZzAJ.o )
    [2 of 2] Compiling StackSetupShim   ( /home/wsl2/.stack/setup-exe-src/setup-shim-mPHDZzAJ.hs, /home/wsl2/.stack/setup-exe-src/setup-shim-mPHDZzAJ.o )
    Linking /home/wsl2/.stack/setup-exe-cache/x86_64-linux-tinfo6/tmp-Cabal-simple_mPHDZzAJ_2.4.0.1_ghc-8.6.5 ...

    Usage:
        stack install.hs <target> [options]
        or
        cabal v2-run install.hs --project-file install/shake.project -- <target> [options]

    Targets:
        help                Show help message including all targets

        hie                 Install hie with the latest available GHC and the data files
        latest              Install hie with the latest available GHC
        data                Get the required data-files for `hie` (Hoogle DB)
        hie-8.4.2           Install hie for GHC version 8.4.2
        hie-8.4.3           Install hie for GHC version 8.4.3
        hie-8.4.4           Install hie for GHC version 8.4.4
        hie-8.6.4           Install hie for GHC version 8.6.4
        hie-8.6.5           Install hie for GHC version 8.6.5
        hie-8.8.2           Install hie for GHC version 8.8.2
        hie-8.8.3           Install hie for GHC version 8.8.3

        dev                 Install hie with the default stack.yaml

        icu-macos-fix       Fixes icu related problems in MacOS

    Options:
        -j[N], --jobs[=N]   Allow N jobs/threads at once [default number of CPUs].
        -s, --silent        Don't print anything.
        -q, --quiet         Print less (pass repeatedly for even less).
        -V, --verbose       Print more (pass repeatedly for even more).

    Build completed in 0.00s
    ```
    </details>

1. Ставим движок поиска документации [Hoogle](https://github.com/ndmitchell/hoogle/blob/master/docs/Install.md)
    ```console
    stack install hoogle
    hoogle generate
    stack haddock --hoogle
    ```


    cabal install hoogle
    echo >> ~/.ghci ':def hoogle \x -> return $ ":!hoogle " ++ x'


## [ <kbd>↑</kbd> ](#up) <a name="step4">[Шаг 4](#step4)</a>
## [ <kbd>↑</kbd> ](#up) <a name="step5">[Шаг 5](#step5)</a>
<details>
<summary>

```console
```
</summary>

```console
```
</details>


