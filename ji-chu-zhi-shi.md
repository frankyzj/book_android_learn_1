# 基础知识

#### Mac

在终端中输入 Python3。

#### 解释器

常见的是 cpython。

#### 命令行下执行 .py 文件

```
C:\work>python calc.py
```

#### 在 Mac 下执行 .py 文件

在文件第一行加入如下代码：

```
#!/usr/bin/env python3
```

然后通过命令行给 .py 文件以执行权限：

```
$ chmod a+x hello.py
```

最后输入\(文件名前加 ./\)

```
./xx.py
```

#### 在 Windows 下执行 .py 文件

在环境变量PATHEXT中添加一个.PY

```
D:\Programming\Python>set PathExt=.PY;%PathExt%

D:\Programming\Python>hello
Hello,world
```

#### 转义符的使用

如果字符串内部既包含`'`又包含`"，`可以用转义字符`\`来标识，比如：

```
'I\'m \"OK\"!'
```



