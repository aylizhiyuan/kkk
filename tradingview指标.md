## tradingview常用指标整理

- 散户视角
    - **趋势类** EMA/SMA/WMA/ADX 判断价格趋势方向/强弱
    - **动能类** MACD/RSI/Stochastic 判断价格涨跌强弱、拐点
    - **波动类** ATR/Bollinger Bands/Super Trend 看价格波动范围
    - **成交量类** OBV/Volume MA 看资金进出强弱
    - **形态类** ZigZag 识别图表形态
- 机构视角
    - **BOS/Choch** 趋势反转确认
    - **OB** 主力建仓区域
    - **Buy/Sell** side liquidity 主力吸单方向
    - **FVG** 价格不平衡区域,易回补

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



Simple Moving Average = (a1+a2+a3...+an) / n 




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

***成交量反转指标**

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
look_back = 20              // 突破的观察周期

// === 状态变量 ===
var bool track_follow = false        // 是否正在跟踪中
var int bars_since_break = 0         // 已经过去的K线数
var int follow_vol_count = 0         // 跟随放量的数量
var bool has_following_vol = false   // 是否确认出现跟随放量
var label buy_label = na  // 保存标签，防止重复画

// === 主放量触发 ===
atr30 = ta.atr(30)          // 每根K线只计算一次
y1 = low - (atr30 * 2.0)

// === 低位突破逻辑 ===
bull_anchor_low = ta.lowest(low, look_back)
is_bull_anchor_break = low <= bull_anchor_low // 当前K线的最低点是20根K线中的最低点
// 爆量K的条件: 成交量1.2, 创造了最低点,阳K
is_main_break = volume > vol_ma * 1.2 and is_bull_anchor_break and close > open

// 假如已经出现了爆量K，并且未启动监控，则启动监控，并恢复初始计时
if is_main_break and not track_follow
    // 只启动跟踪，不立即统计
    track_follow := true
    bars_since_break := 0
    follow_vol_count := 0
    has_following_vol := false

// 爆量已经出现并且启动了监控了
else if track_follow
    // 进入跟踪逻辑
    bars_since_break += 1

    // 检查当前K线是否放量且上涨,放量后进行累加
    if volume > vol_ma * follow_vol_mult and close > open // 必须保证是阳线跟随
        follow_vol_count += 1

    // 若5根内出现至少2根放量，则确认成功
    if follow_vol_count >= follow_vol_required and not has_following_vol
        has_following_vol := true // 展示连续放量标签
        buy_label := label.new(bar_index, low - atr30*2, "BUY\n★", 
                               style=label.style_label_up,
                               color=color.new(color.green,0),
                               textcolor=color.white,
                               size=size.small)

    // 超过5根取消监控并去除标签
    if bars_since_break >= bars_lookahead
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

