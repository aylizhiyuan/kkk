# 量化入门 - tradingView

## 1. 脚本的基本结构

**指标:** 必须包含至少一个在图表上生成输出的函数调用,例如`plot()`,`plotshape()`,`barcolor()`,`line.new()`

**策略:** 必须包含至少一个strategy.*()调用,例如`stragey.entry()`

简单总结:

- `指标`一定要在图表上画画,但是不能买入卖出

- `策略`一定要有买入或者卖出,但是也可以画画

> 看情况选择指标或者策略,不需要买卖数据的情况下选择指标


```
// 脚本 - 指标
// This Pine Script® code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © lizhiyuan2023

//@version=6
indicator("我的脚本")
plot(close)
```

```
// 脚本 - 策略
// This Pine Script® code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © lizhiyuan2023

//@version=6
strategy("我的策略", overlay=true, fill_orders_on_standard_ohlc = true)

longCondition = ta.crossover(ta.sma(close, 14), ta.sma(close, 28))
if (longCondition)
    strategy.entry("My Long Entry Id", strategy.long)

shortCondition = ta.crossunder(ta.sma(close, 14), ta.sma(close, 28))
if (shortCondition)
    strategy.entry("My Short Entry Id", strategy.short)
```

在`打开脚本`地方查看自己的指标和策略,制作完成后发布即可


## 2. 函数

- 内置函数: Pine编辑器自带的可调用函数

- 自定义函数


```

// 内置函数示例
indicator("MACD2.0指标")
// 第一步计算出收盘价的ema12
ema12 = ta.ema(close,12)
// 第二步计算出收盘价的ema26
ema26 = ta.ema(close,26)
// 第三步计算出macd的值(ema12 - ema16)
fast = ema12 - ema26
// 第四步计算出macd的指数平均值
slow = ta.ema(fast,9)
// 第五步画出来
plot(fast) // 画出快线
plot(slow,color=color.green) // 画出慢线

```

```
// 内置函数的写法举例
// 带参数的标准写法
plot(series=close,title='收盘价',color=color.green)
// 不按顺序的标准写法
plot(title='收盘价',series=close,color=color.green)
// 可以省略可选参数
plot(series=close,color=color.green)
// 混合写法
plot(close,'收盘价',color=color.green)
```

**命名空间**

- 数学相关的函数math: `math.abs()`,`math.max()`,`math.random()`,`math.round_to_mintick()`

- 技术指标ta: `ta.sma()`,`ta.macd()`,`ta.rsi()`,`ta.supertrend()`等

- 其他交易品种或时间范围请求数据的函数: `request.dividends()`,`request.earnings()`,`request.financial()`,`request.quandl()`,`request.security()`,`request.splits()`


## 3. 脚本的运行逻辑

**执行模型:** 脚本如何在图表上执行的

**历史柱:** 高/开/低/收/成交量已经确定了,并且不会再产生变化的K柱

**实时柱:** 目前正在变化的K线柱(最右的一根K柱)

**指标在历史柱的执行模型:** 每根历史柱上运行一次脚本

**指标在实时柱上的执行模型:** 脚本在实时柱开盘的时候执行,然后每次更新(即价格或成交量变化)时执行一次,在每次实时更新之前回滚变量,变量在结束栏更新时提交一次(收盘的时候回滚并执行)

你可以理解为`close`就是当前价格,而不是收盘价,因此价格现在在不断变化中....它还没有收盘


**策略在历史柱中的执行模型:** 跟指标的执行模型是一样的,策略的`买入`和`卖出`都是在满足条件后的下一个K线的开盘价位置执行,可以选择`在k线关闭时`则可以直接在当前K线中执行;

**策略在实时柱上的执行模型:** 策略默认是实时柱关闭的时候运行一次;`每笔数据上`会变跟指标一样

> 默认的策略执行模型: 实时柱关闭后运行一次脚本,在满足条件后的下一个K线的开盘价位置执行买入卖出

**系列值:** 值随着K线变化的变量,例如close,open

**时间序列:** 变量随着时间变化而变化的基本结构

- 引用历史值可以使用`[]`符号

```
// close[0] 当前K线的收盘价
// close[1] 上一个K线的收盘价
// close[2] 上两跟K线的收盘价

a = close[1] // 创建a变量,变量的值是close[1]上一根K线的收盘价

plot(a, color=color.white)
```

