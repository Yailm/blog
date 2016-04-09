title: 恢复Archlinux的boot分区
date: 2016-01-15 21:50:55
tags:
  - Archlinux
---
如果平时有备份boot分区的习惯，在发生什么意外的时候能够及时恢复，但由于arch的滚动升级，内核的更新往往很快，而碰巧备份文件里内核映象低于目前系统的版本，直接使用备份可能会出现问题。

在boot分区已经损坏的情况下，不能进入系统，我们只能借助于liveCD的帮助。在电脑上做好一个arch的usb，我们将用它来启动系统。

    $ lsblk 
    sda      8:0    0 931.5G  0 disk 
    +-sda1   8:1    0   268M  0 part /boot
    +-sda6   8:6    0    20G  0 part /
    +-sda7   8:7    0    15G  0 part /var
    +-sda8   8:8    0    25G  0 part /home

    mount /dev/sda6 /mnt
    mount /dev/sda1 /mnt/boot
    mount /dev/sda7 /mnt/var
    mount /dev/sda8 /mnt/home
    arch-chroot /mnt /bin/bash

我们在live上面挂载好分区，然后chroot进去，之后就如同我们操作平时的电脑一样。现在只要恢复备份，然后创建当前系统内核的映象就行了。高端的可以自己生成，参考[ArchWiki mkinitcpio](https://wiki.archlinux.org/index.php/Mkinitcpio)，但我比较懒就一条命令解决了`sudo pacman -S linux`，会重新安装然后再自动生成。如果现在发现没有网络，可以`exit`，配置好网络再`chroot`
