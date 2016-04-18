title: 'u_undo: line numbers wrong'
date: 2016-04-18 21:19:17
tags:
 - vim
---
在一次强制关机后，开机准备继续写wiki，然后竟然发现perl.md只剩下117行，突然感觉自己的汗毛都倒立了。尝试按了下u，vim底下给出错误消息`u_undo: line numbers wrong`并不能恢复，虽然用了git做版本控制，上次commit已经是刚创建这个wiki的时候了，并不能帮到我，提交的时候还是要勤奋点好。

这下让我有点慌了，记得自己在.vimrc中有设置undofile

```vim
try
    set undodir=~/.vim/undodir
    set undofile
catch
```

我马上切换到undodir下，看到了个1.7m大小的undo文件，心里放松了许多，顺手备份了下。接下来尝试创建新的文件来恢复备份也失败了，的确怎么可能做得到

在网上逛了逛，发现stackoverflow上有[类似的提问](http://stackoverflow.com/questions/21385956/undo-recovery-from-swap-file-in-vim)

`:undolist`命令的确能看到我在中午时的历史，回答中提到的插件，我尝试了Gundo。在Vundle加入以下代码，然后安装插件

    Bundle 'sjl/gundo.vim'

`:GundoToggle`，打开了个视图，我找到了中午的记录，跳过有错误的记录，完成恢复，顺便commit了