```js
//General
period = input.int(18, step=1, minval=5, title='Length', group='General', display=display.none)

//SR signal
showSR= input(true, title='Lines', group='Support & Resistance', inline='length', display=display.none)

// Signal settings
showLabels = input(true, title='Labels', group='Show', inline='length1', display=display.none)
showZZ = input(false, title='ZigZag', group='Show', inline='length1', display=display.none)

HHColor = color.green //bullishColorT
LHColor = color.orange //bullTrapColorT
LLColor = color.red //bearishColorT
HLColor = color.lime //bearTrapColorT

// .............Dow's Theory....zigzag........./////
var ZZvalues = array.new_float(0)
var ZZindexes = array.new_int(0)
var ZZdir = array.new_int(0)

int max_array_size = 100
var lineArray = array.new_line(0)
var labelArray = array.new_label(0)
var line dottedLine=na

pivot(period)=>
    float p_h = ta.highestbars(high, period) == 0 ? high : na
    float p_l = ta.lowestbars(low, period) == 0 ? low : na

    dir = 0
    iff_1 = p_l and na(p_h) ? -1 : dir[1]
    dir := p_h and na(p_l) ? 1 : iff_1
    [dir, p_h, p_l]

add_to_array(value, index, dir) =>
    mult = array.size(ZZvalues) < 2 ? 1 : dir * value > dir * array.get(ZZvalues, 1) ? 2 : 1
    array.unshift(ZZindexes, index)
    array.unshift(ZZvalues, value)
    array.unshift(ZZdir, dir * mult)
    if array.size(ZZindexes) > max_array_size
        array.pop(ZZindexes)
        array.pop(ZZvalues)
        array.pop(ZZdir)

add_to_zz(dir, dirchanged, p_h, p_l, index) =>
    value = dir == 1 ? p_h : p_l
    if array.size(ZZvalues) == 0 or dirchanged
        add_to_array(value, index, dir)
    else if dir == 1 and value > array.get(ZZvalues, 0) or dir == -1 and value < array.get(ZZvalues, 0)
        array.shift(ZZvalues)
        array.shift(ZZindexes)
        array.shift(ZZdir)
        add_to_array(value, index, dir)
        
zz(length) =>
    [dir, p_h, p_l] = pivot(length)
    dirchanged = ta.change(dir)
    if p_h or p_l
        add_to_zz(dir, dirchanged, p_h, p_l, bar_index)
zz(period)



if barstate.isconfirmed and array.size(ZZindexes) > 1
    // === 删除旧线 ===
    if array.size(lineArray) > 0
        for i = 0 to array.size(lineArray) - 1
            line.delete(array.get(lineArray, i))
        array.clear(lineArray)
     // === 删除旧标签（可选）===
    if array.size(labelArray) > 0
        for i = 0 to array.size(labelArray) - 1
            label.delete(array.get(labelArray, i))
        array.clear(labelArray)    
    lastHigh = 0.0
    lastLow = 0.0
    for x = 0 to array.size(ZZindexes) - 1 by 1
        i = array.size(ZZindexes) - 1 - x
        index = array.get(ZZindexes, i)
        value = array.get(ZZvalues, i)
        highLow = array.get(ZZdir, i)
        index_offset = bar_index - index

        labelText = highLow == 2 ? 'HH' : highLow == 1 ? 'LH' : highLow == -1 ? 'HL' : 'LL'
        labelColor = highLow == 2 ? HHColor : highLow == 1 ? LHColor : highLow == -1 ? HLColor : LLColor
        lineColor = highLow == 2 ? HHColor : highLow == 1 ? LHColor : highLow == -1 ? HLColor : LLColor
        SRColor = highLow == 2 ? LLColor : highLow == 1 ? LLColor : highLow == -1 ? HHColor : HHColor
        SRColor0 = highLow == 2 ? HHColor : highLow == 1 ? HHColor : highLow == -1 ? LLColor : LLColor
        labelStyle = highLow > 0 ? label.style_label_down : label.style_label_up
        labelLocation = yloc.price
          
        if showLabels

            l = label.new(x=index, y=value, xloc=xloc.bar_index, yloc=labelLocation, style=labelStyle, size=size.tiny, color=labelColor,text=labelText)
            array.unshift(labelArray, l)
            if array.size(labelArray) > max_array_size
                label.delete(array.pop(labelArray))

        if i < array.size(ZZindexes) - 1 and showZZ
            indexLast = array.get(ZZindexes, i + 1)
            valueLast = array.get(ZZvalues, i + 1)

            l = line.new(x1=index, y1=value, x2=indexLast, y2=valueLast, color=labelColor, width=2, style=line.style_solid)
            array.unshift(lineArray, l)
            if array.size(lineArray) > max_array_size
                line.delete(array.pop(lineArray))

        //.............show Support Resistance
        if i < array.size(ZZindexes) - 1 and showSR
            indexLast = array.get(ZZindexes, i + 1)
            valueLast = array.get(ZZvalues, i + 1)
            line.new(x1=index, y1=valueLast, x2=indexLast, y2=valueLast, color=SRColor, width=4, style=line.style_dotted)

            // Create or update a progressing dotted line
            if i==0    
                if na(dottedLine)
            // Create a new dotted line if it doesn't exist
                    dottedLine := line.new(x1=index, y1=value, x2=bar_index, y2=value, color=SRColor0, width=4, style=line.style_dotted)

            if bar_index >= indexLast and i==0
                dottedLine := na  // Reset the dotted line so it stops updating

```

