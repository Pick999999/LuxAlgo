# Project Context: SMC (Smart Money Concepts) Indicator

## Overview
This project focuses on the development and usage of `smc.pine`, a comprehensive Pine Script v5 indicator designed to identify and visualize **Smart Money Concepts (SMC)** on TradingView charts. The script aids traders in recognizing complex market structures, liquidity zones, and institutional order flow.

## Comprehensive Feature List

The `smc.pine` script includes a wide array of features beyond just CHoCH and BOF. Below is the complete list of capabilities found in the code:

### 1. Market Structure & Trend
- **Swing Structure**: Identifies major structural points.
    - **BOS (Break of Structure)**: Trend continuation.
    - **CHoCH (Change of Character)**: Trend reversal.
    - **Swing Points Labels**: Marks pivots as **HH** (Higher High), **LL** (Lower Low), **LH** (Lower High), **HL** (Higher Low). *This matches your "new low/new high" requirement.*
    - **Strong/Weak Highs & Lows**: Highlights significant swing points (Safe/Weak points).
- **Internal Structure**: Identifies minor structural points within the swing structure.
    - **Internal BOS & CHoCH**.
- **Trend Coloring**: Optionally colors candles based on the identified internal structure trend.

### 2. Institutional Zones (Order Flow)
- **Order Blocks (OB)**:
    - **Types**: Internal OBs and Swing OBs.
    - **Visualization**: Boxes extending from the OB candle.
    - **Mitigation Logic**: Removes OBs when price breaches them (configurabled by Close or High/Low).
    - **Filtering**: Options to filter OBs by volatility (ATR or Range).
- **Fair Value Gaps (FVG)**:
    - Automatically detects imbalances (Bullish/Bearish).
    - Filters insignificant gaps based on width thresholds.
    - Auto-extends FVG boxes.

### 3. Liquidity & Levels
- **Equal Highs/Lows (EQH/EQL)**:
    - Identifies relative equal highs/lows which act as liquidity pools.
    - Configurable sensitivity threshold.
- **Multi-Timeframe (MTF) Levels**:
    - Displays **Daily**, **Weekly**, and **Monthly** High/Low levels on the chart.
    - Configurable line styles (Solid, Dashed, Dotted).

### 4. Premium & Discount Zones
- Plots a standard fibonacci-like range based on the current swing leg.
- **Premium Zone** (Expensive, looking for sells).
- **Discount Zone** (Cheap, looking for buys).
- **Equilibrium Zone** (Midpoint).

### 5. Alerts
- Extensive alert conditions built-in for automation:
    - Internal/Swing BOS & CHoCH.
    - Internal/Swing Order Block Breakouts.
    - Equal Highs/Lows.
    - Bullish/Bearish FVGs.

## Files

### Original PineScript
- **`smc.pine`**: The original Pine Script v5 source code (LuxAlgo).

### JavaScript Implementation (for LightweightCharts 4.2+)

#### Core Files (ES6 Modules)
- **`js/smc/SMCIndicator.js`**: Main indicator class with full SMC calculation logic.
- **`js/smc/SMCChartRenderer.js`**: Renderer class for drawing SMC elements.
- **`js/smc/index.js`**: Module index that exports all classes.

#### Standalone Files (No ES6 Modules - works with file:// protocol)
- **`js/smc/SMCIndicator.standalone.js`**: Standalone version of SMCIndicator.
- **`js/smc/DerivSMC.js`**: Deriv.com API integration class.

#### Demo Pages
- **`smc_demo.html`**: Basic demo with sample data.
- **`smc_deriv.html`**: Full Deriv.com integration with live data.

#### Styles
- **`css/smc_deriv.css`**: Styles for Deriv integration page.

---

## Deriv.com API Integration

### Features
| Feature | Description |
|---------|-------------|
| **Symbol Selection** | Volatility Indices (R_100, R_75, R_50, R_25, R_10, 1HZ100V), Forex (EUR/USD, GBP/USD, USD/JPY) |
| **Timeframe** | 1M, 5M, 15M, 1H, 4H, 1D |
| **Date Range** | Start Date / End Date selection |
| **Historical Data** | Load past candles by date range or count |
| **Live Subscription** | Real-time tick updates via WebSocket |
| **SMC Analysis** | Automatic analysis on data load and new candles |

### Usage
1. Open `smc_deriv.html` in browser
2. Select Symbol and Timeframe
3. Set Start/End Date or Candle Count
4. Click **📥 Load Data** to fetch historical data
5. Click **▶️ Live** to subscribe to real-time updates
6. Toggle SMC features (Swing Points, CHoCH/BOS, Order Blocks, FVG, EQH/EQL)
7. Click **🔄 Analyze** to re-run analysis

### DerivSMC Class API
```javascript
// Auto-initialized on page load
const derivSMC = new DerivSMC();

// Manual methods
derivSMC.loadHistoricalData();  // Fetch historical candles
derivSMC.toggleLive();          // Start/stop live subscription
derivSMC.analyze();             // Run SMC analysis
```

### Deriv WebSocket API
- **Endpoint**: `wss://ws.binaryws.com/websockets/v3?app_id=1089`
- **Request**: `ticks_history` with `granularity` and `style: 'candles'`
- **Live**: `subscribe: 1` for real-time OHLC updates

---

## SMCIndicator API Reference

### Configuration Options
```javascript
const config = {
    swingLength: 50,              // Swing structure lookback period
    internalLength: 5,            // Internal structure lookback period
    showInternalStructure: true,  // Enable internal structure detection
    showSwingStructure: true,     // Enable swing structure detection
    showOrderBlocks: true,        // Enable order block detection
    maxOrderBlocks: 5,            // Max order blocks to store per type
    showFVG: true,                // Enable Fair Value Gap detection
    showEqualHL: true,            // Enable Equal Highs/Lows detection
    equalHLLength: 3,             // EQH/EQL lookback period
    equalHLThreshold: 0.1,        // EQH/EQL sensitivity (ATR multiplier)
    showPremiumDiscount: true,    // Enable Premium/Discount zones
    orderBlockFilter: 'atr',      // 'atr' or 'range'
    orderBlockMitigation: 'highlow', // 'close' or 'highlow'
    atrPeriod: 200                // ATR calculation period
};
```

### Methods
| Method | Returns | Description |
|--------|---------|-------------|
| `calculate(ohlcvData)` | `this` | Process OHLCV data and calculate all SMC indicators |
| `getStructures(filter)` | `StructurePoint[]` | Get CHoCH/BOS points |
| `getSwingPoints(filter)` | `SwingPoint[]` | Get HH/HL/LH/LL points |
| `getOrderBlocks(filter)` | `OrderBlock[]` | Get Order Blocks |
| `getFairValueGaps(filter)` | `FairValueGap[]` | Get FVGs |
| `getEqualHighsLows(filter)` | `EqualHighLow[]` | Get EQH/EQL |
| `getTrend(level)` | `'bullish'|'bearish'|'neutral'` | Get current trend |
| `getAllResults()` | `Object` | Get all results in one object |

---

## User Objectives
The user aims to utilize this script to fully capture SMC analysis, including:
- **Trend Changes**: CHoCH.
- **Trend Continuation**: BOF/BOS.
- **Support & Resistance**: Swing points (HH, HL, LH, LL), MTF Levels, EQH/EQL.
- **Price Action & Zones**: Order Blocks, FVGs, Premium/Discount zones.
- **Live Trading**: Real-time analysis with Deriv.com data.
