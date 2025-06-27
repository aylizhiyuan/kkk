# Python3入门

## python基础语法

### 编码

默认情况下python3的源码文件以`UTF-8`编码,所有的字符串也是unicode字符串

### 标识符

- 第一个字符必须是以字母(a-z,A-Z)或下划线_

- 标识符的其他的部分由数字、字母、下划线组成

- 标识符对大小写敏感,count和Count是不同的标识符

- 标识符对长度无硬性限制,但建议保持简洁

- 禁止使用保留关键字,如if、for、class等不能作为标识符

### 注释

python中单行注释以`#`开头

多行注释可以使用多个`#`号,或者是`'''`和`"""`


### 缩进

python中使用缩进来表示代码块,并不需要`{}`

```python

if True:
    print("True")
else:
    print("False")    

```

## python数据类型

python中的变量不需要声明,每个变量在使用前都必须赋值,赋值以后该变量才会被创建

`=`用来给变量赋值

左边是变量名,右边是存储在变量中的值

```python
counter = 100 # 整型变量
miles = 1000.0 # 浮点型变量
name = "test" # 字符串


# 多个变量赋值
a = b = c = 1

a,b,c = 1,2,"test"

print(type(a))
print(type(b))
```

- **Number数字**
- **String字符串**
- **bool布尔类型**
- **List列表**
- **Tuple元组**
- **Set集合**
- **Dictionary字典**

### Number数字

`int`,`float`,`bool`,`complex`

- int 整数,不限大小
- float 浮点数,双精度
- complex 复数
- bool 布尔值(子类化自int)

### String字符串


使用单引号`'`或双引号`"`括起来,同时使用反斜杠`\`转义特殊字符

> 变量[头下标:尾下标]

索引值从0开始,-1是末尾的开始位置

```python

str = "Runoob" # 定义一个字符串变量
print(str) # 打印整个字符串
print(str[0:-1]) # 第一个到倒数第二个字符
print(str[0]) # 打印字符串的第一个字符
print(str[2:5]) # 打印字符串第三个到第五个字符
print(str[2:]) # 打印从第三个字符开始到末尾
print(str * 2) # 打印字符串两次
print(str + "TEST") # 打印字符串和TEST拼接

print('Ru\noob') # 回车
```

### Bool布尔类型

布尔类型就是`True`或者`False`

```python

# 布尔类型的值和类型
a = True
b = False
print(type(a)) # <class 'bool'>
print(type(b)) # <class 'bool'>

# 布尔类型的整数表现

print(int(True)) # 1
print(int(False)) # 0

# 使用bool()函数进行转化
print(bool(0)) # False
print(bool(1)) # True
print(bool('')) # False
print(bool('Python')) # True
print(bool([])) # False
print(bool([1,2,3])) # True

# 布尔逻辑运算

print(True and False) # False
print(True or False) # True
print(not True) # False

# 布尔比较运算
print(5 > 3) # True
print(2 == 2) # True
print(7 < 4) # False

```

**0 、空字符串、空列表、空元组**都是False


### List列表

列表是写在方括号之间`[]`、用逗号分隔开的元素列表

和字符串一样,列表同样可以被索引和截取,列表被截取后返回一个包含所需元素的新的列表

```python
list = ['abcd',786,2.23,'runoob',70.2] # 定义一个列表

tinylist = [123,'runoob']

print(list) # 打印整个列表
print(list[0]) # 打印列表的第一个元素
print(list[1:3]) # 打印列表第二 - 第四个元素
print(list[2:]) # 打印列表从第三个元素开始到末尾
print(tinylist * 2) # 打印tinylist列表两次
print(list + tinylist) # 打印两个列表拼接在一起的结果

# 列表中的元素是可以修改的
a = [1,2,3,4,5,6]
a[2:5] = [13,14,15]
```



### Tuple元组

元组跟列表类似,不同之处在于元组的元素不能修改,元组写在`()`里,元素用逗号隔开,元素类型也可以不相同

```python

tuple = ('abcd',786,2.23,'runoob',70.2)
tinytuple = (123,'runoob')

print(tuple) # 输出完整的元组
print(tuple[0]) # 输出元组的第一个元素
print(tuple[1:3]) # 输出元组的第二个元素开始到第三个元素
print(tuple[2:]) # 输出从第三个元素开始的所有元素
print(tinytuple * 2) # 输出两次元组
print(tuple + tinytuple) # 连接元组
```


### Set集合

集合是一种无序、可变的数据类型,用于存储唯一的元素.集合中的元素不会重复,可以进行交集、并集、差集等常见的集合操作

在python中,集合用大括号`{}`表示,元素之间用`,`分隔

```python

