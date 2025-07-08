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

### 三、价格波动范围

**range 识别震荡区**

```javascript
// This work is licensed under a Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0) https://creativecommons.org/licenses/by-nc-sa/4.0/
// © LuxAlgo

//@version=5
indicator("Range Detector [LuxAlgo]", "LuxAlgo - Range Detector", overlay = true, max_boxes_count = 500, max_lines_count = 500)
//------------------------------------------------------------------------------
//Settings
//-----------------------------------------------------------------------------{
length = input.int(20, 'Minimum Range Length', minval = 2)
mult   = input.float(1., 'Range Width', minval = 0, step = 0.1)
atrLen = input.int(500, 'ATR Length', minval = 1)

//Style
upCss = input(#089981, 'Broken Upward', group = 'Style')
dnCss = input(#f23645, 'Broken Downward', group = 'Style')
unbrokenCss = input(#2157f3, 'Unbroken', group = 'Style')

//-----------------------------------------------------------------------------}
//Detect and highlight ranges
//-----------------------------------------------------------------------------{
//Ranges drawings
var box bx = na
var line lvl = na

//Extensions
var float max = na
var float min = na

var os = 0
color detect_css = na

n = bar_index
atr = ta.atr(atrLen) * mult
ma = ta.sma(close, length)

count = 0
for i = 0 to length-1
    count += math.abs(close[i] - ma) > atr ? 1 : 0

if count == 0 and count[1] != count
    //Test for overlap and change coordinates
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

else if count == 0
    bx.set_right(n)
    lvl.set_x2(n)

//Set color
if close > bx.get_top()
    bx.set_bgcolor(color.new(upCss, 80))
    lvl.set_color(upCss)
    os := 1
else if close < bx.get_bottom()
    bx.set_bgcolor(color.new(dnCss, 80))
    lvl.set_color(dnCss)
    os := -1

//-----------------------------------------------------------------------------}
//Plots
//-----------------------------------------------------------------------------{
//Range detection bgcolor
bgcolor(detect_css)

plot(max, 'Range Top'
  , max != max[1] ? na : os == 0 ? unbrokenCss : os == 1 ? upCss : dnCss)

plot(min, 'Range Bottom'
  , min != min[1] ? na : os == 0 ? unbrokenCss : os == 1 ? upCss : dnCss)

//-----------------------------------------------------------------------------}
```

### 四、成交量


### 五、形态



### 二. 聪明钱SMC

**概念总结**

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







 

