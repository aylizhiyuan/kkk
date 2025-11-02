## tradingview常用指标整理

- 散户视角
    - 趋势类 EMA/SMA/WMA/ADX 判断价格趋势方向/强弱
    - 动能类 MACD/RSI/Stochastic 判断价格涨跌强弱、拐点
    - 波动类 ATR/Bollinger Bands/Super Trend 看价格波动范围
    - 成交量类 OBV/Volume MA 看资金进出强弱
    - 形态类 ZigZag 识别图表形态
- 机构视角
    - BOS/Choch 趋势反转确认
    - OB 主力建仓区域
    - Buy/Sell side liquidity 主力吸单方向
    - FVG 价格不平衡区域,易回补

### 一. 移动平均线 MA/EMA/SMA

**ADX**

主要用于判断单边行情的趋势强弱以及震荡 -- 本质就是趋势持续的时间,长期上涨的话,趋势就变强

- 顺势策略：ADX 高 → 趋势强 → 可以跟随方向操作，ADX减弱 、方向变化时候出场
- 震荡策略：ADX 低 → 趋势弱 → 可以考虑区间操作或等待趋势启动

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

**随机RSI**

随机RSI就是把RSI当做价格，再套用随机指标的计算方式，比较RSI在过去N根中的位置，它告诉你RSI在自己的历史区间是偏高还是偏低

例子:

- RSI = 65，看起来不算超买
- 但是如果过去14根RSI的最高是66，最低是40，那么65已经非常接近最高点了
- 随机RSI就会显示接近1，提醒你RSI已经在区间顶部了

Raw StochRSI  = RSI - min(RSI,N) / max(RSI,N) - min(RSI,N)

min(RSI,N) = 最近N根RSI的最小值
max(RSI,N) = 最近N根RSI的最大值

StochRSI=SMA(Raw StochRSI,smoothK)



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

**ATR = 过去 n 根 K 平均每根波动多少点 = 数字表示 = 某一天大了说明波动大了**

- 先计算TR,取一下三者中的最大值:

TR = max(当日最高价 - 当日最低价, |当日最高价 - 昨日收盘价|, |当日最低价 - 昨日收盘价|)

- 计算N周期的平均值:

ATR = SMA(TR,N)

TR衡量单根K线的真实波动

ATR是TR的平均,表示近期价格的平均波动幅度



**Super Trend**

根据价格和市场波动来计算出上轨和下轨，当价格突破这些轨道的时候，趋势被认为发生了反转

主要作用：判断当前市场是多头趋势还是空头趋势

- 计算ATR：ATR越大，市场的波动越大；ATR越小，市场越平稳

- 计算上下轨：上轨 = 中心价格 + ATR x 系数；下轨 = 中心价格 + ATR x 系数

当价格 > 前一跟K上轨 = 上涨 , 当价格 < 前一根K下轨 = 下跌


### 四、成交量

在Tradingview中原生数据只有`volume`,所有的量能指标都是基于`volume`计算的


**成交量爆量**


```js
// === 成交量多级放量 ===
vol_ma = ta.sma(volume, 20)
vol_mild    = volume > vol_ma * 1.2 and volume <= vol_ma * 1.5
vol_strong  = volume > vol_ma * 1.5 and volume <= vol_ma * 2.0
vol_extreme = volume > vol_ma * 2.0

// === 参数 ===
bars_lookahead = 5          // 跟踪未来5根
follow_vol_mult = 1.0       // 跟随K线的放量标准（>均量即可）
follow_vol_required = 2     // 至少2根跟随放量

// === 状态变量 ===
var bool track_follow = false        // 是否正在跟踪中
var int bars_since_break = 0         // 已经过去的K线数
var int follow_vol_count = 0         // 跟随放量的数量
var bool has_following_vol = false   // 是否确认出现跟随放量

// === 主放量触发 ===
is_main_break = volume > vol_ma * 1.2

// === 核心逻辑：else if 避免主放量K线被统计 ===
if is_main_break
    // 只启动跟踪，不立即统计
    track_follow := true
    bars_since_break := 0
    follow_vol_count := 0
    has_following_vol := false

else if track_follow
    // 进入跟踪逻辑
    bars_since_break += 1

    // 检查当前K线是否放量
    if volume > vol_ma * follow_vol_mult
        follow_vol_count += 1

    // 若5根内出现至少2根放量，则确认成功
    if follow_vol_count >= follow_vol_required
        has_following_vol := true // 展示连续放量标签
    // 超过5根取消监控并去除标签
    if bars_since_break > bars_lookahead
        track_follow := false // 放弃统计并重新进行爆量K的统计
        has_following_vol := false  // 去除标签的展示

// === 可视化标识 ===
vol_symbol =
     vol_extreme       ? "🚀" :
     vol_strong        ? "💥" :
     vol_mild          ? "🟢" :
                         "⚪️"
```






### 五、形态

**zigzag**

