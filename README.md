# Index Pro v3 Stable - Specification

## 1. Overview
Index Pro v3 Stable is a comprehensive TradingView Pine Script v6 indicator tailored for index trading (specifically Indian markets: NIFTY, BANKNIFTY, SENSEX, running 09:15 - 15:30 Asia/Kolkata). It combines trend analysis, volume profiling, price action concepts (FVG, Pivots), and a rules-based entry model into a single overlay.

## 2. Core Components

### 2.1 Trend Bias (HTF ADX + EMA)
- **Fast/Slow EMA:** `emaFast` (default 9), `emaSlow` (default 21).
- **Directional Bias:** Derived from a Higher Timeframe (default 5-minute) using manual ADX calculation.
- **Rules:**
  - Bullish: `emaF > emaS` AND `+DI > -DI` AND `ADX > adxThresh`
  - Bearish: `emaF < emaS` AND `-DI > +DI` AND `ADX > adxThresh`

### 2.2 Volume & Session Profiling
- Tracks the trading session internally based on timezone.
- **Volume Profile:** Manually calculates POC (Point of Control), VAH (Value Area High), and VAL (Value Area Low) using a dynamic array of price bins.
- **Volume Surges:** Highlights bars where `volume > SMA(volume, 20) * 1.5` as high volume.

### 2.3 Price Action Levels
- **Support / Resistance (S/R):** Uses Pivot Highs/Lows (length 10). Consolidates close pivots (within 0.2 ATR) and counts hits to determine level strength.
- **Fair Value Gaps (FVG):** Tracks up to N active FVGs. Bullish FVG occurs when current low > high[2]. Bearish FVG when current high < low[2].
- **Key Strikes:** Draws psychological round numbers based on a strike step (e.g., every 50 or 100 points).

### 2.4 Trade Signal State Machine
The core signal logic relies on price retesting the Volume Profile Value Area boundaries and confirming with a strong candle.
- **LONG Flow:**
  - **State 0 (Wait):** Trend is Bullish. Price enters VAL zone (within 1.5 ATR).
  - **State 1 (Enter Zone):** Price pushes deep into the zone.
  - **State 2 (Retest):** Price rebounds to the trigger zone.
  - **State 3 (Confirm):** A strong bullish candle (close > open, body % > 55, closes above prev high) on sufficient volume triggers the `BUY` signal.
- **SHORT Flow:** Mirrors LONG, using VAH as the resistance zone.

## 3. UI and Visuals
- Plots the 5M Trend EMA.
- Draws FVG boxes, S/R lines, and Volume Profile dashed lines.
- Adds text labels ("B" / "S") for strong conviction candles.
- Displays a top-center dashboard summarizing Trend, ADX, Volume, Bias, and current Signal State.
- Configures alerts for BUY/SELL signals and strong buyer/seller activity.

## 4. Planned Architectural Improvements (v6 Agent Skills Refactor)
- **UDT Encapsulation:** S/R levels, FVG zones, and Trade Signals will be wrapped in User-Defined Types (UDT).
- **Modern Loops:** Replace `while` loops with `for...in`.
- **Drawing Scaling:** Switch `xloc.bar_index` to `xloc.bar_time` to remove historical lookback drawing limits.
- **Helper Methods:** Extract Volume Profile and Signal state logic into isolated methods.
