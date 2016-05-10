title: python将list转换成可变参数
date: 2016-05-08 13:15:44
tags:
  python
---
这是刚刚学会的新技能，比如asyncio模块中的如下一种方法

    import asyncio
    asyncio.subprocess.create_subprocess_exec(program, *args, 
        stdin=None, stdout=None, stderr=None, loop=None, limit=65536, **kwds)

这里的program和args是分开的，如果你有一串命令要执行，比如`ls -al /tmp`。

这里我们要同时将命令分解成ls和它的参数，而且我们若在函数中放入列表就会产生错误，通过计算参数的个数再传入函数也是不合理

然而今天才学到，python可以这样写
```python
command = 'ls -al /tmp'
command_list = command.split()
asyncio.subprocess.create_subprocess_exec(*command_list)
```
不仅列表可以，同样也能适用于字典

    def test(**kw):
        print(kw)

    a = {'a':1, 'c':2}
    test(**a)
