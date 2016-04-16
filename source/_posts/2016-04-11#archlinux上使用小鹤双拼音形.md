title: 在archlinux上使用小鹤双拼音形
date: 2016-04-11 11:14:23
tags:
  - archlinux
---
这是以前配置过的，不过近来写了些相应的脚本，也算是不太水的文章。同时也安利下[小鹤双拼](http://www.flypy.com)，能减少击键次数，提高打字速度

首先从源里安装fcitx

    sudo pacman -S fcitx fcitx-gtk2 fcitx-gtk3 fcitx-qt4 fcitx-qt5 fcitx-rime

这如果想要有图形界面来设置fcitx，可以安装**fcitx-configtool**。**fcitx-rime**是中文输入法[rime](http://rime.im)的fcitx版本，需要设置的东西比较多，自定义当然就强了。

我使用的桌面管理器，并不是完善的环境，系统启用输入法需要在.xinitrc中加入以下环境变量

    export GTK_IM_MODULE=fcitx
    export QT_IM_MODULE=fcitx
    export XMODIFIERS=@im=fcitx

然后在桌面的autostart中加入fcitx的启动，这样进入后就可以调出fcitx。若是其他完善的桌面环境配置起来会容易些，具体看[fcitx on archwiki](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

下面对rime进行配置，顺便一提，fcitx的相关文件默认在`$HOME/.config/fcitx`中，rime的配置文件就在fcitx目录下的rime中。

首先新建用户自己的配置文件**default.custom.yaml**，不建议直接修改在default.yaml上，因为版本升级会覆盖该文件，导致丢失配置

    patch:
      "ascii_composer/good_old_caps_lock": true
      key_binder:
        Caps_Lock: noop
        Control_L: clear        # 清空
        Control_R: commit_text  # 提交
        Shift_L: commit_code    # 提交输入码
        Shift_R: inline_ascii
      menu:
        page_size: 2        # 候选词个数
      schema_list:
        - {schema: flypy}   # 输入方案选择，等下就会配置
      "switcher/hotkeys":
        - "Control+grave"   # 菜单键，设置为ctrl+`(即是TAB上面的那个)

然后我们新建输入方案，**flypy.schema.yaml**

    # Rime schema settings
    # encoding: utf-8

    schema:
      schema_id: flypy
      name: 小鶴音形
      version: "1.2"        # 自己定义的版本号

    switches:
      - name: ascii_mode    # 上面提到的菜单键的菜单选项
        reset: 1            # 预设值，默认为0，也就是下面的中文
        states: [ 中文, 西文 ]
      - name: full_shape
        states: [ 半角, 全角 ]
      - name: ascii_punct
        reset: 0
        states: [ 中标, 英标 ]

    engine:                 # 使用一些输入法引擎
      processors:           # 这里只使用了一些基本配置，[雪齋的文檔](https://github.com/LEOYoon-Tsaw/Rime_collections/blob/master/Rime_description.md)详细介绍了这部分的内容
        - ascii_composer
        - recognizer
        - key_binder
        - speller
        - punctuator
        - selector
        - navigator
        - express_editor
      segmentors:
        - ascii_segmentor
        - matcher
        - abc_segmentor
        - punct_segmentor
        - fallback_segmentor
      translators:
        - punct_translator
        - table_translator
      filters:
        - simplifier
        - uniquifier

    speller:
      alphabet: 'abcdefghjiklmnopqrstuvwxyz'
      delimiter: "`"
      max_code_length: 4
      auto_select: true         # 4字自动上屏

    translator:
      dictionary: flypy
      enable_charset_filter: true
      enable_completion: false
      enable_sentence: true
      enable_user_dict: false

    punctuator:
      import_preset: default

    key_binder:
      import_preset: default
      bindings:
        - {accept: semicolon, send: 2, when: has_menu}      # 分号选择第二个选项

    recognizer:
      import_preset: default

再来看看flypy.dict.yaml的结构

    ----
    name : flypy
    version :  "1.1"
    sort : original         # 顺序不变
    use_preset_vocabulary : false   # 不引入「八股文」〔含字詞頻、詞庫〕
    ...
    啊      a
    按      a
    阿      aa
    阿坝    aab
    。。。

字库的获得一般在小鹤群，也可以从飞扬输入法中导出，由于词库也不是一直是固定的。散步的鹤（也是就创造者）会不定时的更新词库。所以我写了个难使的perl脚本来更新字典，前提是字典被放到了/tmp目录下了

    #!/usr/bin/perl
    # 这脚本将下载下来飞扬码表转化成rime字典
    use warnings;
    use Data::Dumper;

    my $flypy_new = glob '/tmp/飞扬码表*.txt';      # 字典文件
    my $flypy_ori = '/home/fio/.config/fcitx/rime/flypy.dict.yaml';     # 当前配置文件路径

    sub sort_by_code {
        my $codes = [];
        for ($a, $b) {
            /\t(\w+)$/;
            push @$codes, $1;
        }
        $codes->[0] cmp $codes->[1];
    }

    rename $flypy_ori => $flypy_ori.'.bak';     # 备份
    open ORI, '<', $flypy_new or die $@;
    my $dict = [];
    while (<ORI>) {
        next if /#类/;
        if (/^(.*?)#.*[^\d](\d+)/) {
            push @$dict, [$1, $2];
        }
    }
    close ORI;
    my @fina = map { $_->[0] }
        sort { $b->[1] <=> $a->[1] } @$dict;

    @fina = sort sort_by_code @fina;
    my $fina = join "\n", @fina;

    $/ = '';
    my $prefix = <DATA>;
    open OUTP, '>', $flypy_ori or die $@;
    print OUTP $prefix;
    print OUTP $fina;
    close OUTP;

    exec "vim $flypy_ori";  # 查看结果，更新版本号之类的

    __DATA__
    ---
    name : flypy
    version :  "1.1"
    sort : original
    use_preset_vocabulary : false
    ...

最后记得要部署一下

    rime_deployer --build [rime目录]

使用小鹤双拼的时候偶尔会卡字，不清楚字是怎么拆分的，所以反查也是很重要的。一开始没有注意到rime可以设置反查，就又手撸了脚本来查询，效果也不错，而且支持整句，合并了上面的代码

    #!/usr/bin/perl

    use feature qw(say);
    use warnings;
    use Encode;

    my $flypy_ori = '/home/fio/.config/fcitx/rime/flypy.dict.yaml';

    my $arg = $ARGV[0] or &usage;
    if ($arg eq 'update') {
        ## 更新码表
        my $flypy_new = $ARGV[1] || glob '/tmp/飞扬码表*.txt';

        rename $flypy_ori => $flypy_ori.'.bak';
        open ORI, '<', $flypy_new or die $@;
        my $dict = [];
        while (<ORI>) {
            next if /#类/;
            if (/^(.*?)#.*[^\d](\d+)/) {
                push @$dict, [$1, $2];
            }
        }
        close ORI;
        my @fina = map { $_->[0] }
            sort { $b->[1] <=> $a->[1] } @$dict;

        my $fina = join "\n", sort sort_by_code @fina;

        $/ = '';
        my $prefix = <DATA>;
        open OUTP, '>', $flypy_ori or die $@;
        print OUTP $prefix;
        print OUTP $fina;
        close OUTP;

        exec "vim $flypy_ori";
    } else {
        $key = decode('utf-8', $ARGV[0]);
        my @keys = map { encode('utf-8', $_) } split(//, $key);
        my $hah = [];
        my $dicts = {};
        for my $i (0..$#keys) {
            my $max = $i+3 > $#keys ? $#keys : $i+3;
            for my $j ($i..$#keys) {
                push @$hah, join('', @keys[$i..$j]);
            }
        }
        my $sed_string = join(',', @$hah) =~ s/([^\d]+?)(,|$)/\/^$1\\t\/p\n/gr;
        open SED, "sed -ne \"$sed_string\" $flypy_ori | uniq |";
        while (<SED>) {
            /^(.*)\t(.*)$/;
            push @{$dicts->{$1}}, $2;
        }
        close SED;
        for (@$hah) {
            if ($dicts->{$_}) {
                print "└" if length($_) > 3;
                print "$_\t\t\t", join(', ', @{$dicts->{$_}}), "\n";
            }
        }
    }

    sub sort_by_code {
        my $codes = [];
        for ($a, $b) {
            /\t(\w+)$/;
            push @$codes, $1;
        }
        $codes->[0] cmp $codes->[1];
    }

    sub usage {
        print "Usage:
        flypy update [newdict_path]
        flypy keyword\n";
        exit 0;
    }

    __DATA__
    ---
    name : flypy
    version :  "1.1"
    sort : original
    use_preset_vocabulary : false
    ...

这就是脚本最后的形态了，又能查字又能更新

    flypy 我想知道这是为什么
    ---output---
    我                      w, wop, wopd
    想                      xl, xlmx
    知                      viu, viuk
    └知道                   vd
    道                      dc, dcz, dczz
    这                      v, vezw, vwz, vwzw
    └这是为什么             vuwm
    是                      u, uior
    为                      ww, wwdd
    └为什么                 wsm, wume
    什                      ufr, ufru, uiru
    └什么                   sm, ufme
    么                      me, mep, meps, mop, mops

最后，也是希望自己将能做到的麻烦事都转化成或多或少的脚本
