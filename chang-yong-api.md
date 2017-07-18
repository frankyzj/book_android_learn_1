# 常用 API

print\(\)   可以输入多个字符串，整数等。

input\(\): name = input\(\),返回的数据类型是str。

#### 字符串

`ord()`

函数获取字符的整数表示，

`chr()`

函数把编码转换为对应的字符

encode\(\) 、 decode\(\)

`str`通过`encode()`方法可以编码为指定的`bytes:` '中文'.encode\('utf-8'\)。

`len()`

计算`str`的字符数，计算`bytes 的字节数：len('中文') = 3，len('中文'.encode('utf-8')) = 6。`

#### List

classmates.append\('Adam'\)

往 classmates 末尾追加一个元素。

classmates.inset\(1，'Adam'\)

向指定位置插入一个元素。

classmates.pop\(\)

从 classmates 中删除最后一个元素。

classmates.pop\(i\)

删除指定位置的元素。

range\(\)

生成一个整数序列。

#### dict

d.get\('Mike'\)

如果 key 'Mike' 不存在，则返回None，或者 d.get\('Mike', -1\) 返回默认值 -1 。

d.pop\('Bob'\)

从 d 中删除一个键值对。

#### set

s.add\('1'\)

s.remove\('1'\)





