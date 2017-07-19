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

# -*- coding: utf-8 -*-
```

每个 Python 文件都应当添加如上两行，第一行告诉 Linux / OS X 系统，该文件是可执行程序，第二行说明该文档采用 utf-8 编码。

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

\n  换行，\t 制表，用`r''`表示`''`内部的字符串默认不转义。

用 ''' ’'' 包围多行文本，就不必使用 \n 换行。

#### 布尔运算符

and, or, not，结果为True & False

#### 空值

None

#### 运算符

使用 / 的计算结果是浮点数：9/3=3.0，使用 // 才是整除：10//3=3。

#### 浮点数

$$1.23*10^9$$就是`1.23e9`，或者`12.3e8`，0.000012可以写成`1.2e-5。`

#### 变量

把一个变量`a`赋值给另一个变量`b`，这个操作实际上是把变量`b`指向变量`a`所指向的数据。

### 基本类型

#### byte

`bytes`类型的数据用带`b`前缀的单引号或双引号表示：x = b'ABC'。每个字符只占用1个字节。

#### str 占位符

| %d | 整数 |
| :--- | :--- |
| %f | 浮点数 |
| %s | 字符串 |
| %x | 十六进制整数 |

```
'Hi, %s, you have $%d.' % ('Michael', 1000000)。
```

```
>>> '%2d-%02d' % (3, 1)
' 3-01'
>>> '%.2f' % 3.1415926
'3.14'
```

```
>>> 'growth rate: %d %%' % 7
'growth rate: 7 %'
```

### 集合

#### list

list是一种有序的集合，可以随时添加和删除其中的元素。

```
classmates = ['Michael', 'Bob', 'Tracy']
```

用-1做索引，可以取得最后一个元素。

```
classmates[-1]：'Tracy’
```

list 中可以存放不同类型的数据，包括 list 。

可以直接赋值给索引位置： classmates\[1\] = 'Sarah'

#### tuple

tuple一旦初始化就不能修改。

只有1个元素的tuple定义时必须加一个逗号`,`来消除歧义：t = \(1,\)

* 一个可变的tuple：t = \('a', 'b', \['A', 'B'\]\)，可以修改其中 list 的元素。

### 条件判断

#### if x:

只要`x`是非零数值、非空字符串、非空list等，就判断为`True`，否则为`False`。

### 循环

#### for x in ...

#### while

* 少用 break 和 continue，几乎所有 break 和 continue 都可以通过优化逻辑省略。

### dict

使用键-值（key-value）存储，具有极快的查找速度。

```
d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
```

* 通过 d\['Bob'\] 可以取到 value 值，还可以通过 d\['Mike'\] = 75 向字典中添加键值对。

* 如果 key 不存在，dict 会报错，取值前应当用 'Thomas' in d 进行判断。

### set

保存不重复的一组数据。创建一个 set ，需要提供一个 list 作为输入集合。

```
s = set([1, 2, 3]) 输出 {1,2,3}
```



