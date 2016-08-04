title: archlinuxarm上配置使用AUR
date: 2016-08-04 20:41:41
tags:
  - archlinux
---
在树莓派上安装了个archlinuxarm，习惯性想使用AUR里的软件，配置起来有点麻烦就记录一下

首先如通常一样，编辑/etc/pacman.conf文件添加新的archlinuxcn源

    vi /etc/pacman.conf

    # 在最后面追加以下内容
    [archlinuxcn]
    SigLevel=Never
    Server=http://repo.archlinuxcn.org/any

正常情况下，地址是`http://repo.archlinuxcn.org/$arch`，由于是arm版本，在浏览器中打开http://repo.archlinuxcn.org ，发现里面是没有arm之类的目录的。所以如果使用`$arch`的话，你就会得到个'failed retrieving file 'archlinuxcn.db' ....'的提示，并不是网络不好，404表明没有个这文件

之后，在更新源后直接安装yaourt会提示要package-query>=1.8，但package-query不是在AUR里吗？只能手动下载这个包离线安装

    wget https://aur.archlinux.org/cgit/aur.git/snapshot/package-query.tar.gz
    tar -xvf package-query.tar.gz

注意有些依赖

    sudo pacman -S gcc yaji make fakeroot pkg-config
    makepkg -i

最后就能直接安装yaourt了

    sudo pacman -S yaourt
