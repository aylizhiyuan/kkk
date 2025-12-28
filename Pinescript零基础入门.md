# Pinscript语言入门

## 1. 脚本的基本结构
---

Pine Script 脚本主要分为两种类型：**指标（Indicator）** 和 **策略（Strategy）**

#### 指标（Indicator）

**要求：**  
必须包含至少一个在图表上生成可视化输出的函数调用，例如：

- `plot()`
- `plotshape()`
- `barcolor()`
- `line.new()`

**特点：**

- 只能用于分析和展示数据
- 不能进行买入或卖出操作



#### 策略（Strategy）

**要求：**  
必须包含至少一个 `strategy.*()` 调用，例如：

- `strategy.entry()`
- `strategy.exit()`

**特点：**

- 必须包含买入或卖出逻辑
- 同时也可以在图表上绘制指标或辅助图形



#### 简单总结

- **指标**：一定要在图表上“画画”，但**不能买入卖出**
- **策略**：一定要有**买入或卖出**，但也**可以画画**

> 如果不涉及交易执行，仅用于分析行情，建议选择 **指标（Indicator）**

---

#### 示例：指标脚本

```pinescript
// This Pine Script® code is subject to the terms of the Mozilla Public License 2.0
// https://mozilla.org/MPL/2.0/
// © lizhiyuan2023

//@version=6
indicator("我的脚本")
plot(close, title="收盘价")
```


#### 示例：策略脚本

```pinescript
// This Pine Script® code is subject to the terms of the Mozilla Public License 2.0
// https://mozilla.org/MPL/2.0/
// © lizhiyuan2023

//@version=6
strategy(
    "我的策略",
    overlay = true,
    fill_orders_on_standard_ohlc = true
)

// 均线金叉做多
longCondition = ta.crossover(
    ta.sma(close, 14),
    ta.sma(close, 28)
)
if longCondition
    strategy.entry("Long", strategy.long)

// 均线死叉做空
shortCondition = ta.crossunder(
    ta.sma(close, 14),
    ta.sma(close, 28)
)
if shortCondition
    strategy.entry("Short", strategy.short)
```


## 2. 函数
---

在 Pine Script 中，函数主要分为两类：

- **内置函数**：Pine 编辑器自带、可以直接调用的函数  
- **自定义函数**：由用户自行定义、用于封装重复逻辑的函数  

**内置函数示例（MACD）**

下面以 MACD 为例，演示内置函数的基本使用流程：

```pinescript

//@version=6
indicator("MACD 2.0 指标")

// 第一步：计算收盘价的 EMA12
ema12 = ta.ema(close, 12)

// 第二步：计算收盘价的 EMA26
ema26 = ta.ema(close, 26)

// 第三步：计算 MACD 快线（EMA12 - EMA26）
fast = ema12 - ema26

// 第四步：计算 MACD 慢线（快线的 EMA9）
slow = ta.ema(fast, 9)

// 第五步：绘制到图表
plot(fast, title = "MACD 快线")
plot(slow, title = "MACD 慢线", color = color.green)
```

**内置函数的常用写法**

```
// 带参数名的标准写法（推荐，易读）
plot(series = close, title = "收盘价", color = color.green)

// 不按参数顺序的命名参数写法
plot(title = "收盘价", series = close, color = color.green)

// 省略可选参数
plot(series = close, color = color.green)

// 混合写法（常见，简洁）
plot(close, "收盘价", color = color.green)

```

**命名空间（Namespace）**

Pine Script 使用**命名空间**对函数进行分类，不同领域的函数位于不同的命名空间中，调用形式一般为：


#### 数学相关函数（`math`）

- `math.abs()`：计算绝对值  
- `math.max()`：返回最大值  
- `math.random()`：生成随机数  
- `math.round_to_mintick()`：按最小变动单位取整  



#### 技术指标相关函数（`ta`）

- `ta.sma()`：简单移动平均  
- `ta.macd()`：MACD  
- `ta.rsi()`：RSI  
- `ta.supertrend()`：SuperTrend  
- `ta.ema()`：指数移动平均  



#### 数据请求相关函数（`request`）

- `request.security()`：请求其他周期或其他交易品种的数据  
- `request.dividends()`：分红数据  
- `request.earnings()`：财报数据  
- `request.financial()`：财务数据  
- `request.quandl()`：Quandl 数据  
- `request.splits()`：拆股数据  


**自定义函数示例**

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


## 3. 脚本的运行逻辑
--- 

### 什么是执行模型？

**执行模型**指的是：  
Pine Script 脚本是**如何在图表上被反复执行的**

你可以先记住一句最重要的话：

> Pine Script 不是一直运行的程序，而是**每一根 K 线都会从第一行重新执行整个脚本**。


### 历史柱（History Bar）

**历史柱**就是已经走完的 K 线

它的特点是：

- 开盘价、最高价、最低价、收盘价、成交量都已经确定  
- 数据不会再发生变化  
- 图表中**除最右边那一根以外的所有 K 线**，都是历史柱  

