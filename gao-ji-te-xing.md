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



