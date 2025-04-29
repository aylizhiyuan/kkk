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

## 5. 变量

## 6. 数据系统

## 7. 运算符

## 8. if条件

## 9. switch条件

## 10. 实战













