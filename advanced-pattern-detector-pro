//@version=5
indicator("Advanced Pattern Detector Pro [Yosiet]", overlay=true, max_labels_count=500, dynamic_requests=true)
import PineCoders/Time/4

asiaSessionStart = input.int(0, "Asia Session Start Hour (GMT)", minval=0, maxval=23)
asiaSessionEnd = input.int(5, "Asia Session End Hour (GMT)", minval=1, maxval=23)
accumulationPeriod = input.int(10, "Accumulation Period", minval=5, maxval=20)
volumeThreshold = input.float(1.5, "Volume Threshold", minval=1.0, maxval=3.0)
i_bodyThreshold = input.float(0.6, "Candle Body Threshold", minval=0.1, maxval=1, step=0.1)

// Input parameters
startHour = input.int(8, "NY Start-Showtime Hour")
endHour = input.int(10, "NY End-Showtime Hour")

// Function to check if we're in the strategy background hours
isShowtimeHours() =>
    hour == startHour and minute == 0 or
     (hour > startHour and hour < endHour) or
     (hour == endHour and minute < 60)

// Color for the background
customColor = input.color(color.new(color.blue, 90), "NY 8am - 10am Background")

// Apply background color
bgcolor(isShowtimeHours() ? customColor : na)

//mark LL and HH for showtime hours 
[h,l,c] = request.security(syminfo.tickerid, '60', [high, low, close]) 
ll = l
hh = h
if isShowtimeHours()
    ll := math.min(ll[1], l)
    hh := math.max(hh[1], h)
plot(isShowtimeHours() ? ll : na , title="LL",style=plot.style_circles, color=color.gray, linewidth=1) 
plot(isShowtimeHours() ? hh : na , title="HH",style=plot.style_circles, color=color.gray, linewidth=1) 




//chech if is Asia session 
inAsianSession() =>
    currentTime = time("1", "GMT")
    currentHour = hour(currentTime)
    if asiaSessionStart < asiaSessionEnd
        currentHour >= asiaSessionStart and currentHour < math.min(asiaSessionEnd, asiaSessionStart + 5)
    else
        (currentHour >= asiaSessionStart and currentHour < 24) or (currentHour >= 0 and currentHour < math.min(asiaSessionEnd, (asiaSessionStart + 5) % 24))


// Helper functions
bodySize(index) => math.abs(close[index] - open[index])
isWhite(index) => close[index] > open[index]
isBlack(index) => close[index] < open[index]
bodyRatio(idx) => math.abs(close[idx] - open[idx]) / (high[idx] - low[idx])

//Patterns detection functions
threeStarsInSouth() =>
    bearish1 = isBlack(3)
    bearish2 = isBlack(2)
    bearish3 = isBlack(1)
    smallerBodies = bodySize(1) < bodySize(2) and bodySize(2) < bodySize(3)
    lowerHighs = high[2] < high[3] and high[1] < high[2]
    higherLows = low[2] > low[3] and low[1] > low[2]
    lastStarDoji = bodyRatio(1) < i_bodyThreshold
    bearish1 and bearish2 and bearish3 and smallerBodies and lowerHighs and higherLows and lastStarDoji

engulfingBullish() =>
    prevBearish = isBlack(1)
    currBullish = isWhite(0)
    engulfing = open <= close[1] and close >= open[1]
    volumeIncrease = volume > volume[1] * volumeThreshold
    prevBearish and currBullish and engulfing and volumeIncrease

threeBlackCrows() =>
    bearish1 = isBlack(2)
    bearish2 = isBlack(1)
    bearish3 = isBlack(0)
    lowerCloses = close < close[1] and close[1] < close[2]
    smallUpperWicks = (high - open) / (high - low) < 0.2 and (high[1] - open[1]) / (high[1] - low[1]) < 0.2 and (high[2] - open[2]) / (high[2] - low[2]) < 0.2
    volumeIncrease = volume > volume[1] and volume[1] > volume[2]
    bearish1 and bearish2 and bearish3 and lowerCloses and smallUpperWicks// and volumeIncrease

priceRangeAccumulation(period) =>
    highestHigh = ta.highest(high, period)
    lowestLow = ta.lowest(low, period)
    priceRange = (highestHigh - lowestLow) / lowestLow
    averageVolume = ta.sma(volume, period)
    volumeCondition = volume > averageVolume
    priceRange <= 0.03 and volumeCondition

//dectect
var threeStarsEngulfing = false
var threeBlackCrowsAccumulation = false
var asianSessionThreeBlackCrows = false

threeStarsEngulfing := threeStarsInSouth()[1] and engulfingBullish()
threeBlackCrowsAccumulation := threeBlackCrows() and priceRangeAccumulation(accumulationPeriod)
asianSessionThreeBlackCrows := threeBlackCrows() and inAsianSession()

//get trend strength
getPatternStrength(patternDetected) =>
    if patternDetected
        volumeStrength = volume > ta.sma(volume, 20) ? 50 : 0
        trendStrength = ta.ema(close, 10) > ta.ema(close, 20) ? 50 : 0
        int(volumeStrength + trendStrength)
    else
        0

threeStarsEngulfingStrength = getPatternStrength(threeStarsEngulfing)
threeBlackCrowsAccumulationStrength = getPatternStrength(threeBlackCrowsAccumulation)
asianSessionThreeBlackCrowsStrength = getPatternStrength(asianSessionThreeBlackCrows)

// Create a table to act as the textbox
var table cornerTable = table.new(position.top_right, 1, 1, bgcolor = color.new(color.black, 50), border_width = 1)

// Function to update the table content
updateTable(content) =>
    table.cell(cornerTable, 0, 0, content, text_color = color.white, text_size = size.normal, text_halign = text.align_left)

// Example content (you can modify this as needed)
textboxContent = "STRATEGY STEPS\n1. Detection of the 3 Black Crows pattern in the Asian session 1H.\n2. Mark lowest low between Asia and London Sessions.\n3. Long entry ONLY after LL between 8am and 9am candles in the New York session.\n4. Exit trade on close of 10am 1H candle\n5. Use Price action patterns for validating exits."

// Update the table on each bar
updateTable(textboxContent)

//show labels with background depending on the pattern+trend strength 
plotshape(threeStarsEngulfing, title="Three Stars + Engulfing", text="3S+E\n", style=shape.labelup, location=location.belowbar, color=color.new(color.green, 100 - threeStarsEngulfingStrength), textcolor=color.white, size=size.small)
plotshape(threeBlackCrowsAccumulation, title="Three Crows + Accumulation", text="3C+A\n", style=shape.labeldown, location=location.abovebar, color=color.new(color.red, 100 - threeBlackCrowsAccumulationStrength), textcolor=color.white, size=size.small)
plotshape(asianSessionThreeBlackCrows, title="Asian Three Crows", text="A3C\n", style=shape.labeldown, location=location.abovebar, color=color.new(color.blue, 100 - asianSessionThreeBlackCrowsStrength), textcolor=color.white, size=size.small)

alertcondition(threeStarsEngulfing, title="Three Stars in the South + Engulfing Bullish", message="Three Stars in the South followed by Engulfing Bullish pattern detected.")
alertcondition(threeBlackCrowsAccumulation, title="Three Black Crows + Accumulation", message="Three Black Crows followed by price range accumulation detected.")
alertcondition(asianSessionThreeBlackCrows, title="Asian Session Three Black Crows", message="Three Black Crows detected in the first 5 hours of Asian session.")
