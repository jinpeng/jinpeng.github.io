title: Python debug with PDB
date: 2016-06-01 18:18:18
tags:
---
## PDB调试启动方式

- 对于从命令行运行的程序

```shell
$ python -m pdb myscript.py
```

- 对于较为复杂大型程序或者需要特殊脚本启动的程序

在感兴趣的代码中插入：

```python
import pdb
pdb.set_trace()
```
然后用原有的方式启动，程序运行到代码段会进入PDB调试状态

- 在Python REPL中调试

```python
>>> import pdb
>>> import mymodule
>>> pdb.run('mymodule.test()')
```

## 常用PDB命令

- 帮助命令

```
(Pdb) h
Documented commands (type help <topic>):
========================================
EOF    bt         cont      enable  jump  pp       run      unt
a      c          continue  exit    l     q        s        until
alias  cl         d         h       list  quit     step     up
args   clear      debug     help    n     r        tbreak   w
b      commands   disable   ignore  next  restart  u        whatis
break  condition  down      j       p     return   unalias  where

Miscellaneous help topics:
==========================
exec  pdb

Undocumented commands:
======================
retval  rv
```

- 查看当前代码片段

```
(Pdb) l
  1  ->	import mysql.connector
  2  	from mysql.connector import fabric
  3  	from mysql.connector import errors
  4  	import time
  5  	config = {
  6  	    'fabric': {
  7  	        'host': 'localhost',
  8  	        'port': 32274,
  9  	        'username': 'admin',
 10  	        'password': 'admin',
 11  	        'report_errors': True
```

- 设置断点

```
(Pdb) b 122
```

- 设置条件断点

```
(Pdb) b 122, a == 2
```

- 清除断点

```
(Pdb) cl(ear)
```
```
(Pdb) cl(ear) 2
```

- 禁用／激活断点

```
(Pdb) disable 3
```
```
(Pdb) enable 3
```

- 单步执行

Step over

```
(Pdb) n(ext)
```

Step into

```
(Pdb) s(tep)
```

- 继续执行

```
(Pdb) c(ont(inue))
```

- 跳转

跳转不会执行被跳过的代码

```
(Pdb) j(ump) 18
```

- 打印变量

```
(Pdb) p fcnx
```

- 打印当前函数的参数

```
(Pdb) a(rgs)
```

- 执行表达式

```
(Pdb) !fcnx=None
```

- 重新启动调试器

```
(Pdb) run [args]
```

- 退出调试

```
(Pdb) q(uit)
```

