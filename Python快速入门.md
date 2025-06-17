# Python入门

## 1. 数据类型

- Numbers数字
    - int 有符号整数
    - long 长整型
    - float 浮点型
    - complex 复数
```
int : 10,100,-786,080,-0490,-0x260,0x69
long: 51924361L -0x19323L 0122L 0xDEFABCECBDAECBFBAEl 535633629843L  -052318172735L	-4721885298529L
float: 0.0 15.20 -21.9 32.3e+18 -90. -32.54e100 70.2E-12
complex: 3.14j 45.j 9.322e-36j .876j -.6545+0J 3e+26J 4.53e-7j
```   
- String字符串

字符串就是由数字、字母、下划线组成的一串字符

```python
# 从左到右索引从0 - n-1
# 从右开始 -1 - n
s = "a1a2...an" # n>=0
# 如果要实现从字符串中截取的话可以使用[头下标:尾下标]
s = 'abcdef'
s[1:5] #bcde 不包含尾下标的字符
str = 'hello World!'
print str # 输出完整字符串
print str[0] # 输出字符串中的第一个字符
print str[2:5] # 输出字符串中第三至第六个之间的字符串
print str[2:] # 输出从第三个字符开始的字符串
print str * 2 # 输出字符串两次
print str + 'TEST' # 输出连接的字符串
```

- List列表

列表可以完成大多数集合类的数据结构实现,它支持字符,数字,字符串甚至可以包含列表

```python

list = ['test','789',2.23,'join',70.2]
tinylist = [123,'join']

print list # 输出完整列表
print list[0] # 输出列表的第一个元素
print list[1:3] # 输出第二个至第三个元素
print list[2:] # 输出从第三个开始至末尾的所有元素
print tinylist * 2 # 输出列表两次
print list + tinylist # 打印组合的列表
```

- Tuple元祖

元祖用`()`表示,内部元素用逗号隔开,元祖不可以二次赋值,相当于只读列表

```python

tuple = ('test',786,2.23,'join',70.2)
tinytuple = (123,'join')

print tuple # 输出完整元组
print tuple[0] # 输出元组的第一个元素
print tuple[1:3] # 输出元组的第二个至第四个(不包含)的元素
print tuple[2:] # 输出从第三个开始至末尾的元素
print tinytuple * 2 # 输出元组两次
print tuple + tinytuple # 打印组合的元组

```
- Dictionary字典

字典是除列表以外python中最灵活的内置数据结构类型,列表是有序的对象集合,字典是无序的对象集合,字典中的元素通过键来存取,而不是通过偏移

```python
dict = {}
dict['one'] = 'This is one'
dict[2] = "This is two"

tinydict = {'name':'test','code':6734,'dept':'sales'}

print dict['one'] # 输出键'one'的值
print dict[2] # 输出键为2的值
print tinydict # 输出完整的字典
print tinydict.keys() # 输出所有的键
print tinydict.values() # 输出所有的值
```


数据类型之间转化

1. `int(x,[,base])` 整数转化
2. `long(x,[,base])` 长整数转化
3. `float(x)` 浮点数转化
4. `complex(real,[,imag])` 创建复数
5. `str(x)` 将对象x转换为字符串
6. `repr(x)` 将对象x转化为表达式字符串
7. `eval(str)`计算字符串中的有效表达式
8. `tuple(s)`将序列s转化为元组
9. `list(s)`将序列s转化为列表
10. `set(s)`将序列s转化为可变集合
11. `dict(d)`创建字典,d必须是序列(key,value)元组
12. `frozenset(s)`转化为不可变集合
13. `chr(x)`将一个整数转化为一个字符
14. `unichr(x)`将一个整数转化为unicode字符
15. `ord(x)`将一个字符转化为它的整数值
16. `hex(x)`整数转化为十六进制字符串
17. `oct(x)`整数转化为一个八进制字符串

## 2. 运算符

**算数运算符**

**比较运算符**

**赋值运算符**

**位运算符**

**逻辑运算符**

**成员运算符**

**身份运算符**

## 3. 条件语句

## 4. 循环语句

## 5. 字符串

## 6. 列表

## 7. 元组

## 8. 字典

## 9. 日期和时间

## 10. 函数

## 11. 模块

## 12. 文件IO

## 13. File方法

## 14. 异常处理

## 15. OS文件目录

## 16. 内置函数


