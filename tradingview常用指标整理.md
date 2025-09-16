## tradingview常用指标整理

- 散户视角
    - 趋势类 EMA/SMA/WMA/ADX 判断价格趋势方向/强弱
    - 动能类 MACD/RSI/Stochastic 判断价格涨跌强弱、拐点
    - 波动类 ATR/Bollinger Bands 看价格波动范围
    - 成交量类 OBV/Volume MA 看资金进出强弱
    - 形态类 ZigZag 识别图表形态

- 机构视角
    - BOS/Choch 趋势反转确认
    - OB 主力建仓区域
    - Buy/Sell side liquidity 主力吸单方向
    - FVG 价格不平衡区域,易回补

### 一. 移动平均线 MA/EMA/SMA

**ADX**

`ADX`使用每根K线的最高价和最低价来计算波动幅度，`RSI`使用的是价格的收盘价的涨幅和跌幅来看近期上涨和下跌的速度或者动量

```javascript
//@version=5
indicator("手动实现 ADX", overlay=false)

// === 参数 ===
length = input.int(14, "ADX周期")

// === 第一步：计算 TR, +DM, -DM ===
up   = ta.change(high)
down = -ta.change(low)
// 向上变动
plusDM  = (up > down and up > 0)   ? up   : 0
// 向下变动
minusDM = (down > up and down > 0) ? down : 0
// 价格波动范围,true range
tr = ta.tr

// === 第二步：对 TR, +DM, -DM 进行 Wilder 平滑 ===
smoothedTR     = ta.rma(tr, length)
smoothedPlusDM = ta.rma(plusDM, length)
smoothedMinusDM = ta.rma(minusDM, length)

// === 第三步：计算 +DI, -DI ===
plusDI  = 100 * smoothedPlusDM / smoothedTR
minusDI = 100 * smoothedMinusDM / smoothedTR

// === 第四步：计算 DX ===
dx = 100 * math.abs(plusDI - minusDI) / (plusDI + minusDI)

// === 第五步：计算 ADX（对 DX 再做一次平滑）===
adx = ta.rma(dx, length)

// === 画出 ADX 线和 +DI, -DI ===
plot(adx,      title="ADX",      color=color.orange, linewidth=2)
plot(plusDI,   title="+DI",      color=color.green)
plot(minusDI,  title="-DI",      color=color.red)
hline(25,      "趋势强度分界线", color=color.gray, linestyle=hline.style_dotted)

```


**SMA**

Simple Moving Average = (a1+a2+a3...+an) / n 


**EMA**

- 计算平滑系数,当前价格所占的比重吧

α = 2 / (10 + 1) = 0.1818, 10代表的是10EMA

- 计算10日EMA, 1 - α 是前一日所占的比重

10EMA = α x 当前收盘价格 + (1 - α) x 前一日EMA

看起来只计算了前一根的EMA,但是前一个EMA也是依靠上上根的EMA计算的,每次只用前一根K线的EMA，但其实隐含了所有历史的痕迹，只是权重递减



**多周期移动平均线EMA21/55**

```javascript
//@version=6
indicator("低周期图中显示日线EMA21/55 + 1小时EMA7", overlay=true)

// === 参数 ===
show_ema21d  = input.bool(true, "显示日线 EMA 21", group="EMA 设置")
show_ema55d  = input.bool(true, "显示日线 EMA 55", group="EMA 设置")

// === 请求不同周期的 EMA ===
get_ema(tf, len) =>
    request.security(syminfo.tickerid, tf, ta.ema(close, len))

// 计算 EMA 值
ema21_day = get_ema("D", 21)
ema55_day = get_ema("D", 55)

// === 是否是5分钟图？
is_5min = timeframe.period == "5"

// === 绘制 ===
plot(show_ema21d and not is_5min ? ema21_day : na, title="日线 EMA21", color=color.blue,   linewidth=1)
plot(show_ema55d and not is_5min ? ema55_day : na, title="日线 EMA55", color=color.green,  linewidth=1)

```


### 二、价格涨跌速度

**MACD**

MACD = 快慢均线的距离

快速EMA(21)
慢速EMA(55)

DIF线 = 快EMA - 慢EMA = 它们的位置偏离 = 均线开口的高度差

DEA线 = DIF的9日EMA(平滑后的偏离趋势) = 均值,偏离的平均水平

MACD柱状图 = DIF - DEA(动能增减的体现) = 反应这个开口相对于平均趋势是在扩大还是缩小 = 当前开口情况

背离情况: 价格在上涨，但上涨的速度越来越慢，MACD柱状图逐渐缩短（正值变小），动能减弱，形成MACD顶背离;价格在下跌，但下跌的速度越来越慢，MACD柱状图逐渐缩短（负值变小），动能衰竭，形成MACD底背离。

**RSI**

RSI = 价格上涨下跌力度的强弱 = 通过计算一定周期内上涨和下跌的平均幅度，得出一个 0 到 100 之间的数值

假设周期是N,涨幅为正,下跌为0,跌幅为正,上涨为0

平均上涨幅度 = N天内所有上涨日的涨幅涨幅之和 / N