### 六. 聪明钱SMC

#### 1. 概念

订单块 = 庄家成本区 

他们会买一部分的货,在这波拉升这里会形成一个订单块,这里就是庄家的成本区间,所以,当价格再次回到这个订单块,也就是庄家的成本区的时候,价格大概率就会在这个订单块得到反弹以及拉升

趋势突破 = Break of Structure(BOS),价格突破前高,形成顶顶高的上升趋势架构,这意味着上升趋势的延续,后市看涨;价格突破前低形成底底低的下跌趋势架构,这意味着下跌趋势的延续,后市看跌


趋势破坏 = Change of Character(Choch)价钱跌破前低形成底底低的下跌趋势架构,这意味着上升趋势大概率转为下跌趋势;价钱突破前高形成顶顶高的上升趋势架构,这意味着大概率转为上升趋势


#### 2.市场结构

- 市场只有三种结构：上涨 -> 下跌 -> 盘整
- 趋势由高低点结构定义(HH,HL,LH,LL)
- 结构反转(CHoCH),趋势确认(BOS)

把市场切成一格一格的小单元(每5根K线),每一格检查有没有趋势反转,如果反转 ——> 记录pivot,如果没有反转 -> 忽略(不计入结构)

新 K 线到来 → 窗口滑动 → 再用当前 5 根重算 → 持续寻找新的 pivot

```
时间 → 
 bar1      bar2     bar3      bar4      bar5       bar6(current)
high[5]   high[4]  hgih[3]   hight[2]  high[1]    high[0]

检查bar1

bar1 vs [bar2,bar3,bar4,bar5,current[bar6]]

检查bar2

bar2 vs [bar3,bar4,bar5,bar6,current(bar7)]

检查bar3

bar3 vs [bar4,bar5,bar6,bar7,current(bar8)]

```

**1. 核心逻辑**

```js

leg(int size) =>
    var leg     = 0
    // 看跌leg
    // 过往第6根高点 > 之前5根的最高点,所以他是一组结构中(5根为一组)的最高点
    newLegHigh  = high[size] > ta.highest( size)
    // 看涨leg
    // 过往第6根的低点 < 之前5根的最低点,所以它是一组结构中的最低点
    newLegLow   = low[size]  < ta.lowest( size)
    
    if newLegHigh
        leg := BEARISH_LEG // 看跌 0
    else if newLegLow
        leg := BULLISH_LEG // 看涨 1
    leg

// 是否发生了反转,也就是说上涨 -> 下跌 or 下跌 -> 上涨
startOfNewLeg(int leg)      => ta.change(leg) != 0

// 当前leg从涨到跌
startOfBearishLeg(int leg)  => ta.change(leg) == -1

// 当前leg从跌变涨
startOfBullishLeg(int leg)  => ta.change(leg) == +1


```

**2. 获取结构块**

有了结构块以后,我们就可以画出HH HL LL LH这种反转点了, 高点(HH or HL)/低点(LH LL)分别标注

