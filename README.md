// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © loxx

//@version=5
indicator("DSS of Advanced Kaufman AMA [Loxx]", shorttitle='DSSAKAMA [Loxx]', timeframe="", overlay = false, timeframe_gaps=true)

greencolor = #2DD204  
redcolor = #D2042D 

_jfract(count)=> 
    window = math.ceil(count/2)
    _hl1  = (ta.highest(high[window], window) - ta.lowest(low[window], window)) / window
    _hl2  = (ta.highest(high, window) - ta.lowest(low, window)) / window
    _hl   = (ta.highest(high, count) - ta.lowest(low, count)) / count
    _d    = (math.log(_hl1 + _hl2) - math.log(_hl)) / math.log(2)
    dim  = _d < 1 ? 1 : _d > 2 ? 2 : _d

_kama(src, len, fast, slow, jcount, power, efratiocalc) =>
    fastend = (2.0 /(fast + 1))
    slowend = (2.0 /(slow + 1))
    mom = math.abs(ta.change(src, len))
    vola = math.sum(math.abs(ta.change(src)), len)
    efratio = efratiocalc == "Regular" ? (vola != 0 ? mom / vola : 0) : math.min(2.0-_jfract(jcount), 1.0) 
    alpha = math.pow(efratio * (fastend - slowend) + slowend, power)
    kama = 0.0
    kama := alpha * src + (1 - alpha) * nz(kama[1], src)
    kama


blsrc = input.source(close, "Source", group = "Kaufman AMA Settings")
period = input.int(10, "Period", minval = 0, group = "Kaufman AMA Settings")
kama_fastend = input.float(2, "Kaufman AMA Fast-end Period", minval = 0.0, group = "Kaufman AMA Settings")
kama_slowend = input.float(30, "Kaufman AMA Slow-end Period",  minval = 0.0, group = "Kaufman AMA Settings")
efratiocalc = input.string("Fractal Dimension Adaptive", "Efficiency Ratio Type", options = ["Regular", "Fractal Dimension Adaptive"], group = "Kaufman AMA Settings")
jcount = input.int(defval=2, title="Fractal Dimension Count ", group = "Kaufman AMA Settings")
SmoothPower = input.int(2, "Kaufman Power Smooth", group = "Kaufman AMA Settings")

stochLen = input.int(30, "Stoch Smooth Period", group = "Stochastic Settings")
smEMA = input.int(9, "Intermediate Smooth Period", group = "Stochastic Settings")
sigEMA = input.int(5, "Signal Smooth Period", group = "Stochastic Settings")

colorbars = input.bool(true, "Color bars?", group = "UI Options")

 
kamaC = _kama(blsrc, period, kama_fastend, kama_slowend, jcount, SmoothPower, efratiocalc) 
kamaHi = ta.highest(kamaC, stochLen)
kamaLo = ta.lowest(kamaC, stochLen) 

st1 = ta.stoch(kamaC, kamaHi, kamaLo, stochLen)
emaout = ta.ema(st1, smEMA)

firsthi = ta.highest(emaout, stochLen)
firstlo = ta.lowest(emaout, stochLen)

out = ta.stoch(emaout, firsthi, firstlo, stochLen)
outer = ta.ema(out, smEMA)

signal = ta.ema(outer, sigEMA)

plot(signal, color = color.white, linewidth = 1)

plot(50, color=color.new(color.gray, 30), linewidth=1, style=plot.style_circles, title = "Zero line")
plot(outer, color = outer > signal ? greencolor : redcolor, linewidth = 2)

barcolor(colorbars ? outer > signal ? greencolor : redcolor : na)