- 时间序列与内置函数的结合

```
// 对K线的收盘价做一个截止到当前K线的随着时间不断累加的值
b = ta.cum(close)
plot(b,color=color.white)
```

- 连续柱上的函数调用结果也会在时间序列中留下可以使用`[]`

```
// 从过去10根K线中最大的最高价
c = ta.highest(high,10)
// 从过去10根K线中最大的最高价的前一个K线的最高值
d = ta.highest(high[1],10)
// 跟d画出来的效果是一样的,实际就是c[1]
e = ta.highest(high,10)[1]

plot(c,color=color.white)
plot(d,color=color.green)
plot(e,color=color.rgb(224,235,61))
```

## 4. 书写规则

**命名约定**

- 变量或者函数名称使用驼峰式: maFast
- 常量大写: FROFIT_COLOR, STOP_COMMENT


**间距规则**

除了一元运算符`()`以外,所有的运算符的两边应该都是有空格的,在所有的`逗号`之后和使用命名函数参数的时候,也建议使用p'
```
plot(series = close, color = color.red)
```

**换行规则**

根据实际情况可将函数参数进行一些换行优化

## 5. 变量

```javascript
var string name1 = '测试' // 可以省略声明/数据类型
name2 = '测试' // 可以是文字
name3 = name2 // 可以是变量
age = 18 + 1 // 可以是表达式
age = if 1 > 0 
    18 // 可以是if/for/while/switch结构
size = ta.rsi(close,7) // 可以是函数调用
// 多个变量的赋值
[d,f,g] = ta.bb(close,20,2)

// var声明的变量是全局变量,变量仅初始化一次
// 适用于长期存放累加的数据,并不需要实时的计算
var data = 0
data := data + 1 // 重新赋值的写法

// 不加var的情况下
data = 0
data := data + 1 // 每次脚本执行的时候,data都会等于0
```


## 6. 数据类型

**[限定类型] - [变量类型]**

**限定类型**

- const: 适用于不随时间变化的常量值
- input: 让用户通过设置面板自定义脚本参数
- simple: 只会在脚本初始化阶段执行一次,不会随K线每一根更新而更新
- series: 变量的值可以随着每一跟K线而更新,如果你不写修饰符,变量就是series

> const < input < simple < series 

```
ma = ta.sma(close,14) // 默认就是series类型
simple initClose = close // 加载图表的时候只记录那一时刻的收盘价,然后一直维持不变
```

**变量类型**

- 基本类型: int float bool color string
- 特殊类型: plot(绘图) hline(水平线) line(线) linefill(填充线) box(方框) polyline(折线) label(表格) table(图表) chart.point(点) array(数组) matrix(矩阵) map
- 自定义类型: type 关键字允许建立使用者定义的类型
- 4na(空白) void类型

```javascript
a = 10  // 默认类型判断是const int

a = 1.5 // const float 

a = "hello" // const string 

a = close // 默认是series float类型,随着K线更新

a = ta.sma(close,14) // series float

c = input(b * 2, 'input:c') // input

d = input.source(close,'标题') // series 

```

## 7. 运算符

- 算数运算符 + - * / %
    - +可以用来字符串相连
    - float 和int运算,结果为float
    - 如果有一个值是na,结果也是na
- 比较运算符
    - <  小于
    - <= 小于等于
    - !=  
    - == 
    - `>` 
    - `>=`    
- 逻辑运算符
    - not 
    - and  
    - or
- 三元运算符
- 历史引用运算符
    - 可以使用[]历史引用运算符来引用时间序列的过去值        

## 8. if条件

```javascript

if a - 1
    c:= color.red
else if (b)
    c:= color.green
else if close < high
    c:= color.rgb(6,221,249)
else
    c:= color.rgb(233,243,33)
```

## 9. switch条件

```javascript

float ma = switch maType
    "EMA" => ta.ema(close,maLength)
    "SMA" => ta.sma(close,maLength)
    "RMA" => ta.rma(close,maLength)
    "WMA" => ta.wma(close,maLength)

switch
    close > open => color_data = color.green
    close < high => color_data = color.yellow
```

## 10. 自定义函数

```javascript

f1(x,y) => x + y // 单行函数

a = f1(1,1)
b = f1(close,open)
c = f1(open,1)

f2(m,n) =>
    a = m + n 
    b = a + 1
    b // 返回值

d = f2(a,2)

```












