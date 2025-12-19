```
//@version=5
indicator('ICT', '乘舟ICT', overlay = true, max_labels_count = 500, max_lines_count = 500, max_boxes_count = 500)
//---------------------------------------------------------------------------------------------------------------------}
// 常量 & 输入
//---------------------------------------------------------------------------------------------------------------------{
BULLISH_LEG                     = 1
BEARISH_LEG                     = 0

BULLISH                         = +1
BEARISH                         = -1

GREEN                           = #089981
RED                             = #F23645
BLUE                            = #2157f3
GRAY                            = #878b94
MONO_BULLISH                    = #b2b5be
MONO_BEARISH                    = #5d606b

HISTORICAL                      = 'Historical'
PRESENT                         = 'Present'

COLORED                         = 'Colored'
MONOCHROME                      = 'Monochrome'

ALL                             = 'All'
BOS                             = 'BOS'
CHOCH                           = 'CHoCH'

TINY                            = size.tiny
SMALL                           = size.small
NORMAL                          = size.normal

ATR                             = 'Atr'
RANGE                           = 'Cumulative Mean Range'

CLOSE                           = 'Close'
HIGHLOW                         = 'High/Low'

SOLID                           = '⎯⎯⎯'
DASHED                          = '----'
DOTTED                          = '····'

SMART_GROUP                     = '聪明资金概念'
INTERNAL_GROUP                  = '小周期结构'
SWING_GROUP                     = '大周期结构'
BLOCKS_GROUP                    = '订单块'
EQUAL_GROUP                     = '相等高点/低点'



modeTooltip                     = '允许显示历史结构或仅显示最近的结构'
styleTooltip                    = '指标的颜色主题'
showTrendTooltip                = '显示额外的蜡烛图，颜色反映由结构检测到的当前趋势'
showInternalsTooltip            = '显示内部市场结构'
internalFilterConfluenceTooltip = '过滤不重要的内部结构突破'
showStructureTooltip            = '显示波段市场结构'
showSwingsTooltip               = '在图表上以标签形式显示波段点'
showHighLowSwingsTooltip        = '高亮显示最近的强弱高点和低点'
showEqualHighsLowsTooltip       = '在图表上显示相等的高点和低点'
equalHighsLowsLengthTooltip     = '用于确认相等高点和低点的K线数量'
equalHighsLowsThresholdTooltip  = '检测相等高点和低点的敏感度阈值，范围 (0,1)\n\n值越低,结果越少但越精确'


modeInput                       = input.string(HISTORICAL, '模式', group = SMART_GROUP, tooltip = modeTooltip, options = [HISTORICAL, PRESENT])
styleInput                      = input.string(COLORED, '样式', group = SMART_GROUP, tooltip = styleTooltip, options = [COLORED, MONOCHROME])


showInternalsInput              = input(true, '显示小周期结构', group = INTERNAL_GROUP, tooltip = showInternalsTooltip)
showInternalBullInput           = input.string(ALL, '小周期看涨结构', group = INTERNAL_GROUP, inline = 'ibull', options = [ALL,BOS,CHOCH])
internalBullColorInput          = input(GREEN, '', group = INTERNAL_GROUP, inline = 'ibull')
showInternalBearInput           = input.string(ALL, '小周期看跌结构', group = INTERNAL_GROUP, inline = 'ibear', options = [ALL,BOS,CHOCH])
internalBearColorInput          = input(RED, '', group = INTERNAL_GROUP, inline = 'ibear')
internalFilterConfluenceInput   = input(false, '小周期结构汇合过滤', group = INTERNAL_GROUP, tooltip = internalFilterConfluenceTooltip)
internalStructureSize           = input.string(TINY, '小周期标签大小', group = INTERNAL_GROUP, options = [TINY,SMALL,NORMAL])

showStructureInput              = input(true, '显示大周期结构', group = SWING_GROUP, tooltip = showStructureTooltip)
showSwingBullInput              = input.string(ALL, '大周期看涨结构', group = SWING_GROUP, inline = 'bull', options = [ALL,BOS,CHOCH])
swingBullColorInput             = input(GREEN, '', group = SWING_GROUP, inline = 'bull')
showSwingBearInput              = input.string(ALL, '大周期看跌结构', group = SWING_GROUP, inline = 'bear', options = [ALL,BOS,CHOCH])
swingBearColorInput             = input(RED, '', group = SWING_GROUP, inline = 'bear')
swingStructureSize              = input.string(SMALL, '大周期标签大小', group = SWING_GROUP, options = [TINY,SMALL,NORMAL])
showSwingsInput                 = input(false, '大周期zig', group = SWING_GROUP, tooltip = showSwingsTooltip, inline = 'swings')
showHighLowSwingsInput          = input(true, '显示强弱高/低点', group = SWING_GROUP, tooltip = showHighLowSwingsTooltip)
swingsLengthInput               = input.int(50, '定义周期的长度/K线数量', group = SWING_GROUP, minval = 10, inline = 'swings')


showEqualHighsLowsInput         = input(true, '相等高/低点', group = EQUAL_GROUP, tooltip = showEqualHighsLowsTooltip)
equalHighsLowsLengthInput       = input.int(3, '确认K线数量', group = EQUAL_GROUP, tooltip = equalHighsLowsLengthTooltip, minval = 1)
equalHighsLowsThresholdInput    = input.float(0.1, '阈值', group = EQUAL_GROUP, tooltip = equalHighsLowsThresholdTooltip, minval = 0, maxval = 0.5, step = 0.1)
equalHighsLowsSizeInput         = input.string(TINY, '标签大小', group = EQUAL_GROUP, options = [TINY,SMALL,NORMAL])



//---------------------------------------------------------------------------------------------------------------------}
// 数据类型 & 变量


// @type                            UDT representing last swing extremes (top & bottom)
// @field top                       last top swing price
// @field bottom                    last bottom swing price
// @field barTime                   last swing bar time
// @field barIndex                  last swing bar index
// @field lastTopTime               last top swing time
// @field lastBottomTime            last bottom swing time
type trailingExtremes
    float top
    float bottom
    int barTime
    int barIndex
    int lastTopTime
    int lastBottomTime

// @type                            UDT representing trend bias
// @field bias                      BULLISH or BEARISH
type trend
    int bias    

// @type                            UDT representing Equal Highs Lows display
// @field l_ine                     displayed line
// @field l_abel                    displayed label
type equalDisplay
    line l_ine      = na
    label l_abel    = na

// @type                            UDT representing a pivot point (swing point) 
// @field currentLevel              current price level
// @field lastLevel                 last price level
// @field crossed                   true if price level is crossed
// @field barTime                   bar time
// @field barIndex                  bar index    
type pivot
    float currentLevel
    float lastLevel
    bool crossed
    int barTime     = time
    int barIndex    = bar_index

// @type                            UDT representing an order block
// @field barHigh                   bar high
// @field barLow                    bar low
// @field barTime                   bar time
// @field bias                      BULLISH or BEARISH
type orderBlock
    float barHigh
    float barLow
    int barTime    
    int bias

// @variable                        current swing pivot high    
var pivot swingHigh                 = pivot.new(na,na,false)
// @variable                        current swing pivot low
var pivot swingLow                  = pivot.new(na,na,false)
// @variable                        current internal pivot high
var pivot internalHigh              = pivot.new(na,na,false)
// @variable                        current internal pivot low
var pivot internalLow               = pivot.new(na,na,false)
// @variable                        current equal high pivot
var pivot equalHigh                 = pivot.new(na,na,false)
// @variable                        current equal low pivot
var pivot equalLow                  = pivot.new(na,na,false)
// @variable                        swing trend bias
var trend swingTrend                = trend.new(0)
// @variable                        internal trend bias
var trend internalTrend             = trend.new(0)
// @variable                        equal high display
var equalDisplay equalHighDisplay   = equalDisplay.new()
// @variable                        equal low display
var equalDisplay equalLowDisplay    = equalDisplay.new()
// @variable                        storage for bar time values
var array<int> times                = array.new<int>()
// @variable                        last trailing swing high and low
var trailingExtremes trailing       = trailingExtremes.new()
// @variable                        color for swing bullish structures
var swingBullishColor               = styleInput == MONOCHROME ? MONO_BULLISH : swingBullColorInput
// @variable                        color for swing bearish structures
var swingBearishColor               = styleInput == MONOCHROME ? MONO_BEARISH : swingBearColorInput
// @variable                        bar index on current script iteration
varip int currentBarIndex           = bar_index
// @variable                        bar index on last script iteration
varip int lastBarIndex              = bar_index
// @variable                        time at start of chart
var initialTime                     = time

// 平均波动幅度
atrMeasure                          = ta.atr(200)

//---------------------------------------------------------------------------------------------------------------------}
// 函数
//---------------------------------------------------------------------------------------------------------------------{

// @function            Get the value of the current leg, it can be 0 (bearish) or 1 (bullish)
// @returns             int
leg(int size) =>
    var leg     = 0
    // 判断某个历史位置是否是近期的最高价，例如往回数第5根是不是从当前算的最高价 
    newLegHigh  = high[size] > ta.highest( size)
    // 判断某个历史位置是否是近期的最低价,假如往回数第5根是不是最近5根中的最低价
    newLegLow   = low[size]  < ta.lowest(  size)
    
    if newLegHigh
        leg := BEARISH_LEG // 看跌
    else if newLegLow
        leg := BULLISH_LEG // 看涨
    leg

// leg已经知道了是涨跌,是否发生了反转
startOfNewLeg(int leg)      => ta.change(leg) != 0

// 当前波段从涨变跌
startOfBearishLeg(int leg)  => ta.change(leg) == -1

// 当前波段从跌变涨
startOfBullishLeg(int leg)  => ta.change(leg) == +1

// @function            create a new label
// @param labelTime     bar time coordinate
// @param labelPrice    price coordinate
// @param tag           text to display
// @param labelColor    text color
// @param labelStyle    label style
// @returns             label ID
drawLabel(int labelTime, float labelPrice, string tag, color labelColor, string labelStyle) =>    
    var label l_abel = na

    if modeInput == PRESENT
        l_abel.delete()

    l_abel := label.new(chart.point.new(labelTime,na,labelPrice),tag,xloc.bar_time,color=color(na),textcolor=labelColor,style = labelStyle,size = size.small)

// @function            create a new line and label representing an EQH or EQL
// @param p_ivot        starting pivot
// @param level         price level of current pivot
// @param size          how many bars ago was the current pivot detected
// @param equalHigh     true for EQH, false for EQL
// @returns             label ID
drawEqualHighLow(pivot p_ivot, float level, int size, bool equalHigh) =>
    equalDisplay e_qualDisplay = equalHigh ? equalHighDisplay : equalLowDisplay
    
    string tag          = 'EQL'
    color equalColor    = swingBullishColor
    string labelStyle   = label.style_label_up

    if equalHigh
        tag         := 'EQH'
        equalColor  := swingBearishColor
        labelStyle  := label.style_label_down

    if modeInput == PRESENT
        line.delete(    e_qualDisplay.l_ine)
        label.delete(   e_qualDisplay.l_abel)
        
    e_qualDisplay.l_ine     := line.new(chart.point.new(p_ivot.barTime,na,p_ivot.currentLevel), chart.point.new(time[size],na,level), xloc = xloc.bar_time, color = equalColor, style = line.style_dotted)
    labelPosition           = math.round(0.5*(p_ivot.barIndex + bar_index - size))
    e_qualDisplay.l_abel    := label.new(chart.point.new(na,labelPosition,level), tag, xloc.bar_index, color = color(na), textcolor = equalColor, style = labelStyle, size = equalHighsLowsSizeInput)

// @function            store current structure and trailing swing points, and also display swing points and equal highs/lows
// @param size          (int) structure size
// @param equalHighLow  (bool) true for displaying current highs/lows
// @param internal      (bool) true for getting internal structures
// @returns             label ID
getCurrentStructure(int size,bool equalHighLow = false, bool internal = false) =>        
    currentLeg              = leg(size)
    newPivot                = startOfNewLeg(currentLeg)
    pivotLow                = startOfBullishLeg(currentLeg)
    pivotHigh               = startOfBearishLeg(currentLeg)

    if newPivot
        if pivotLow
            pivot p_ivot    = equalHighLow ? equalLow : internal ? internalLow : swingLow    

            if equalHighLow and math.abs(p_ivot.currentLevel - low[size]) < equalHighsLowsThresholdInput * atrMeasure                
                drawEqualHighLow(p_ivot, low[size], size, false)
        

            p_ivot.lastLevel    := p_ivot.currentLevel
            p_ivot.currentLevel := low[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.bottom         := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastBottomTime := p_ivot.barTime

            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel < p_ivot.lastLevel ? 'LL' : 'HL', swingBullishColor, label.style_label_up)            
        else
            pivot p_ivot = equalHighLow ? equalHigh : internal ? internalHigh : swingHigh

            if equalHighLow and math.abs(p_ivot.currentLevel - high[size]) < equalHighsLowsThresholdInput * atrMeasure
                drawEqualHighLow(p_ivot,high[size],size,true)
                           

            p_ivot.lastLevel    := p_ivot.currentLevel
            p_ivot.currentLevel := high[size]
            p_ivot.crossed      := false
            p_ivot.barTime      := time[size]
            p_ivot.barIndex     := bar_index[size]

            if not equalHighLow and not internal
                trailing.top            := p_ivot.currentLevel
                trailing.barTime        := p_ivot.barTime
                trailing.barIndex       := p_ivot.barIndex
                trailing.lastTopTime    := p_ivot.barTime

            if showSwingsInput and not internal and not equalHighLow
                drawLabel(time[size], p_ivot.currentLevel, p_ivot.currentLevel > p_ivot.lastLevel ? 'HH' : 'LH', swingBearishColor, label.style_label_down)
                
// @function                draw line and label representing a structure
// @param p_ivot            base pivot point
// @param tag               test to display
// @param structureColor    base color
// @param lineStyle         line style
// @param labelStyle        label style
// @param labelSize         text size
// @returns                 label ID
drawStructure(pivot p_ivot, string tag, color structureColor, string lineStyle, string labelStyle, string labelSize) =>    
    var line l_ine      = line.new(na,na,na,na,xloc = xloc.bar_time)
    var label l_abel    = label.new(na,na)

    if modeInput == PRESENT
        l_ine.delete()
        l_abel.delete()

    l_ine   := line.new(chart.point.new(p_ivot.barTime,na,p_ivot.currentLevel), chart.point.new(time,na,p_ivot.currentLevel), xloc.bar_time, color=structureColor, style=lineStyle)
    l_abel  := label.new(chart.point.new(na,math.round(0.5*(p_ivot.barIndex+bar_index)),p_ivot.currentLevel), tag, xloc.bar_index, color=color(na), textcolor=structureColor, style=labelStyle, size = labelSize)



// @function            detect and draw structures, also detect and store order blocks
// @param internal      true for internal structures or order blocks
// @returns             void
displayStructure(bool internal = false) =>
    var bullishBar = true
    var bearishBar = true

    if internalFilterConfluenceInput
        bullishBar := high - math.max(close, open) > math.min(close, open - low)
        bearishBar := high - math.max(close, open) < math.min(close, open - low)
    
    pivot p_ivot    = internal ? internalHigh : swingHigh
    trend t_rend    = internal ? internalTrend : swingTrend

    lineStyle       = internal ? line.style_dashed : line.style_solid
    labelSize       = internal ? internalStructureSize : swingStructureSize

    extraCondition  = internal ? internalHigh.currentLevel != swingHigh.currentLevel and bullishBar : true
    bullishColor    = styleInput == MONOCHROME ? MONO_BULLISH : internal ? internalBullColorInput : swingBullColorInput

    if ta.crossover(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        string tag = t_rend.bias == BEARISH ? CHOCH : BOS


        p_ivot.crossed  := true
        t_rend.bias     := BULLISH

        displayCondition = internal ? showInternalsInput and (showInternalBullInput == ALL or (showInternalBullInput == BOS and tag != CHOCH) or (showInternalBullInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBullInput == ALL or (showSwingBullInput == BOS and tag != CHOCH) or (showSwingBullInput == CHOCH and tag == CHOCH))

        if displayCondition                        
            drawStructure(p_ivot,tag,bullishColor,lineStyle,label.style_label_down,labelSize)


    p_ivot          := internal ? internalLow : swingLow    
    extraCondition  := internal ? internalLow.currentLevel != swingLow.currentLevel and bearishBar : true
    bearishColor    = styleInput == MONOCHROME ? MONO_BEARISH : internal ? internalBearColorInput : swingBearColorInput

    if ta.crossunder(close,p_ivot.currentLevel) and not p_ivot.crossed and extraCondition
        string tag = t_rend.bias == BULLISH ? CHOCH : BOS


        p_ivot.crossed := true
        t_rend.bias := BEARISH

        displayCondition = internal ? showInternalsInput and (showInternalBearInput == ALL or (showInternalBearInput == BOS and tag != CHOCH) or (showInternalBearInput == CHOCH and tag == CHOCH)) : showStructureInput and (showSwingBearInput == ALL or (showSwingBearInput == BOS and tag != CHOCH) or (showSwingBearInput == CHOCH and tag == CHOCH))
        
        if displayCondition                        
            drawStructure(p_ivot,tag,bearishColor,lineStyle,label.style_label_up,labelSize)



// @function            get line style from string
// @param style         line style
// @returns             string
getStyle(string style) =>
    switch style
        SOLID => line.style_solid
        DASHED => line.style_dashed
        DOTTED => line.style_dotted


// @function            true if chart timeframe is higher than provided timeframe
// @param timeframe     timeframe to check
// @returns             bool
higherTimeframe(string timeframe) => timeframe.in_seconds() > timeframe.in_seconds(timeframe)

// @function            update trailing swing points
// @returns             int
updateTrailingExtremes() =>
    trailing.top            := math.max(high,trailing.top)
    trailing.lastTopTime    := trailing.top == high ? time : trailing.lastTopTime
    trailing.bottom         := math.min(low,trailing.bottom)
    trailing.lastBottomTime := trailing.bottom == low ? time : trailing.lastBottomTime

// @function            draw trailing swing points
// @returns             void
drawHighLowSwings() =>
    var line topLine        = line.new(na, na, na, na, color = swingBearishColor, xloc = xloc.bar_time)
    var line bottomLine     = line.new(na, na, na, na, color = swingBullishColor, xloc = xloc.bar_time)
    var label topLabel      = label.new(na, na, color=color(na), textcolor = swingBearishColor, xloc = xloc.bar_time, style = label.style_label_down, size = size.tiny)
    var label bottomLabel   = label.new(na, na, color=color(na), textcolor = swingBullishColor, xloc = xloc.bar_time, style = label.style_label_up, size = size.tiny)

    rightTimeBar            = last_bar_time + 20 * (time - time[1])

    topLine.set_first_point(    chart.point.new(trailing.lastTopTime, na, trailing.top))
    topLine.set_second_point(   chart.point.new(rightTimeBar, na, trailing.top))
    topLabel.set_point(         chart.point.new(rightTimeBar, na, trailing.top))
    topLabel.set_text(          swingTrend.bias == BEARISH ? 'Strong High' : 'Weak High')

    bottomLine.set_first_point( chart.point.new(trailing.lastBottomTime, na, trailing.bottom))
    bottomLine.set_second_point(chart.point.new(rightTimeBar, na, trailing.bottom))
    bottomLabel.set_point(      chart.point.new(rightTimeBar, na, trailing.bottom))
    bottomLabel.set_text(       swingTrend.bias == BULLISH ? 'Strong Low' : 'Weak Low')




//---------------------------------------------------------------------------------------------------------------------}
// 执行
//---------------------------------------------------------------------------------------------------------------------{

//  // 单独处理插针的高点、低点的强弱程度
if showHighLowSwingsInput
    updateTrailingExtremes()
    drawHighLowSwings()
// 大周期结构获取
getCurrentStructure(swingsLengthInput,false)
// 小周期结构获取
// getCurrentStructure(10,false,true)
// 等高等低结构获取
if showEqualHighsLowsInput
    getCurrentStructure(equalHighsLowsLengthInput,true)
// 小周期结构突破
// if showInternalsInput
//     displayStructure(true)
// 大周期结构突破
if showStructureInput or showHighLowSwingsInput
    displayStructure()



// zigzag

//General
period = input.int(20, step=1, minval=5, title='Length', group='zigzag', display=display.none)

//SR signal
showSR= input(false, title='支撑/压力线', group='zigzag', inline='length', display=display.none)

// Signal settings
showLabels = input(true, title='是否显示标签', group='zigzag', inline='length1', display=display.none)
showZZ = input(true, title='是否显示转折线', group='zigzag', inline='length1', display=display.none)

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