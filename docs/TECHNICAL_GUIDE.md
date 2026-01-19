# Technical Implementation Guide

## Architecture Overview

This indicator suite follows a modular architecture with reusable libraries and standalone indicators.

```
System Architecture:
┌─────────────────────────────────────────────┐
│           Library Layer                      │
│  ┌──────────────────┐  ┌─────────────────┐ │
│  │ TimeFrameUtils   │  │ CommonSignals   │ │
│  │ - MTF helpers    │  │ - Signal std    │ │
│  │ - Anti-repaint   │  │ - Confluence    │ │
│  │ - Performance    │  │ - Formatting    │ │
│  └──────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────┘
                    ▲
                    │ import
                    │
┌───────────────────┴─────────────────────────┐
│           Indicator Layer                    │
│  ┌────────────┐  ┌────────────┐            │
│  │ MTF Scanner│  │    SMC     │            │
│  └────────────┘  └────────────┘            │
│  ┌────────────┐  ┌────────────┐            │
│  │Vol Profile │  │  Structure │            │
│  └────────────┘  └────────────┘            │
│        ┌────────────────┐                   │
│        │   Dashboard    │                   │
│        └────────────────┘                   │
└─────────────────────────────────────────────┘
```

## Anti-Repaint Design

### Problem: Repainting
Repainting occurs when indicator signals change after a bar closes. This creates misleading backtests and unreliable live signals.

### Solutions Implemented

#### 1. **lookahead=barmerge.lookahead_off** (Critical)
```pine
request.security(symbol, tf, expression,
    lookahead=barmerge.lookahead_off,  // NEVER use lookahead_on
    gaps=barmerge.gaps_off)
```

#### 2. **Confirmed Bar Mode** (Recommended for Live Trading)
```pine
// Option A: Use [1] offset
confirmedValue = request.security(symbol, tf, close[1],
    lookahead=barmerge.lookahead_off)

// Option B: Use barstate.isconfirmed filter
if barstate.isconfirmed
    // Only trigger signals on confirmed bars
```

#### 3. **Pivot Confirmation**
```pine
// WRONG: Repaint risk
pivotHigh = ta.pivothigh(high, lookback, 0)  // Right bars = 0

// CORRECT: Confirmed pivots only
pivotHigh = ta.pivothigh(high, lookback, lookback)  // Equal left/right
```

#### 4. **Divergence Detection**
```pine
// Use confirmed pivots with lookback offset
// This ensures pivot is finalized before detection
if pricePivotLow and indicatorPivotLow
    prevPriceLow = ta.valuewhen(condition, price, 1)  // Previous confirmed
```

### Repaint Risk Matrix

| Feature | Risk Level | Mitigation |
|---------|-----------|------------|
| request.security() with lookahead_on | ❌ HIGH | Use lookahead_off |
| Real-time bar signals | ⚠️ MEDIUM | Use barstate.isconfirmed |
| Pivot detection (right=0) | ⚠️ MEDIUM | Use equal left/right bars |
| Variable-length pivots | ❌ HIGH | Use fixed lookback |
| [0] on HTF data | ⚠️ MEDIUM | Use [1] for confirmation |

## Performance Optimization

### request.security() Limits

TradingView imposes limits on `request.security()` calls:
- **40 calls per script** (typical limit)
- Exceeding causes "Too many security calls" error

### Optimization Strategy: Tuple Returns

**WRONG (Inefficient):**
```pine
rsi1 = request.security(symbol, tf, ta.rsi(close, 14))
macd1 = request.security(symbol, tf, ta.macd(...))
stoch1 = request.security(symbol, tf, ta.stoch(...))
// 3 security calls per timeframe × 4 timeframes = 12 calls
```

**CORRECT (Optimized):**
```pine
[rsi1, macd1, stoch1] = request.security(symbol, tf,
    [ta.rsi(close, 14), ta.macd(...), ta.stoch(...)])
// 1 security call per timeframe × 4 timeframes = 4 calls
```

### Performance Best Practices

