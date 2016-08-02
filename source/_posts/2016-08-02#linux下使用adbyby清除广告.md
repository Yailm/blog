title: linux下使用adbyby清除广告
date: 2016-08-02 20:47:36
tags:
  - archlinux
---
[adbyby](http://www.adbyby.com/)是一款跨平台的专业去广告软件，虽然提供的linux比较简陋，但也保持了它优秀的性能（不卡网，过滤强劲）。虽然现在浏览器的插件已经包揽了去广告的市场，但是在使用其他的软件就不能获得过滤（比如我使用的goldendic电子词典时，载入网络查词时也加载广告，占用了许多时间）

要安装adbyby，官网提供了64位和86位，我这里是64位系统（话说32位也。。），顺便解压一下

    wget http://update.adbyby.com/download/linux.64.tar.gz
    tar -xvf linux.64.tar.gz

得到一个bin文件夹，里面就有我们想要的可执行文件了，直接执行`./adbyby &`就在本机的8118端口开启过滤了。但真正安定下来还需要做几件事

- 代理的设置

  - 设置系统代理，在.bashrc中追加

        export http_proxy=http://127.0.0.1:8118/
        export https_proxy=$http_proxy

    只要软件使用了系统代理，通过的数据就会被过滤。但也有的软件会`direct link`，但也提供系统代理（如chrome、goldendict），需要自己到高级设置里去找

  - chromium系的浏览器推荐使用[SwitchyOmega](https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=en-US)，好用是有道理的，如果只想在浏览器里使用的话，新建个配置adbyby，设定到本机的8118端口即可

- adbyby的自启动

  adbyby有专门的配置文件adhook.ini，也提供了[配置说明](http://www.adbyby.com/setup.htm)，但没有在默认adhook.ini出现的选项都是无效的，而且已经不用再设置了

  一般方便管理软件的启动我倾向于写成system unit，在`/usr/lib/systemd/system/`下创建文件adbyby.service，内容如下

      #systemd service configuration file for adbyby

      [Unit]
      Description=Adbyby
      After=network.target

      [Service]
      User=fio
      Type=simple
      ExecStart=/path/to/adbyby binary

      [Install]
      WantedBy=multi-user.target

  然后设置成开机启动项

      systemctl daemon-reload
      systemctl start adbyby.service    # 启动adbyby服务
      systemctl enable adbyby.service   # 设置成自启动