平均下跌幅度 = N天内所有下跌日的跌幅之和 / N 

RS = 平均上涨幅度  / 平均下跌幅度

RS大的话,说明上涨 > 下跌; RS很小的话,下跌 > 上涨

RS本质上是一个0到正无穷的数值,RSI把RS映射到0-100之间,形成一个标准化的指标

RSI = 100 - (100 / 1 + RS)

顶背离情况: 价格创新高，而RSI未创新高，预示上涨动力减弱，可能出现下跌

底背离情况: 价格创新低，而RSI未创新低，表明下跌动能减弱，可能反转上涨

### 三、价格波动范围

**BOLLING**

Bolling = 价格波动幅度 = 价格默认会在这个布林带中波动

中轨线 = 最近N天的简单平均价(SMA)

上轨线 = 中轨 + 2 x 标准差

下轨线 = 中轨 - 2 x 标准差

\[
\sigma = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (x_i - \mu)^2}
\]

(收盘价 - 平均值)^2 → 负数变正 + 强调大波动 → 得到 a1, a2, a3...

a1 + a2 + a3 ... / n → 平均这些偏离的平方值

最后开根号 → 得出标准差（衡量波动有多大）

常用设置:

- 短期交易 Length: 10,标准差的乘积: 1.5
- 长期交易 Length: 50,标准差的乘积: 2.5


**ATR**

ATR = 价格波动的强度 = 数字表示 = 某一天大了说明波动大了

-  假设周期是N(包括当前K线在内的N根K线),首先计算TR,TR 是衡量每天实际波动的范围，考虑跳空等情况，取三者中的最大值,最终算出N根K线的TR的值
\[
\text{TR} = \max \left( 
\begin{array}{l}
\text{High}_t - \text{Low}_t \\
|\text{High}_t - \text{Close}_{t-1}| \\
|\text{Low}_t - \text{Close}_{t-1}|
\end{array}
\right)
\]

- 假设你已经计算出每根K线的 TR 值：
    - TR1：当前K线的真实波幅

    - TR2：上一根K线的真实波幅
    - TR3：上上一根K线的真实波幅
    - 以此类推...

- 计算第一个 ATR（第10根K线）

\[
ATR_{10} = \frac{TR_1 + TR_2 + \cdots + TR_{10}}{10}
\]

> 用前10根K线的TR值简单平均，得到第一个ATR值。

-  递推计算后续ATR（第11根及以后）对于 \( t > 10 \):

\[
ATR_t = \frac{(ATR_{t-1} \times 9) + TR_t}{10}
\]

| K线序号 \(t\) | TR 值       | ATR 计算公式                         |
|---------------|-------------|-------------------------------------|
| 1 ~ 9         | 已计算的TR值 | 无ATR，等待凑满10根数据             |
| 10            | TR_10       | \(ATR_{10} = \frac{\sum_{i=1}^{10} TR_i}{10}\) |
| 11            | TR_11       | \(ATR_{11} = \frac{9 \times ATR_{10} + TR_{11}}{10}\) |
| 12            | TR_12       | \(ATR_{12} = \frac{9 \times ATR_{11} + TR_{12}}{10}\) |
| ...           | ...         | 持续递推计算                         |

这样，你可以从第10根K线开始，拿到第一个ATR值，然后用递推公式计算之后所有ATR

这也意味着我如果想知道当前K线的波动,那么必须等待10根K线后才可以

### 四、成交量


### 五、形态


### 六. 聪明钱SMC

#### 1. 概念

订单块 = 庄家成本区 

他们会买一部分的货,在这波拉升这里会形成一个订单块,这里就是庄家的成本区间,所以,当价格再次回到这个订单块,也就是庄家的成本区的时候,价格大概率就会在这个订单块得到反弹以及拉升

趋势突破 = Break of Structure(BOS),价格突破前高,形成顶顶高的上升趋势架构,这意味着上升趋势的延续,后市看涨;价格突破前低形成底底低的下跌趋势架构,这意味着下跌趋势的延续,后市看跌


趋势破坏 = Change of Character(Choch)价钱跌破前低形成底底低的下跌趋势架构,这意味着上升趋势大概率转为下跌趋势;价钱突破前高形成顶顶高的上升趋势架构,这意味着大概率转为上升趋势

**大阳线大概率才可以证明是庄家和机构的介入**


流动性 = 钱 = 庄家止损散户的区域


支撑位附近的流动性 = 做多的止损单 + 做空的突破单

一旦砸破支撑位,会被触发大量的卖单(散户)以及大量的买单(聪明钱)

压力位附近的流动性 = 做多的突破单 + 做空的止损单

一旦上涨突破压力位,会触发大量的买单(散户)以及大量的卖单(聪明钱)

**常见的支撑阻力就是陷阱**

价格在震荡区间内盘整，其上下边缘（等高点/等低点）积累了大量的止损和突破挂单。主力常利用这种区间制造双向假突破，分别扫掉 Buy Side 与 Sell Side 流动性，完成对手盘收集，之后再选择真实方向进行趋势推动。







 