1. **Minimize Security Calls**: Use tuple returns for multiple indicators
2. **Limit Lookback**: Set reasonable `max_bars_back` (500-1000)
3. **Array Management**: Delete old array elements beyond lookback
4. **Object Limits**: Limit lines/boxes/labels (max 50-500 depending on object)
5. **Conditional Execution**: Only calculate on necessary bars

```pine
// Good: Limit calculations
if barstate.islast or barstate.isconfirmed
    // Only calculate on relevant bars
```

## Signal Standardization

All indicators use unified signal format:
- `BUY = 1`
- `NEUTRAL = 0`
- `SELL = -1`

### Signal Generation Pattern

```pine
// RSI Example
f_rsiSignal(float rsi, float oversold, float overbought) =>
    int signal = 0
    if not na(rsi)
        signal := rsi <= oversold ? 1 : (rsi >= overbought ? -1 : 0)
    signal

// MACD Example
f_macdSignal(float histogram) =>
    int signal = 0
    if not na(histogram)
        signal := histogram > 0 ? 1 : (histogram < 0 ? -1 : 0)
    signal
```

### Confluence Scoring

```pine
// Calculate individual signals
int rsiSig = f_rsiSignal(rsi, 30, 70)
int macdSig = f_macdSignal(macdHist)
int stochSig = f_stochSignal(k, d, 20, 80)

// Sum for confluence
int confluence = rsiSig + macdSig + stochSig
// Range: -3 (all bearish) to +3 (all bullish)

// Threshold-based final signal
int finalSignal = confluence >= 2 ? 1 : (confluence <= -2 ? -1 : 0)
```

## Multi-Timeframe Analysis

### Timeframe Validation

```pine
// Check if target TF is higher than chart TF
f_isHigherTF(string tf) =>
    f_tfToMinutes(tf) > f_tfToMinutes(timeframe.period)

// Only request if valid HTF
if f_isHigherTF(tf)
    htfValue = request.security(...)
else
    htfValue = currentValue  // Use current TF
```

### Timeframe Conversion

```pine
// Robust conversion handling all formats
f_tfToMinutes(string tf) =>
    // Handles: "1", "5", "15", "60", "D", "W", "M"
    // Returns: Minutes as float
    // Examples: "15" → 15, "1H" → 60, "D" → 1440
```

## Alert System

### Alert Conditions

```pine
// Basic alert
alertcondition(
    condition=totalConfluence >= threshold,
    title="Bullish Confluence",
    message="{{ticker}}: Confluence score {{plot_0}}"
)

// Barstate-filtered alert (no repaint)
alertcondition(
    condition=signal == 1 and barstate.isconfirmed,
    title="Confirmed Buy Signal",
    message="{{ticker}}: Buy signal at {{close}}"
)
```

### Alert Message Variables

- `{{ticker}}`: Symbol ticker
- `{{close}}`, `{{high}}`, `{{low}}`, `{{open}}`: Price values
- `{{volume}}`: Current volume
- `{{plot_0}}`, `{{plot_1}}`: Plot values
- `{{interval}}`: Chart timeframe

## Testing on TradingView

### Step-by-Step Testing Process

1. **Publish Libraries First**
   ```
   1. Open libraries/TimeFrameUtils.pine
   2. Publish → Publish as Library → Private
   3. Note your username
   4. Repeat for CommonSignals.pine
   ```

2. **Update Imports in Indicators**
   ```pine
   // Replace YourUsername with your actual username
   import YourUsername/TimeFrameUtils/1 as TF
   import YourUsername/CommonSignals/1 as Sig
   ```

3. **Load Indicator**
   ```
   1. Copy indicator code
   2. Paste into Pine Editor
   3. Click "Add to Chart"
   ```

4. **Configure Settings**
   ```
   1. Click indicator name on chart
   2. Settings (gear icon)
   3. Adjust timeframes, colors, thresholds
   4. Enable "Wait for Bar Close" (anti-repaint)
   ```

5. **Verify Behavior**
   - Check signals on historical bars (should not change)
   - Watch real-time bar (signal may update until close)
   - With "Wait for Bar Close": signal appears only after bar closes
   - Without: signal may disappear (repaint)

### Common Issues & Fixes

