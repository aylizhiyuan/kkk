## tradingview常用指标整理

### 一. 移动平均线 MA/EMA/SMA


***多周期移动平均线EMA21/55***

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



### 二. 聪明钱SMC

订单块 = 庄家成本区 

他们会买一部分的货,在这波拉升这里会形成一个订单块,这里就是庄家的成本区间,所以,当价格再次回到这个订单块,也就是庄家的成本区的时候,价格大概率就会在这个订单块得到反弹以及拉升

趋势突破 = Break of Structure(BOS),价格突破前高,形成顶顶高的上升趋势架构,这意味着上升趋势的延续,后市看涨;价格突破前低形成底底低的下跌趋势架构,这意味着下跌趋势的延续,后市看跌


趋势破坏 = Change of Character(Choch)价钱跌破前低形成底底低的下跌趋势架构,这意味着上升趋势大概率转为下跌趋势;价钱突破前高形成顶顶高的上升趋势架构,这意味着大概率转为上升趋势

**大阳线大概率才可以证明是庄家和机构的介入**


流动性 = 钱 = 庄家止损散户的区域


- 震荡行情

支撑位,附近做多,压力位附近做空,庄家会先打掉你的止损单,无论是支撑位的还是压力位的

- 上升趋势

散户会在趋势线附近做多,止损点会在趋势线的下方,庄家会打掉散户的止损单后拉升

这就是为啥这么多的假突破



### 三、zig







 

