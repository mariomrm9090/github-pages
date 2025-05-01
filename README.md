//@version=5
indicator("Webby\'s Bob Marley Off-High (Yellow Zone Only)", shorttitle = 'WBM Yellow Only')

//----------settings----------//
select  = input.string('Percent', '% or ATR off High', ['Percent', 'ATR'])
atrLen  = input.int(21, 'ATR Length')
src     = input.source(close, 'Source to Meassure', tooltip = 'Source for meassurement. Example: if low is selected the indicator will meassure how far the bars low is from the 52 week high')
lineCol = input.color(color.blue, 'Off 52 Week High Line Color')
col50   = input.color(color.fuchsia, 'Off 50 Day High Line Color')
col18   = input.color(color.black, 'Off 18 Month High Line Color')
z2Col   = input.color(color.yellow, 'Yellow Zone Background Color', tooltip = 'For % this zone is 8-15%, ATR is 4-8')
zoneTransparency = input.int(80, 'Zone Transparency', minval=0, maxval=100, tooltip='Higher values = more transparent')
off     = input.bool(true, 'Cut off', tooltip = 'If selected anything more than 15% or 8 ATR off the high will be drawn at bottom level to keep the scaling and plot in the zones')

//----------caluclations----------//
yearhigh    = ta.highest(high, 251)
high50Day   = ta.highest(high, 49)
high18M     = ta.highest(high, 375)
atr         = ta.atr(atrLen)
distAway    = ((src - yearhigh) / yearhigh) * 100
distAway50  = ((src - high50Day) / high50Day) * 100
distAway18  = ((src - high18M) / high18M) * 100
distATR     = (src - yearhigh) / atr
distATR50   = (src - high50Day) / atr
distATR18   = (src - high18M) / atr

// Apply cutoff and stay within boundaries for percent
if distAway < -15 and off
    distAway := -15
if distAway > 0
    distAway := 0
    
if distAway50 < -15 and off
    distAway50 := -15
if distAway50 > 0
    distAway50 := 0

if distAway18 < -15 and off
    distAway18 := -15
if distAway18 > 0
    distAway18 := 0

// Apply cutoff and stay within boundaries for ATR
if distATR < -8 and off
    distATR := -8
if distATR > 0
    distATR := 0

if distATR50 < -8 and off
    distATR50 := -8
if distATR50 > 0
    distATR50 := 0

if distATR18 < -8 and off
    distATR18 := -8
if distATR18 > 0
    distATR18 := 0

// Define yellow zone thresholds
yellowZoneUpperThreshold = select == 'Percent' ? -8.0 : -4.0
yellowZoneLowerThreshold = select == 'Percent' ? -15.0 : -8.0

// Check if we're crossing into the yellow zone (from above)
crossingIntoYellowZone = false
if select == 'Percent'
    crossingIntoYellowZone := distAway[1] > yellowZoneUpperThreshold and distAway <= yellowZoneUpperThreshold
else
    crossingIntoYellowZone := distATR[1] > yellowZoneUpperThreshold and distATR <= yellowZoneUpperThreshold

// Create a state variable to track if we've crossed into the yellow zone
var hasTriggered = false

// Set the trigger if we cross into the yellow zone
if crossingIntoYellowZone
    hasTriggered := true

// Check if each indicator is in the yellow zone
inYellowZone_52w = false
inYellowZone_50d = false
inYellowZone_18m = false

if select == 'Percent'
    inYellowZone_52w := distAway <= yellowZoneUpperThreshold and distAway > yellowZoneLowerThreshold
    inYellowZone_50d := distAway50 <= yellowZoneUpperThreshold and distAway50 > yellowZoneLowerThreshold
    inYellowZone_18m := distAway18 <= yellowZoneUpperThreshold and distAway18 > yellowZoneLowerThreshold
else
    inYellowZone_52w := distATR <= yellowZoneUpperThreshold and distATR > yellowZoneLowerThreshold
    inYellowZone_50d := distATR50 <= yellowZoneUpperThreshold and distATR50 > yellowZoneLowerThreshold
    inYellowZone_18m := distATR18 <= yellowZoneUpperThreshold and distATR18 > yellowZoneLowerThreshold

// Display conditions for each plot - only show when that specific indicator is in yellow zone AND after crossing into it
shouldDisplay_52w = hasTriggered and inYellowZone_52w
shouldDisplay_50d = hasTriggered and inYellowZone_50d
shouldDisplay_18m = hasTriggered and inYellowZone_18m

//----------plots----------//
plot(shouldDisplay_52w ? distAway : na, '% Off 52 Week High', lineCol, display = select == 'Percent' ? display.all : display.none)
plot(shouldDisplay_50d ? distAway50 : na, '% Off 50 Day High', col50, display = select == 'Percent' ? display.all : display.none)
plot(shouldDisplay_18m ? distAway18 : na, '% Off 18 Month High', col18, display = select == 'Percent' ? display.all : display.none)
plot(shouldDisplay_52w ? distATR : na, 'ATR Off 52 Week High', lineCol, display = select == 'ATR' ? display.all : display.none)
plot(shouldDisplay_50d ? distATR50 : na, 'ATR Off 50 Day High', col50, display = select == 'ATR' ? display.all : display.none)
plot(shouldDisplay_18m ? distATR18 : na, 'ATR Off 18 Month High', col18, display = select == 'ATR' ? display.all : display.none)

//----------lines for fill----------//
// Only create the yellow zone boundaries
z2Top = hline(select == 'Percent' ? -8.0 : -4.00, 'Yellow Zone Top', color.new(color.yellow,100))
z2Btm = hline(select == 'Percent' ? -15.0 : -8.00, 'Yellow Zone Bottom', color.new(color.yellow,100))

//----------fills----------//
// Apply user-defined transparency to zone color
z2Color = color.new(z2Col, zoneTransparency)

// Only fill the yellow zone
fill(z2Top, z2Btm, z2Color)

// Add an alert condition for yellow zone crossing
alertcondition(crossingIntoYellowZone, "Yellow Zone Crossing", "Price has crossed into the yellow zone")