| Issue | Cause | Solution |
|-------|-------|----------|
| "Cannot find library" | Not published | Publish library first |
| "Too many calculations" | Too many timeframes | Reduce TF count or increase chart TF |
| Signals repainting | lookahead_on or [0] on HTF | Use lookahead_off and [1] |
| Slow performance | Too many objects | Limit labels/lines/boxes |
| "Script timeout" | Heavy calculations | Reduce lookback or optimize loops |

## Indicator-Specific Notes

### MTF Confluence Scanner
- **Ready for use**: Fully implemented, production-ready
- **Key feature**: 4 timeframes, 3 indicators, confluence scoring
- **Anti-repaint**: Enabled by default with toggle
- **Performance**: Optimized with inline functions (no library dependency for standalone use)

### Smart Money Concepts
- **Status**: Skeleton with TODO blocks
- **To implement**: Order blocks, FVG, liquidity sweeps, BOS/CHoCH
- **Complexity**: High (requires arrays, state tracking, object management)
- **Estimated work**: Full implementation requires 200-300 lines of logic

### Volume Profile
- **Status**: Skeleton with TODO blocks
- **To implement**: Session detection, price binning, POC/VA calculation, histogram
- **Complexity**: High (computationally intensive, many objects)
- **Estimated work**: 300-400 lines for complete implementation

### Market Structure
- **Status**: Skeleton with TODO blocks
- **To implement**: HH/HL/LH/LL detection, trend classification, S/R levels
- **Complexity**: Medium (pivot tracking, comparison logic)
- **Estimated work**: 150-200 lines

### Master Dashboard
- **Status**: Skeleton with TODO blocks
- **To implement**: Multi-symbol scanning, signal aggregation, table rendering
- **Complexity**: High (security call limits, performance critical)
- **Estimated work**: 250-350 lines

## Next Steps for Development

### Phase 1: Complete MTF Scanner (✅ Done)
- [x] Standalone implementation
- [x] Dashboard table
- [x] Alert system
- [x] Anti-repaint features

### Phase 2: Smart Money Concepts
1. Implement order block detection
2. Add FVG identification
3. Create liquidity sweep logic
4. Implement BOS/CHoCH detection
5. Add visual overlays (boxes, lines)
6. Create comprehensive alerts

### Phase 3: Volume Profile
1. Session boundary detection
2. Price level binning algorithm
3. Volume accumulation
4. POC calculation
5. Value Area algorithm
6. Histogram rendering
7. Zone visualization

### Phase 4: Market Structure
1. Swing high/low detection
2. HH/HL/LH/LL classification
3. Trend determination
4. S/R level clustering
5. Visual labels and lines
6. Structure break alerts

### Phase 5: Master Dashboard
1. Multi-symbol data fetching
2. Indicator calculations per symbol
3. Confluence scoring
4. Dynamic table rendering
5. Sorting and filtering
6. Performance optimization (tuple returns)

## Best Practices Summary

✅ **DO:**
- Use `lookahead=barmerge.lookahead_off`
- Enable "Wait for Bar Close" for live trading
- Use tuple returns for multiple indicators
- Limit drawn objects (lines/boxes/labels)
- Test on multiple symbols and timeframes
- Validate historical signals (no changes)
- Use confirmed pivots for swing detection
- Document all parameters clearly

❌ **DON'T:**
- Use `lookahead_on` (causes repaint)
- Use `[0]` on HTF data without understanding
- Create unlimited objects (memory issues)
- Exceed 40 security() calls per script
- Use variable-length pivots
- Skip bar confirmation in live trading
- Assume indicators work on all markets
- Deploy without thorough testing

## Resources

- **Pine Script v5 Documentation**: https://www.tradingview.com/pine-script-docs/
- **Anti-Repaint Guide**: Search TradingView for "request.security repaint"
- **Performance Best Practices**: TradingView Help Center
- **Community Scripts**: Study top-rated indicators for patterns

## Support

For issues or questions:
1. Check this technical guide
2. Review README.md for setup instructions
3. Test indicator on demo account first
4. Report bugs with: symbol, timeframe, settings, screenshot
5. GitHub Issues: [Create Issue](../../issues)
