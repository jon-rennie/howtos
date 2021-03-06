## Installing Fedora 33 as a WSL2 distribition

This howto describes how to add Fedora 33 as a distribution in Windows Subsystem for Linux (WSL) version 2

Based on article at [FedoraMagazine.org](https://fedoramagazine.org/wsl-fedora-33/)


## 001 - download a rootfs image
Download tarball from
https://github.com/fedora-cloud/docker-brew-fedora/raw/33/x86_64/fedora-33.20201218-x86_64.tar.xz and extract the .tar file

    PS C:\Users\jon> dir Downloads\fedora-*.tar
    d-----        20/12/2020  10:20 am                fedora-33.20201218-x86_64.tar

## 002 - import the rootfs

Create a directory to hold your distro files (optional)

    PS C:\Users\jon> mkdir WSL-Distros

Import the rootfs

    PS C:\Users\jon> wsl.exe --import Fedora-33 C:\Users\Jon\WSL-Distros\Fedora-33 C:\Users\Jon\Downloads\fedora-33.20201218-x86_64.tar

Check it has imported

    PS C:\Users\jon> wsl.exe -l -v
        NAME            STATE           VERSION
        Ubuntu-20.04    Running         2
        Fedora-33       Stopped         2

## 003 - start the distro, update packages, add a non-root user
    PS C:\Users\jon> wsl -d Fedora-33
    [root@jon-lenovo-m700 jon]# dnf update
    [root@jon-lenovo-m700 jon]# dnf install wget curl sudo ncurses dnf-plugins-core dnf-utils passwd findutils
    [root@jon-lenovo-m700 jon]# useradd -G wheel jon 
    [root@jon-lenovo-m700 jon]# passwd jon
    [root@jon-lenovo-m700 jon]# exit

## 004 - can now start the distro as new user
    PS C:\Users\jon> wsl -d Fedora-33 -u jon
    [jon@jon-lenovo-m700 jon]$ id -u
    1000
    [jon@jon-lenovo-m700 jon]$ exit

## 005 -set Windows Property to start as the user you created - uid 1000
    PS C:\Users\jon> Get-ItemProperty Registry::HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss\*\ DistributionName | Where-Object -Property DistributionName -eq Fedora-33  | Set-ItemProperty -Name DefaultUid -Value 1000

## 006 - add COPR repo for wslu utilities and install wslu
    PS C:\Users\jon> wsl -d Fedora-33
    [jon@jon-lenovo-m700 jon]$ sudo dnf copr enable trustywolf/wslu
    [jon@jon-lenovo-m700 jon]$ sudo dnf install wslu
    [jon@jon-lenovo-m700 jon]$ wslfetch

            /:-------------:\         Windows Subsystem for Linux (WSL2)
            :-------------------::       jon@jon-lenovo-m700
        :-----------/shhOHbmp---:\     Build: 19042
        /-----------omMMMNNNMMD  ---:    Branch: vb_release
    :-----------sMMMMNMNMP.    ---:   Release: Fedora 33 (Container Image)
    :-----------:MMMdP-------    ---\  Kernel: Linux 4.19.128-microsoft-standard
    ,------------:MMMd--------    ---:  Uptime: 1d 21h 6m
    :------------:MMMd-------    .---:
    :----    oNMMMMMMMMMNho     .----:
    :--     .+shhhMMMmhhy++   .------/
    :-    -------:MMMd--------------:
    :-   --------/MMMd-------------;
    :-    ------/hMMMy------------:
    :-- :dMNdhhdNMMNo------------;
    :---:sdNMMMMNds:------------:
    :------:://:-------------::
    :---------------------://

## 007 - create windows shortcut to start Fedora 33
    [jon@jon-lenovo-m700 ~]$ wslusc -I
    [info] Welcome to wslu shortcut creator interactive mode.
    [input] Command (Without Parameter): /bin/bash
    [input] Command param:
    [input] Shortcut name [optional, ENTER for default]: Fedora33
    [input] Is it a GUI application? [if yes, input 1; if no, input 0]: 0
    [input] Pre-executed command [optional, ENTER for default]:
    [input] Custom icon Linux path (support ico/png/xpm/svg) [optional, ENTER for default]:
    [info] Create shortcut Fedora33.lnk successful
Then you can find shortcut on Desktop, modify .ico link, apply. Right click on shortcut, pin to start menu.

## 008 - what next?

Next up, try adding pdoman, buildah and skopeo to your Fedora 33 distro, so you can start building and running containers without needing to use docker or root privileges. Use this [howto](https://github.com/jon-rennie/howtos/blob/main/podman%20in%20Fedora%2033%20under%20WSL2.md)