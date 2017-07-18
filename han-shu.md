## 函数

### 函数的定义

定义一个函数要使用`def`语句，依次写出函数名、括号、括号中的参数和冒号：。

如果没有`return`语句，函数执行完毕后也会返回结果，只是结果为`None`。

#### 位置参数

power\(x, n\)

#### 默认参数

power\(x, n = 2\)

* 当函数有多个参数时，把变化大的参数放前面，变化小的参数放后面。变化小的参数就可以作为默认参数。

* 也可以不按顺序提供部分默认参数。当不按顺序提供部分默认参数时，需要把参数名写上。比如调用

  `enroll('Adam', 'M', city='Tianjin')`，意思是，`city`参数用传进去的值，其他默认参数继续使用默认值。

* 默认参数必须指向不变对象。

#### 可变参数

calc\(\*nums\)

可以传入任意个参数。在函数内部，参数`nums`接收到的是一个tuple。

* 可以在 list 或 tuple 前面加一个`*`号，把list或tuple的元素变成可变参数传进去。

#### 关键字参数

传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。函数的调用者可以传入任意不受限制的关键字参数。

```
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
```

调用1：

```
person('Michael', 30)
```

调用2：

```
person('Bob', 35, city='Beijing')
输出：
Bob age: 35 other: {'city': 'Beijing'}
```

调用3：

```
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Jack', 24, **extra)
输出：
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

#### 命名关键字参数

如果要限制关键字参数的名字，就可以用命名关键字参数，例如，只接收`city`和`job`作为关键字参数。命名关键字参数必须传入参数名。

```
def person(name, age, *, city, job):
    print(name, age, city, job)
```

* 后面的参数被视为命名关键字参数。

* 如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符`*`了：

```
def person(name, age, *args, city, job):
    print(name, age, args, city, job)
```

* 命名关键字参数可以有默认值，从而简化调用。



