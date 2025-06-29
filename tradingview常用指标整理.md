## tradingview常用指标整理

### 1. 多周期日均线

```javascript

//@version=6
indicator("低周期图中显示日线EMA21/50/200", overlay=true)

// 参数
show_ema21 = input.bool(true, "显示日线 EMA 21", group="EMA 设置")
show_ema50 = input.bool(true, "显示日线 EMA 50", group="EMA 设置")
show_ema200 = input.bool(true, "显示日线 EMA 200", group="EMA 设置")

// 请求日线收盘价
get_daily_ema(len) =>
    request.security(syminfo.tickerid, "D", ta.ema(close, len))

// 获取三条EMA
ema21 = get_daily_ema(21)
ema50 = get_daily_ema(50)
ema200 = get_daily_ema(200)

// 绘制
plot(show_ema21 ? ema21 : na, title="日线EMA21", color=color.orange, linewidth=1)
plot(show_ema50 ? ema50 : na, title="日线EMA50", color=color.blue, linewidth=1)
plot(show_ema200 ? ema200 : na, title="日线EMA200", color=color.red, linewidth=1)

```



### 2. 聪明钱SMC