```js
getCurrentStructure(int size,bool equalHighLow = false, bool internal = false) => 
    // 看涨还是看跌       
    currentLeg              = leg(size)
    // 发生反转再记录
    newPivot                = startOfNewLeg(currentLeg)
    // 看跌反转
    pivotLow                = startOfBullishLeg(currentLeg)
    // 看涨反转
    pivotHigh               = startOfBearishLeg(currentLeg)
    // 结构块反转后记录反转的价格
    if newPivot
        // 下跌 --> 上涨，记录上涨起涨点
        if pivotLow
            // 低点序列
            pivot p_ivot    = equalHighLow ? equalLow : internal ? internalLow : swingLow    
            // 上一个 pivot 低点跟当前检测到的 pivot 低点是否相等
            // 这里是3根为一组来进行判断的,并且数据是放在equalLow
            if equalHighLow and math.abs(p_ivot.currentLevel - low[size]) < equalHighsLowsThresholdInput * atrMeasure                
                drawEqualHighLow(p_ivot, low[size], size, false)
            // 上一个 pivot 的价格
            p_ivot.lastLevel    := p_ivot.currentLevel
            // 当前检测到的 pivot 的价格
            p_ivot.currentLevel := low[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]
            // 大周期单独记录到traliing中去
            if not equalHighLow and not internal
                trailing.bottom         := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastBottomTime := p_ivot.barTime
            // 大周期(50根K)画HH LL
            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel < p_ivot.lastLevel ? 'LL' : 'HL', swingBullishColor, label.style_label_up)            
        else
            // 上涨 -> 下跌,记录下跌的起跌点
            pivot p_ivot = equalHighLow ? equalHigh : internal ? internalHigh : swingHigh
            // 上一个 pivot 的高点 跟当前检测到的 pivot 的高点是否相等
            if equalHighLow and math.abs(p_ivot.currentLevel - high[size]) < equalHighsLowsThresholdInput * atrMeasure
                drawEqualHighLow(p_ivot,high[size],size,true)
                           
            // 上一个 pivot 高点 
            p_ivot.lastLevel    := p_ivot.currentLevel
            // 当前检测到的 pivot 高点
            p_ivot.currentLevel := high[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.top            := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastTopTime    := p_ivot.barTime
            // 波段画出  HH LH 
            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel > p_ivot.lastLevel ? 'HH' : 'LH', swingBearishColor, label.style_label_down)
```



**3. 结构突破CHOCH + BOS**

**当价格有效突破反转结构块的高点的时候,意味着价格从下跌转为了上涨**
**当价格有效跌破反转结构块的低点的时候,意味着价格从上涨转为了下跌**


```js
// true 内部结构画线 false 波段画线
displayStructure(bool internal = false) =>
    var bullishBar = true
    var bearishBar = true 

    if internalFilterConfluenceInput
        bullishBar := high - math.max(close, open) > math.min(close, open - low)
        bearishBar := high - math.max(close, open) < math.min(close, open - low)
    // 当前的高点 pivot 内部 or 波段
    pivot p_ivot    = internal ? internalHigh : swingHigh
    trend t_rend    = internal ? internalTrend : swingTrend
    // 内部画虚线，波段画实线
    lineStyle       = internal ? line.style_dashed : line.style_solid
    labelSize       = internal ? internalStructureSize : swingStructureSize

    extraCondition  = internal ? internalHigh.currentLevel != swingHigh.currentLevel and bullishBar : true
    bullishColor    = styleInput == MONOCHROME ? MONO_BULLISH : internal ? internalBullColorInput : swingBullColorInput
    // 价格突破了当前下跌前的高点 → 只有有效突破了下跌趋势的高点(5K),我们认定是看涨突破
    if ta.crossover(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        // 第一次多头突破用CHOCH,随后就用BOS 表示延续
        string tag = t_rend.bias == BEARISH ? CHOCH : BOS


        p_ivot.crossed  := true // 被标记
        t_rend.bias     := BULLISH // 多头标记

        displayCondition = internal ? showInternalsInput and (showInternalBullInput == ALL or (showInternalBullInput == BOS and tag != CHOCH) or (showInternalBullInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBullInput == ALL or (showSwingBullInput == BOS and tag != CHOCH) or (showSwingBullInput == CHOCH and tag == CHOCH))
        // 画出突破横线
        if displayCondition                        
            drawStructure(p_ivot,tag,bullishColor,lineStyle,label.style_label_down,labelSize)
        // 存储看涨的订单块
        if (internal and showInternalOrderBlocksInput) or (not internal and showSwingOrderBlocksInput)
            storeOrdeBlock(p_ivot,internal,BULLISH)
    // 低点突破
    p_ivot          := internal ? internalLow : swingLow    
    extraCondition  := internal ? internalLow.currentLevel != swingLow.currentLevel and bearishBar : true
    bearishColor    = styleInput == MONOCHROME ? MONO_BEARISH : internal ? internalBearColorInput : swingBearColorInput
    // 价格下破了当前上涨前的低点” → 当前结构向空头突破 → 空头订单块
    if ta.crossunder(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
    // 第一次空头突破用CHOCH,随后用BOS
        string tag = t_rend.bias == BULLISH ? CHOCH : BOS
        p_ivot.crossed := true
        t_rend.bias := BEARISH

        displayCondition = internal ? showInternalsInput and (showInternalBearInput == ALL or (showInternalBearInput == BOS and tag != CHOCH) or (showInternalBearInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBearInput == ALL or (showSwingBearInput == BOS and tag != CHOCH) or (showSwingBearInput == CHOCH and tag == CHOCH))
        
        if displayCondition                        
            drawStructure(p_ivot,tag,bearishColor,lineStyle,label.style_label_up,labelSize)
        // 存储看跌的订单块
        if (internal and showInternalOrderBlocksInput) or (not internal and showSwingOrderBlocksInput)
            storeOrdeBlock(p_ivot,internal,BEARISH)

```

