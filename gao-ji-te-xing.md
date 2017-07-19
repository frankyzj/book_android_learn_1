### 切片Slice

取一个list或tuple的部分元素。

```
>>> L[1:3]
['Sarah', 'Tracy']
```

取 L 的前10个数

`L[:10]`

取 L 的后10个数

```
L[-10:]
```

前10个数，每两个取一个：

```
L[:10:2]
```

所有数，每5个取一个：

```
L[::5]
```

复制：

```
L[:]
```

其中 L 可以是 list、tuple 或者 字符串。

### 迭代Iteration

dict 的迭代：

```
d = {'a': 1, 'b': 2, 'c': 3}
for key in d:
或者
for value in d.values()：
或者
for k, v in d.items()：
```

* 判断一个对象是否可迭代

```
from collections import Iterable
isinstance('abc', Iterable) # str是否可迭代，返回True
```

* Python内置的`enumerate`函数可以把一个list变成索引-元素对，这样就可以在`for`循环中同时迭代索引和元素本身：

```
for i, value in enumerate(['A', 'B', 'C']):
     print(i, value)
```

* for 循环可以同时引用两个变量

```
for x, y in [(1, 1), (2, 4), (3, 9)]:
     print(x, y)
```

### 列表生成式List Comprehensions

用于创建list。

```
[x * x for x in range(1, 11)]
输出：
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

其他用法：

* 带 if 判断：

```
[x * x for x in range(1, 11) if x % 2 == 0]
```

* 双层循环

```
[m + n for m in 'ABC' for n in 'XYZ']
输出
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

列出当前目录下的所有文件和目录：

```
>>> import os # 导入os模块
>>> [d for d in os.listdir('.')] # os.listdir可以列出文件和目录
['.emacs.d', '.ssh', '.Trash', 'Adlm', 'Applications', 'Desktop', 'Documents', 
'Downloads', 'Library', 'Movies', 'Music', 'Pictures', 'Public', 'VirtualBox VMs', 'Workspace', 'XCode']
```

### 生成器Generator

在循环的过程中不断推算出后续的元素。

```
 g = (x * x for x in range(10))
```

next\(g\) 获取下一个值，或者使用 for 循环。

* 另一种写法（斐波那契数列）：

```
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b # 生成器的关键字
        a, b = b, a + b
        n = n + 1
    return 'done'
```

generator的函数在每次调用`next()`的时候执行，遇到`yield`语句返回，再次执行时从上次返回的`yield`语句处继续执行。

用`for`循环调用generator时，发现拿不到 generator 的`return`语句的返回值。如果想要拿到返回值，必须捕获`StopIteration`错误，返回值包含在`StopIteration`的`value`中。

```
g = fib(6)
while True:
     try:
         x = next(g)
         print('g:', x)
     except StopIteration as e:
         print('Generator return value:', e.value)
         break
```

### 迭代器Iterator