sites = {'Google','Taobao','Runoob','Facebook','Zhihu','Baidu'}

print(sites) # 输出集合,重复的元素被自动去掉


# set可以进行集合运算

a = set('abracadabra')
b = set('alacazam')

print(a - b) # a和b的差集

print(a | b) # a和b的并集

print(a & b) # a和b的交集

print(a ^ b) # a和b中不同时存在的元素
```

### Dictionary字典

字典是一种映射类型,字典使用`{}`标识,它是一个无序的`键:值`集合

```python
dict = {}
dict['one'] = "1 - 克制"
dict[2] = "2 - 克制"
tinydict = {'name':'runoob','code':1,'site':'runoob'}

print(dict['one']) # 输出键为one的值
print(dict[2]) # 输出键位2的值
print(tinydict) # 输出完整的字典
print(tinydict.keys()) # 输出所有键
print(tinydict.values) # 输出所有值

```


## python数据类型转化

- 隐式转化 ,自动完成
- 显式类型转化,需要使用类型函数来转换

```python

num_int = 123

num_flo = 1.23

num_new = num_int + num_flo

print("num_new 值为:",num_new)
print("num_new 数据类型为:",type(num_new)) # float,两种不同类型的数据进行运算,较低的就会转化为较高的数据类型以避免数据丢失

num_int = 123
num_str = '456'

print(num_int + num_str) # 无法进行隐式转换,需要进行显式转换

```

```python

x = int(1) # x的输出结果为1
y = int(2.8) # y的输出结果为2
z = int("3") # z的输出结果为3

x = float(1)     # x 输出结果为 1.0
y = float(2.8)   # y 输出结果为 2.8
z = float("3")   # z 输出结果为 3.0
w = float("4.2") # w 输出结果为 4.2

x = str("s1") # x 输出结果为 's1'
y = str(2)    # y 输出结果为 '2'
z = str(3.0)  # z 输出结果为 '3.0'
```

## python运算符


### 算数运算符

```python

a = 21
b = 10
c = 0

c = a + b
print ("1 - c 的值为：", c)
 
c = a - b
print ("2 - c 的值为：", c)
 
c = a * b
print ("3 - c 的值为：", c)
 
c = a / b
print ("4 - c 的值为：", c)
 
c = a % b
print ("5 - c 的值为：", c)
 
# 修改变量 a 、b 、c
a = 2
b = 3
c = a**b 
print ("6 - c 的值为：", c) # a的b次方
 
a = 10
b = 5
c = a//b 
print ("7 - c 的值为：", c) # 取整除

```

### 比较运算符

```python

a = 21
b = 10
c = 0
 
if ( a == b ):
   print ("1 - a 等于 b")
else:
   print ("1 - a 不等于 b")
 
if ( a != b ):
   print ("2 - a 不等于 b")
else:
   print ("2 - a 等于 b")
 
if ( a < b ):
   print ("3 - a 小于 b")
else:
   print ("3 - a 大于等于 b")
 
if ( a > b ):
   print ("4 - a 大于 b")
else:
   print ("4 - a 小于等于 b")
 
# 修改变量 a 和 b 的值
a = 5
b = 20
if ( a <= b ):
   print ("5 - a 小于等于 b")
else:
   print ("5 - a 大于  b")
 
if ( b >= a ):
   print ("6 - b 大于等于 a")
else:
   print ("6 - b 小于 a")

```

### 赋值运算符

```python
a = 21
b = 10
c = 0
 
c = a + b
print ("1 - c 的值为：", c)
 
c += a
print ("2 - c 的值为：", c)
 
c *= a
print ("3 - c 的值为：", c)
 
c /= a 
print ("4 - c 的值为：", c)
 
c = 2
c %= a
print ("5 - c 的值为：", c)
 
c **= a
print ("6 - c 的值为：", c)
 
c //= a
print ("7 - c 的值为：", c)


# 海象运算符

# 传统写法
n = 10
if n > 5:
    print(n)

# 使用海象运算符
if (n := 10) > 5:
    print(n)

```

### python位运算

```python
a = 60            # 60 = 0011 1100 
b = 13            # 13 = 0000 1101 
c = 0
 
c = a & b        # 12 = 0000 1100
print ("1 - c 的值为：", c)
 
c = a | b        # 61 = 0011 1101 
print ("2 - c 的值为：", c)
 
c = a ^ b        # 49 = 0011 0001
print ("3 - c 的值为：", c)
 
