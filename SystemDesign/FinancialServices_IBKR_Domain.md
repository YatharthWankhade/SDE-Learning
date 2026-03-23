# Financial Services Domain – Interview Notes (IBKR JD)

## Table of Contents
1. [Capital Markets Overview](#1-capital-markets-overview)
2. [Order Types & Trade Lifecycle](#2-order-types--trade-lifecycle)
3. [Instrument Types](#3-instrument-types)
4. [Market Structure & Exchanges](#4-market-structure--exchanges)
5. [Settlement & Clearing](#5-settlement--clearing)
6. [Risk Management Basics](#6-risk-management-basics)
7. [IBKR-Specific Context](#7-ibkr-specific-context)
8. [Reference Data Systems](#8-reference-data-systems)
9. [Regulatory Basics](#9-regulatory-basics)
10. [Common Interview Questions](#10-common-interview-questions)

---

## 1. Capital Markets Overview

### Market Segments
| Segment | Description | Examples |
|---|---|---|
| **Primary Market** | Companies raise capital (IPOs, bonds) | NYSE IPO, SEBI listing |
| **Secondary Market** | Trading existing securities between investors | NYSE, NASDAQ, BSE, NSE |
| **OTC Market** | Bilateral trading outside exchanges | Forex, interest rate swaps |

### Key Participants
```
Retail Investors   → buy/sell via brokers like IBKR
Institutional     → hedge funds, pension funds, mutual funds
Market Makers     → provide liquidity (bid/ask quotes)
Brokers           → execute trades on behalf of clients (IBKR's role)
Custodians        → hold securities on behalf of clients
Clearing Houses   → DTCC, LCH – manage settlement risk
Regulators        → SEC (US), FCA (UK), SEBI (India)
```

---

## 2. Order Types & Trade Lifecycle

### Order Types
| Order | Description | Use Case |
|---|---|---|
| **Market Order** | Execute immediately at best available price | Urgent; liquidity sufficient |
| **Limit Order** | Execute at specified price or better | Price-sensitive; patient |
| **Stop Order** | Becomes market order when price hits stop | Loss limiting |
| **Stop-Limit** | Becomes limit order when stop hit | More controlled stop |
| **GTC** | Good Till Cancelled | Persistent limit orders |
| **IOC** | Immediate or Cancel | Fill what you can now |
| **FOK** | Fill or Kill | All or nothing, immediately |

### Trade Lifecycle
```
1. ORDER ENTRY
   Client submits order → IBKR validates (account, margin, symbol)

2. PRE-TRADE CHECKS
   Risk checks: position limits, margin availability, regulatory halts

3. ROUTING
   Smart Order Routing (SOR) → best execution across exchanges

4. EXECUTION
   Order matches with counterparty on exchange

5. CONFIRMATION
   Execution report sent to client (FIX protocol / API notification)

6. CLEARING
   DTCC / central counterparty guarantees settlement

7. SETTLEMENT
   Exchange of securities and cash
   T+1 (US equities), T+2 (EU equities), T+0 (crypto)

8. RECONCILIATION
   Broker reconciles positions and cash with custodian
```

### FIX Protocol (Financial Information eXchange)
```
Industry standard messaging protocol for trade communications
Key message types:
  35=D  → New Order Single
  35=8  → Execution Report
  35=G  → Order Cancel/Replace Request
  35=F  → Order Cancel Request
  35=0  → Heartbeat

Example FIX message:
8=FIX.4.4|35=D|49=CLIENT|56=IBKR|54=1|55=AAPL|38=100|40=2|44=175.00
          (type)                   (buy)  (symbol)(qty)(limit)(price)
```

---

## 3. Instrument Types

### Equities
```
Common Stock      → ownership share in company; voting rights; dividends
Preferred Stock   → priority dividends; no voting rights
ETF               → basket of assets; trades like stock (SPY, QQQ)
ADR               → foreign stock listed on US exchange
```

### Fixed Income (Bonds)
```
Government Bond   → US Treasury; low risk
Corporate Bond    → company debt; higher yield/risk
Yield = Coupon / Price  (inverse relationship: price ↑ → yield ↓)
Duration          → sensitivity to interest rate changes
```

### Derivatives
```
Future           → obligation to buy/sell at future date/price
Option           → right (not obligation) to buy (call) or sell (put)
                   Premium = Intrinsic Value + Time Value
                   In-the-money (ITM): exercising is profitable
                   Greeks: Delta, Gamma, Theta (time decay), Vega (volatility)
Swap             → exchange of cash flows (interest rate swap, FX swap)
```

### Identifiers (Critical for Reference Data)
| Identifier | Description |
|---|---|
| **ISIN** | International Securities Identification Number (12-char) |
| **CUSIP** | US/Canada identifier (9-char) |
| **SEDOL** | UK identifier (7-char) |
| **TICKER** | Exchange-specific symbol (e.g., AAPL, GOOGL) |
| **FIGI** | Bloomberg's open financial instrument global identifier |
| **LEI** | Legal Entity Identifier (20-char, for firms) |

---

## 4. Market Structure & Exchanges

### US Market Structure
```
NYSE (New York Stock Exchange)    → largest by market cap
NASDAQ                            → tech-heavy, electronic
CBOE (Chicago Board Options)      → options exchange
BATS / Cboe BZX                   → electronic exchange
Dark Pools                        → private exchanges; large block trades away from public market
```

### Market Microstructure
```
Bid Price  = highest price buyer will pay
Ask Price  = lowest price seller will accept
Spread     = Ask - Bid  (market maker profit; measure of liquidity)
Depth      = volume available at each price level (Level 2 data)
VWAP       = Volume-Weighted Average Price (benchmark for execution quality)
TWAP       = Time-Weighted Average Price
```

### Circuit Breakers (US Equities)
```
Level 1: S&P 500 drops 7%  → 15-minute trading halt
Level 2: S&P 500 drops 13% → 15-minute trading halt
Level 3: S&P 500 drops 20% → trading halted for remainder of day
```

---

## 5. Settlement & Clearing

### DTCC (Depository Trust and Clearing Corporation)
```
NSCC (National Securities Clearing Corp) → nets trades (reduces settlement obligations)
DTC  (Depository Trust Company)          → holds securities; book-entry transfers

T+1 settlement (US, effective May 2024):
  Trade Date: March 10 (Monday)
  Settlement Date: March 11 (Tuesday)
  → Securities and cash exchanged
```

### Concepts
```
DVP  = Delivery vs Payment  → securities delivered only when payment received
CCP  = Central Counterparty → stands between buyer and seller; manages default risk
Margin Call → broker demands more funds when position value drops
Mark-to-Market → daily revaluation of positions to current market prices
```

---

## 6. Risk Management Basics

### Types of Risk
| Risk | Description | Managed By |
|---|---|---|
| **Market Risk** | Loss from price movements | VaR, Delta-hedging |
| **Credit Risk** | Counterparty default | Credit ratings, collateral |
| **Liquidity Risk** | Can't exit position at fair price | Position limits, bid-ask spread |
| **Operational Risk** | System failures, human errors | Controls, DR/BCP |
| **Regulatory Risk** | Non-compliance with rules | Compliance team |

### VaR (Value at Risk)
```
"95% VaR of $1M over 1 day" = 
→ 95% chance of losing less than $1M in one day
→ 5% chance of losing more than $1M in one day

Methods:
- Historical simulation: use past returns
- Monte Carlo: simulate thousands of scenarios
- Parametric VaR: assumes normal distribution
```

### Margin
```
Initial Margin  → upfront deposit to open position (e.g., 10% of position value)
Maintenance Margin → minimum equity to keep position open (e.g., 25%)
Margin Call     → when equity falls below maintenance → must deposit or liquidate
Pattern Day Trader (PDT) Rule → US: need $25K to make > 3 day trades per week
```

---

## 7. IBKR-Specific Context

### Interactive Brokers Overview
```
One of world's largest electronic brokerage firms
Clients: retail investors, hedge funds, institutions, advisors
Technology-driven: automated systems, low commissions

Products:
- TWS (Trader Workstation) – desktop trading platform
- IBKR Client Portal – web portal
- IBKR Mobile
- API access (FIX, REST, WebSocket) for algorithmic trading

Technology Stack (known):
- Java (backend systems)
- Oracle databases (reference data, trade data)
- Kafka (event streaming)
- Linux infrastructure
```

### Reference Database Systems at IBKR
```
Reference Database = Master Data for:
  - Instruments (stocks, futures, options, ETFs) – ISIN, CUSIP, ticker, exchange
  - Accounts & clients
  - Counterparties
  - Exchanges & markets
  - Corporate actions (dividends, splits, mergers)
  - Pricing (end-of-day, intraday)
  - Settlement calendars

Key challenges:
  - Instrument universe: 1M+ instruments globally
  - Real-time updates: corporate actions, exchange changes
  - Data quality: multiple conflicting external sources
  - Historical data: SCD Type 2 for audit trail
  - Global coverage: 150+ markets, 30+ currencies
```

---

## 8. Reference Data Systems

### Corporate Actions
```
Types:
  Cash Dividend    → company pays cash per share; ex-div date matters
  Stock Split      → 2:1 split → 2x shares, half the price
  Reverse Split    → 1:10 reverse → fewer shares, 10x price
  Merger/Acquisition → company absorbed; old symbol may cease
  Rights Offering  → existing shareholders buy new shares at discount
  Spin-off         → subsidiary becomes separate public company

When processing corporate actions in code:
→ Adjust historical prices (adjusted close price)
→ Update position quantities
→ Maintain audit trail
```

### Price Data
```
EOD Prices:       official closing prices from exchanges
Intraday Ticks:   ms-resolution quotes and trades
Adjusted Prices:  historical prices corrected for splits and dividends
Bid/Ask/Last:     market snapshot data
OHLCV:            Open, High, Low, Close, Volume (OHLC bars)
```

---

## 9. Regulatory Basics

| Regulation | Region | Purpose |
|---|---|---|
| **MiFID II** | EU | Best execution, trade reporting |
| **Dodd-Frank** | US | Post-GFC derivatives reform |
| **SEC Rule 15c3-3** | US | Customer protection (segregate client funds) |
| **FINRA** | US | Broker-dealer oversight |
| **Basel III** | Global | Bank capital requirements |
| **GDPR** | EU | Data privacy |
| **KYC/AML** | Global | Know Your Customer, Anti-Money Laundering |

### Best Execution (IBKR's SMART routing)
```
Brokers are required to seek best execution for client orders:
→ Best price
→ Fastest execution
→ Highest likelihood of execution
→ Minimal market impact

IBKR's smart router evaluates multiple venues
simultaneously and routes to the best option.
```

---

## 10. Common Interview Questions

| Question | Key Answer |
|---|---|
| **What is a broker-dealer?** | Firm that buys/sells securities for clients (broker) and its own account (dealer); IBKR is both |
| **What is settlement risk?** | Risk that one party delivers but the other doesn't pay (solved by CCP/DVP) |
| **What happens on ex-dividend date?** | Stock price adjusts down by dividend amount; must own shares before ex-date to receive dividend |
| **What is the bid-ask spread?** | Difference between buy and sell price; profit for market makers; cost for traders |
| **What is short selling?** | Borrowing shares, selling them, buying back at lower price; profit from decline |
| **What is a margin call?** | Broker demands more funds when position losses reduce equity below maintenance requirement |
| **What is mark-to-market?** | Daily revaluation of positions at current market prices; futures settle daily in cash |
| **What is T+1 settlement?** | Trades settle one business day after trade date (US equities since May 2024) |
| **What is a dark pool?** | Private exchange for large block trades; pre-trade transparent to participants only |
| **Why is reference data important at IBKR?** | Every trade, risk calculation, reporting, and client position depends on accurate instrument/account reference data; errors cascade across all downstream systems |