对于历史柱来说，脚本只会在加载图表时执行一次，结果是稳定的


### 实时柱（Realtime Bar）

**实时柱**是图表最右边那一根，正在形成中的 K 线

它的特点是：

- 价格和成交量会不断变化  
- 还没有真正“收盘”  
- 每次价格变化，脚本都会重新执行  

因此在实时柱中：

- close 表示的是**当前最新价格**
- 而不是最终的收盘价  

这也是为什么指标在最后一根 K 线上会不停跳动


### 指标（Indicator）的执行方式

#### 1. 指标在历史柱上的执行

- 每一根历史 K 线，脚本都会执行一次  
- 执行结果会被保存  
- 不会回头修改  

所以历史部分的指标是稳定不变的


#### 2. 指标在实时柱上的执行（重点）

实时柱中执行逻辑比较特殊：

- K 线刚开始时，执行一次脚本  
- 价格变化一次，就重新执行一次脚本  
- 每次执行前，变量都会被“回滚”  
- 只有在 K 线真正收盘时，结果才会被最终确认  

你看到的现象就是：

- 实时柱的指标一直在变  
- 一旦收盘，就立刻固定下来  


### 五、策略（Strategy）的执行方式

#### 1. 策略在历史柱上的执行

策略在历史柱中的执行方式和指标是一样的：

- 每根历史 K 线执行一次脚本  

不同点在于：  
策略会产生**买入和卖出记录**，用于回测


#### 2. 策略的默认下单逻辑（非常重要）

默认情况下，策略的行为是：

- 当前 K 线满足条件  
- **下一根 K 线的开盘价才真正成交**

也就是说：

- 这一根 K 线负责“判断信号”  
- 下一根 K 线负责“执行买卖”  

这可以避免未来函数问题


#### 3. 策略在实时柱上的执行

默认设置下：

- 实时柱只在**收盘时**执行一次脚本  
- 不会随着价格每一次变动就反复下单  

如果开启“每笔数据上运行”：

- 执行方式会更像指标  
- 行为会更复杂  
- 初学者一般不建议使用  


### 什么是系列值（Series）

**系列值**指的是：  
会随着 K 线一根一根变化的变量。

常见的系列值包括：

- open  
- high  
- low  
- close  
- volume  

这些值：

- 每一根 K 线都有一个  
- 可以通过历史索引访问过去的数据  


### 什么是时间序列（最核心概念）

**时间序列**可以理解为：

> 一串按照时间顺序排列的数据

例如收盘价：

- 当前 K 线一个值  
- 上一根 K 线一个值  
- 再上一根 K 线一个值  

Pine Script 本质上就是在操作这些**随时间变化的数据序列**。


### 小白版重点总结

只记住下面几条就够了:

- Pine Script 是由 K 线驱动执行的  
- 每一根 K 线，脚本都会从头跑一遍  
- 历史柱数据固定，不会变  
- 实时柱数据会变，close 是当前价格  
- 指标只负责画图  
- 策略用于回测，默认下一根 K 线才成交  
- 大多数变量都是时间序列  

> 一句话总结: Pine Script 写的不是“程序”，而是“随 K 线执行的规则”


### 示例

- **引用历史值可以使用`[]`符号**

```
// close[0] 当前K线的收盘价
// close[1] 上一个K线的收盘价
// close[2] 上两跟K线的收盘价

a = close[1] // 创建a变量,变量的值是close[1]上一根K线的收盘价

plot(a, color=color.white)
```

- **时间序列与内置函数的结合**

```
// 对K线的收盘价做一个截止到当前K线的随着时间不断累加的值
b = ta.cum(close)
plot(b,color=color.white)
```

- **连续柱上的函数调用结果也会在时间序列中留下可以使用`[]`**

```
// 从过去10根K线中最大的最高价,包括当前K
c = ta.highest(high, 10)

// 不包括当前K,从上一根 K 线开始算
d = ta.highest(high[1], 10)

// 跟d画出来的效果是一样的
e = ta.highest(high, 10)[1]

plot(c,color=color.white)
plot(d,color=color.green)
plot(e,color=color.rgb(224,235,61))
```

## 4. 书写规则
---
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
---
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
---

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
---

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

---
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
---

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

## 10. 循环

Pine Script **支持循环语句**，但循环的使用方式与传统编程语言不同，具有较多限制，因此在实际开发中需要谨慎使用

---

**`for` 循环（推荐使用）**

`for` 循环用于**循环次数在编译期可确定**的场景，是 Pine Script 中最常用、也是最安全的循环方式


```js
for i = 起始值 to 结束值
    // 循环体
```

```js
//@version=6
indicator("for 循环示例")

float highestHigh = high[0]

for i = 1 to 9
    highestHigh := math.max(highestHigh, high[i])

plot(highestHigh)

```

**while 循环（不推荐）**

Pine Script 支持 while 循环，但限制非常严格：

- 必须保证循环次数有限

- 编译器会检测潜在死循环

- 实际开发中极少使用