```javascript
zz(length) => 
    [dir,p_h,p_l] = pivot(length)
    // 检测当前K线的方向是否跟上一根方向不同
    // 0 表示不变,2方向反转向上，-2 方向反转向下
    dirchanged = ta.change(dir)
    if p_h or p_l
        add_to_zz(dir,dirchanged,p_h,p_l,bar_index)
zz(period)        
```

- 每根bar调用pivot(period),判断当前是否为pivot(高/低)

- dirchange使用`ta.change(dir)`只反应当前K线与前一根K线的变化幅度

- 如果检测到p_h或p_l(存在更高的高点，低点)就调用add_to_zz把点加入到zz数组结构中


```js
pivot(period) =>
    // 在多跟K线中，它是否是最高价格的那根,如果是保存，否则为na
    float p_h = ta.highestbars(high,period) == 0 ? high : na
    float p_l = ta.lowestbars(low,period) == 0 ? low : na
    dir = 0
    // 如果当前是低点的话,得到-1,否则继承上一跟dir[1]
    iff_1 = p_l and na(p_h) ? -1 : dir[1]
    // 如果当前是高点,dir = 1
    dir := p_h and na(p_l) ? 1 : iff_1
    [dir,p_h,p_l]
```


```js
// 价格，索引，方向
add_to_array(value,index,dir) =>
    // 检查当前高点是否比上一个高点更高，低点是否比上一个低点低
    // 用 mult 标记当前点是“新高/新低”（权重 2），还是普通 pivot（权重 1）,就是转折点
    mult = array.size(ZZvalues) < 2 ? 1 : dir * value > dir * array.get(ZZvalues, 1) ? 2 : 1
    array.unshift(ZZindexes, index) // 插头K线索引
    array.unshift(ZZvalues, value) // 插头K线价格
    array.unshift(ZZdir, dir * mult) // 插头K线的方向（-1/+1）* 权重(2/1)
    // 超过最大数组长度 → 删除最旧的点
    if array.size(ZZindexes) > max_array_size
        array.pop(ZZindexes)
        array.pop(ZZvalues)
        array.pop(ZZdir)
```


```js
// 当前K线的方向、是否反转、高点、低点、K线索引
add_to_zz(dir, dirchanged, p_h, p_l, index) =>
    // 找到有效的高点或者低点
    value = dir == 1 ? p_h : p_l
    // 假设新增的高点低点方向发生了反转
    if array.size(ZZvalues) == 0 or dirchanged
        add_to_array(value, index, dir)
    // 同一个方向，但是出现了新高或者新低 
    // 删除数组头部元素（最最近的 pivot 点）
    // 替换原来的 pivot 点   
    else if dir == 1 and value > array.get(ZZvalues, 0) or dir == -1 and value < array.get(ZZvalues, 0)
        array.shift(ZZvalues)
        array.shift(ZZindexes)
        array.shift(ZZdir)
        add_to_array(value, index, dir)
```

```js

if barstate.isconfirmed and array.size(ZZindexes) > 1
    lastHigh = 0.0
    lastLow = 0.0
    // 遍历数组，从0开始，步长是1，数组的长度 - 1就是遍历的终点
    for x = 0 to array.size(ZZindexes) - 1 by 1
        // i是从后往前，所以，i一般是最末尾的点
        i = array.size(ZZindexes) - 1 - x
        index = array.get(ZZindexes, i)
        value = array.get(ZZvalues, i)
        highLow = array.get(ZZdir, i)
        index_offset = bar_index - index
        // 注释文字
        labelText = highLow == 2 ? 'HH' : highLow == 1 ? 'LH' : highLow == -1 ? 'HL' : 'LL'
        // 注释颜色
        labelColor = highLow == 2 ? HHColor : highLow == 1 ? LHColor : highLow == -1 ? HLColor : LLColor
        // 线的颜色
        lineColor = highLow == 2 ? HHColor : highLow == 1 ? LHColor : highLow == -1 ? HLColor : LLColor
        // 上升还是下降 
        labelStyle = highLow > 0 ? label.style_label_down : label.style_label_up
        // label的坐标
        labelLocation = yloc.price
        
        // 画label
        if showLabels

            l = label.new(x=index, y=value, xloc=xloc.bar_index, yloc=labelLocation, style=labelStyle, size=size.tiny, color=labelColor,text=labelText)
            array.unshift(labelArray, l)
            if array.size(labelArray) > max_array_size
                label.delete(array.pop(labelArray))
        // 线
        // 遍历的时候从末尾最后一个点开始遍历
        if i < array.size(ZZindexes) - 1 and showZZ
            // 从后往前画，旧的先画，新的最后画
            indexLast = array.get(ZZindexes, i + 1)
            valueLast = array.get(ZZvalues, i + 1)

            l = line.new(x1=index, y1=value, x2=indexLast, y2=valueLast, color=labelColor, width=2, style=line.style_solid)
            array.unshift(lineArray, l)
            if array.size(lineArray) > max_array_size
                line.delete(array.pop(lineArray))
```

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







 

