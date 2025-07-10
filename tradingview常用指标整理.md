## tradingview常用指标整理

- 散户视角
    - 趋势类 EMA/SMA/WMA 判断价格趋势方向
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


### 二、价格涨跌强度

**MACD**

MACD = 快慢均线的距离

快速EMA(21)
慢速EMA(55)

DIF线 = 快EMA - 慢EMA = 它们的位置偏离 = 均线开口的高度差

DEA线 = DIF的9日EMA(平滑后的偏离趋势) = 均值,偏离的平均水平

MACD柱状图 = DIF - DEA(动能增减的体现) = 反应这个开口相对于平均趋势是在扩大还是缩小 = 当前开口情况

**RSI**

RSI = 价格上涨下跌力度的强弱 = 通过计算一定周期内上涨和下跌的平均幅度，得出一个 0 到 100 之间的数值

假设周期是N,涨幅为正,下跌为0,跌幅为正,上涨为0

平均上涨幅度 = N天内所有上涨日的涨幅涨幅之和 / N

平均下跌幅度 = N天内所有下跌日的跌幅之和 / N 

RS = 平均上涨幅度  / 平均下跌幅度

RS大的话,说明上涨 > 下跌; RS很小的话,下跌 < 上涨

RS本质上是一个0到正无穷的数值,RSI把RS映射到0-100之间,形成一个标准化的指标

RSI = 100 - (100 / 1 + RS)




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


**ATR**

ATR = 价格波动的强度 = 数字表示 = 某一天大了说明波动大了





**range 识别震荡区**

```javascript
// This work is licensed under a Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) https://creativecommons.org/licenses/by-nc-sa/4.0/
// © LuxAlgo

//@version=5
// max_boxes_count 最多绘制500个box
// max_lines_count 最多绘制500个line
indicator("Range Detector [LuxAlgo]", "LuxAlgo - Range Detector", overlay = true, max_boxes_count = 500, max_lines_count = 500)
//------------------------------------------------------------------------------
//Settings
// length 检测一个震荡区间最少包含多少根K线
length = input.int(20, 'Minimum Range Length', minval = 2)
// 范围宽度乘数(用于放大ATR)
mult   = input.float(1., 'Range Width', minval = 0, step = 0.1)
// ATR的计算周期
atrLen = input.int(500, 'ATR Length', minval = 1)

//Style
upCss = input(#089981, 'Broken Upward', group = 'Style')
dnCss = input(#f23645, 'Broken Downward', group = 'Style')
unbrokenCss = input(#2157f3, 'Unbroken', group = 'Style')

//-----------------------------------------------------------------------------}
//Detect and highlight ranges
//-----------------------------------------------------------------------------{
// bx 和lvl用于保存当前绘制的box和line对象
var box bx = na
var line lvl = na

// 定义当前box的上下边界
var float max = na
var float min = na

// os用于记录突破方向(0代表未突破,1向上,-1向下)
var os = 0
// 背景色,在识别范围期间用灰色显示
color detect_css = na
// 当前bar的索引
n = bar_index
// 使用art * mult 得到区上下边界范围
atr = ta.atr(atrLen) * mult
// 使用SMA对收盘价做平滑处理,作为区间中心
ma = ta.sma(close, length)

count = 0
// 前25跟K线拿出来跟均值ma比较
for i = 0 to length-1
    count += math.abs(close[i] - ma) > atr ? 1 : 0
// 刚进入震荡区间
if count == 0 and count[1] != count
    // 合并两个震荡区
    if n[length] <= bx.get_right()
        max := math.max(ma + atr, bx.get_top())
        min := math.min(ma - atr, bx.get_bottom())
        
        //Box new coordinates
        bx.set_top(max)
        bx.set_rightbottom(n, min)
        bx.set_bgcolor(color.new(unbrokenCss, 80))

        //Line new coordinates
        avg = math.avg(max, min)
        lvl.set_y1(avg)
        lvl.set_xy2(n, avg)
        lvl.set_color(unbrokenCss)
    else
        // 新画一个震荡区
        max := ma + atr
        min := ma - atr

        //Set new box and level
        bx := box.new(n[length], ma + atr, n, ma - atr, na
          , bgcolor = color.new(unbrokenCss, 80))
        
        lvl := line.new(n[length], ma, n, ma
          , color = unbrokenCss
          , style = line.style_dotted)

        detect_css := color.new(color.gray, 80)
        os := 0
// 已经在震荡区间了,覆盖更多的K线
else if count == 0
    // 把之前画的震荡区右边界,往当前这根K线的位置延伸,让框变宽
    bx.set_right(n)
    // 把中线的终点也延伸到当前K线的位置
    lvl.set_x2(n)

// 当前收盘价是否高于震荡区上边界
if close > bx.get_top()
    // 定义为向上突破
    bx.set_bgcolor(color.new(upCss, 80))
    lvl.set_color(upCss)
    os := 1
else if close < bx.get_bottom()
    // 定义为向下突破
    bx.set_bgcolor(color.new(dnCss, 80))
    lvl.set_color(dnCss)
    os := -1

//-----------------------------------------------------------------------------}
//Plots
//-----------------------------------------------------------------------------{
bgcolor(detect_css)
// 画出上下边界的线,以均线为中心,加上或者减去一定倍数的ATR,算出来 
plot(max, 'Range Top'
  , max != max[1] ? na : os == 0 ? unbrokenCss : os == 1 ? upCss : dnCss)

plot(min, 'Range Bottom'
  , min != min[1] ? na : os == 0 ? unbrokenCss : os == 1 ? upCss : dnCss)
```

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







 

