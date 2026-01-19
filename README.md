# TradingView Pine Script v5 Indicator Suite

Professional-grade TradingView Pine Script v5 indicators for advanced technical analysis and trading decision support.

## 📋 Features

- **Multi-Timeframe Analysis**: Confluence scanning across multiple timeframes
- **Smart Money Concepts**: Order blocks, FVG, liquidity sweeps, BOS/CHoCH detection
- **Volume Profile**: Session-based POC, VAH, VAL with visual zones
- **Market Structure**: Swing highs/lows, trend direction, market structure labels
- **Master Dashboard**: Multi-symbol screener combining all signals
- **Anti-Repaint Design**: Proper lookahead settings and barstate confirmation
- **Customizable**: Full control over colors, periods, and sensitivity

## 🗂️ Project Structure

```
tradingview-indicators/
├── libraries/              # Reusable Pine Script libraries
│   ├── TimeFrameUtils.pine       # MTF utilities and anti-repaint helpers
│   └── CommonSignals.pine        # Standardized signal generation
├── indicators/            # Production-ready indicators
│   ├── mtf_confluence_scanner.pine    # Multi-timeframe confluence detector
│   ├── smart_money_concepts.pine      # Smart money analysis
│   ├── volume_profile.pine            # Volume profile analysis
│   ├── market_structure.pine          # Market structure identification
│   └── master_dashboard.pine          # Multi-indicator dashboard
├── strategies/            # Trading strategies (planned)
└── docs/                  # Additional documentation
```

## 🚀 Setup & Usage

### Step 1: Open TradingView
1. Go to [TradingView](https://www.tradingview.com)
2. Open the Pine Editor (Alt+E or click Pine Editor at bottom)

### Step 2: Load Libraries First
Libraries must be published before indicators can use them:

1. **Copy `libraries/TimeFrameUtils.pine`** → Paste into Pine Editor
2. Click **"Publish Script"** → Select **"Publish as Library"**
3. Set visibility (Private/Invite-only recommended)
4. Note your TradingView username (needed for imports)

5. **Repeat for `libraries/CommonSignals.pine`**

### Step 3: Load Indicators
1. Copy desired indicator file (e.g., `indicators/mtf_confluence_scanner.pine`)
2. Paste into Pine Editor
3. **Update import statements** with your TradingView username:
   ```pine
   import YourUsername/TimeFrameUtils/1 as TF
   import YourUsername/CommonSignals/1 as Sig
   ```
4. Click **"Add to Chart"** or **"Save"**

### Step 4: Configure
- Click the indicator name on chart
- Select "Settings" (gear icon)
- Customize timeframes, colors, thresholds
- Enable/disable alerts

## 📊 Indicator Details

### 1. MTF Confluence Scanner
Scans multiple timeframes for RSI, MACD, and Stochastic alignment.
- **Signals**: Buy/Neutral/Sell on each timeframe
- **Confluence Score**: Sum of aligned signals
- **Alerts**: Triggered when confluence threshold reached
- **Anti-Repaint**: Confirmed bar signals only

### 2. Smart Money Concepts
Identifies institutional trading patterns.
- Order blocks (bullish/bearish)
- Fair Value Gaps (FVG)
- Liquidity sweeps
- Break of Structure (BOS) / Change of Character (CHoCH)

### 3. Volume Profile
Session-based volume distribution analysis.
- Point of Control (POC)
- Value Area High (VAH) / Low (VAL)
- Visual price zones
- Session segmentation

### 4. Market Structure
Automated swing detection and trend analysis.
- Swing highs/lows identification
- HH/HL/LH/LL labels
- Trend direction classification
- Support/resistance levels

### 5. Master Dashboard
Multi-symbol screener combining all indicators.
- Real-time signal aggregation
- Symbol comparison table
- Customizable watchlist
- One-glance market overview

## ⚙️ Configuration Tips

### Timeframe Selection
- **Scalping**: 1m, 5m, 15m, 1h
- **Day Trading**: 15m, 1h, 4h, D
- **Swing Trading**: 1h, 4h, D, W
- **Position Trading**: 4h, D, W, M

### Performance Optimization
- Limit timeframes to 3-5 maximum
- Use higher timeframes on lower-power devices
- Disable unused visual elements

### Alert Setup
1. Right-click chart → "Add Alert"
2. Select indicator + condition
3. Configure notification method
4. Set expiration and frequency

## 🔧 Customization

All indicators support extensive customization:
- **Colors**: Match your chart theme
- **Periods**: RSI (default 14), MACD (12,26,9), Stochastic (14,3,3)
- **Thresholds**: Overbought/oversold levels
- **Sensitivity**: Divergence detection, structure breaks
- **Display**: Toggle visual elements on/off

## 📝 Best Practices

1. **Always use confirmed bars** for live trading (enable anti-repaint)
2. **Backtest thoroughly** on your preferred symbols and timeframes
3. **Combine multiple indicators** for higher probability setups
4. **Use proper risk management** - indicators are tools, not guarantees
5. **Paper trade first** before using real capital

## ⚠️ Important Notes

- **No Repainting**: All indicators use `lookahead=barmerge.lookahead_off`
- **Security Limits**: TradingView limits `request.security()` calls (40 max)
- **Calculation Limits**: Complex indicators may timeout on very old data
- **Premium Features**: Some features require TradingView Premium for faster refresh

## 🤝 Contributing

This is a professional indicator suite. Contributions welcome:
1. Fork the repository
2. Create feature branch
3. Test thoroughly on TradingView
4. Submit pull request with examples

## 📄 License

MIT License - see LICENSE file for details

## 🐛 Troubleshooting

**Compilation Error "Cannot find library"**
- Ensure libraries are published first
- Update import statements with your username
- Check library version numbers

**"Script has too many calculations"**
- Reduce number of timeframes
- Increase chart timeframe
- Simplify visual elements

**Alerts not triggering**
- Enable "alertcondition" in settings
- Check alert expiration date
- Verify threshold values

**Signals repainting**
- Enable "Wait for bar close" option
- Use `barstate.isconfirmed` mode
- Avoid `lookahead_on` mode

## 📞 Support

For issues, feature requests, or questions:
- GitHub Issues: [Create Issue](../../issues)
- TradingView: Test on demo account first
- Documentation: See `/docs` folder for advanced guides

---

**Disclaimer**: These indicators are for educational and informational purposes only. Not financial advice. Trade at your own risk.