c = ~a           # -61 = 1100 0011
print ("4 - c 的值为：", c)
 
c = a << 2       # 240 = 1111 0000
print ("5 - c 的值为：", c)
 
c = a >> 2       # 15 = 0000 1111
print ("6 - c 的值为：", c)

```

### python逻辑运算符

```python

a = 10
b = 20
 
if ( a and b ):
   print ("1 - 变量 a 和 b 都为 true")
else:
   print ("1 - 变量 a 和 b 有一个不为 true")
 
if ( a or b ):
   print ("2 - 变量 a 和 b 都为 true，或其中一个变量为 true")
else:
   print ("2 - 变量 a 和 b 都不为 true")
 
# 修改变量 a 的值
a = 0
if ( a and b ):
   print ("3 - 变量 a 和 b 都为 true")
else:
   print ("3 - 变量 a 和 b 有一个不为 true")
 
if ( a or b ):
   print ("4 - 变量 a 和 b 都为 true，或其中一个变量为 true")
else:
   print ("4 - 变量 a 和 b 都不为 true")
 
if not( a and b ):
   print ("5 - 变量 a 和 b 都为 false，或其中一个变量为 false")
else:
   print ("5 - 变量 a 和 b 都为 true")

```



### python成员运算符

```python
a = 10
b = 20
list = [1, 2, 3, 4, 5 ]
 
if ( a in list ):
   print ("1 - 变量 a 在给定的列表中 list 中")
else:
   print ("1 - 变量 a 不在给定的列表中 list 中")
 
if ( b not in list ):
   print ("2 - 变量 b 不在给定的列表中 list 中")
else:
   print ("2 - 变量 b 在给定的列表中 list 中")
 
# 修改变量 a 的值
a = 2
if ( a in list ):
   print ("3 - 变量 a 在给定的列表中 list 中")
else:
   print ("3 - 变量 a 不在给定的列表中 list 中")

```

### python身份运算符

```python

a = 20
b = 20
 
if ( a is b ):
   print ("1 - a 和 b 有相同的标识")
else:
   print ("1 - a 和 b 没有相同的标识")
 
if ( id(a) == id(b) ):
   print ("2 - a 和 b 有相同的标识")
else:
   print ("2 - a 和 b 没有相同的标识")
 
# 修改变量 b 的值
b = 30
if ( a is b ):
   print ("3 - a 和 b 有相同的标识")
else:
   print ("3 - a 和 b 没有相同的标识")
 
if ( a is not b ):
   print ("4 - a 和 b 没有相同的标识")
else:
   print ("4 - a 和 b 有相同的标识")
```

## python条件控制

```python
# if判断
age = int(input("请输入你家狗狗的年龄: "))
print("")
if age <= 0:
    print("你是在逗我吧!")
elif age == 1:
    print("相当于 14 岁的人。")
elif age == 2:
    print("相当于 22 岁的人。")
elif age > 2:
    human = 22 + (age -2)*5
    print("对应人类年龄: ", human)

```

## python循环语句

```python
n = 100
sum = 0
counter = 1
while counter <= n:
   sum = sum + counter
   counter += 1
print("1 到 %d 之和为: %d",(n,sum))

# 列表循环
sites = ["Baidu", "Google","Runoob","Taobao"]
for site in sites:
    print(site)
# 字符串循环
word = 'runoob'
 for letter in word:
    print(letter) 
# 计数循环
for number in range(1,6):
   print(number)

# for....else
for x in range(6):
  print(x)
else:
  print("Finally finished!")

# for....break
sites = ["Baidu", "Google","Runoob","Taobao"]
for site in sites:
    if site == "Runoob":
        print("教程!")
        break
    print("循环数据 " + site)
else:
    print("没有循环数据!")
print("完成循环!")  

# while...break 跳出循环
n = 5
while n > 0:
    n -= 1
    if n == 2:
        break
    print(n)
print('循环结束。')

# while...continue
n = 5
while n > 0:
    n -= 1
    if n == 2:
        continue
    print(n)
print('循环结束。')

# for....break
for letter in 'Runoob':
   if letter == 'b':
      break
   print('当前字母为:',letter)   

# for...continue
for letter in 'Runoob':     # 第一个实例
   if letter == 'o':        # 字母为 o 时跳过输出
      continue
   print ('当前字母 :', letter)

```

## python迭代器

## python with

## python函数

## python装饰器

## python数据结构

## python模块

## python name

## python输入和输出

## python file

## python OS

## python错误和异常

## python面对对象

## python命名空间/作用域

## python标准库概览