**4. orderBlock的删除逻辑**

```js
// 删除订单块的逻辑 , true 内部 false 波段
deleteOrderBlocks(bool internal = false) =>
    array<orderBlock> orderBlocks = internal ? internalOrderBlocks : swingOrderBlocks

    for [index,eachOrderBlock] in orderBlocks
        bool crossedOderBlock = false
        
        if bearishOrderBlockMitigationSource > eachOrderBlock.barHigh and eachOrderBlock.bias == BEARISH
            crossedOderBlock := true
        else if bullishOrderBlockMitigationSource < eachOrderBlock.barLow and eachOrderBlock.bias == BULLISH
            crossedOderBlock := true
        if crossedOderBlock                    
            orderBlocks.remove(index)      


```

**5. order Block的绘制**

```js
drawOrderBlocks(bool internal = false) =>        
    array<orderBlock> orderBlocks = internal ? internalOrderBlocks : swingOrderBlocks
    orderBlocksSize = orderBlocks.size()

    if orderBlocksSize > 0        
        maxOrderBlocks                      = internal ? internalOrderBlocksSizeInput : swingOrderBlocksSizeInput
        array<orderBlock> parsedOrdeBlocks  = orderBlocks.slice(0, math.min(maxOrderBlocks,orderBlocksSize))
        array<box> b_oxes                   = internal ? internalOrderBlocksBoxes : swingOrderBlocksBoxes        

        for [index,eachOrderBlock] in parsedOrdeBlocks
            orderBlockColor = styleInput == MONOCHROME ? (eachOrderBlock.bias == BEARISH ? color.new(MONO_BEARISH,80) : color.new(MONO_BULLISH,80)) : internal ? (eachOrderBlock.bias == BEARISH ? internalBearishOrderBlockColor : internalBullishOrderBlockColor) : (eachOrderBlock.bias == BEARISH ? swingBearishOrderBlockColor : swingBullishOrderBlockColor)

            box b_ox        = b_oxes.get(index)
            b_ox.set_top_left_point(    chart.point.new(eachOrderBlock.barTime,na,eachOrderBlock.barHigh))
            b_ox.set_bottom_right_point(chart.point.new(last_bar_time,na,eachOrderBlock.barLow))        
            b_ox.set_border_color(      internal ? na : orderBlockColor)
            b_ox.set_bgcolor(           orderBlockColor)
```

















 

