#### 使用 \_\_slots\_\_

使用 \_\_slots\_\_ 变量，可以限制一个 class 实例能添加的变量。

```
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

* 使用 \_\_slots\_\_  定义的属性仅对当前类起作用，对子类不起作用。



