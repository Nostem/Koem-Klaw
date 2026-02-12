# Polymarket Copy Trading Monitor

## Overview
Automated monitoring system for Polymarket copy trading via PolyGun Telegram bot. Tracks wallet performance, scores copied wallets, generates daily/weekly reports, and serves a mobile-friendly dashboard.

## Architecture

```
polymarket-monitor/
├── config.json              # Tracked wallets, scoring weights, thresholds
├── scripts/
│   ├── fetch_wallet_trades.py   # Pull trade history per wallet from Polymarket API
│   ├── fetch_portfolio.py       # Pull current positions per wallet
│   ├── score_wallets.py         # Calculate composite wallet scores
│   ├── daily_report.py          # Generate daily PnL snapshot
│   └── weekly_report.py         # Deep analysis + rotation recommendations
├── dashboard/
│   └── index.html               # Static HTML/CSS/JS dashboard (mobile-friendly)
├── data/
│   ├── wallets/{address}/trades.json   # Per-wallet trade history
│   ├── wallet_scores.json              # Latest composite scores
│   ├── reports/YYYY-MM-DD.json         # Daily reports (one per day, never overwritten)
│   └── weekly_report.json              # Latest weekly analysis
└── README.md
```

## Data Sources
- **Polymarket Data API**: `https://data-api.polymarket.com/activity?user={wallet}` — trade history
- **Polymarket Positions**: `https://data-api.polymarket.com/positions?user={wallet}` — current positions
- **Polymarket Leaderboard**: `/v1/leaderboard` — discover top-performing wallets
- **Chain**: Polygon (MATIC)

## Wallet Scoring (Composite)
| Metric | Weight | Description |
|--------|--------|-------------|
| Win Rate | 40% | % of profitable trades |
| Avg ROI | 30% | Average return per trade |
| Consistency | 20% | Variance-adjusted returns |
| Recency | 10% | Recent performance weighted higher |

## Auto-Remove Triggers
- Win rate drops below 40% over rolling 7-day window
- 3+ consecutive losses
- No activity for 7+ days

## Dashboard
- Pure static HTML/CSS/JS — no build tools, no npm
- 4 tabs: Overview, Wallet Leaderboard, Daily History, Weekly Analysis
- Mobile-responsive
- GitHub Pages compatible
- Loads data from JSON files

## Cron Schedule
| Job | Schedule | Model | Output |
|-----|----------|-------|--------|
| Daily Report | 0500 EST daily | Grok 4.1 | `data/reports/YYYY-MM-DD.json` + Discord summary |
| Weekly Report | 0500 EST Sunday | Grok 4.1 | `data/weekly_report.json` + Discord analysis |

## Key Design Decisions
- **Daily reports are append-only**: Each day creates a new `YYYY-MM-DD.json` file, never overwrites
- **Full history retained**: Future option for rolling 30-day window, but keeping all for now
- **Flat JSON storage**: No database needed, everything is simple JSON files
- **Static dashboard**: Can be hosted on GitHub Pages for phone access

## Config
Edit `config.json` to:
- Add/remove tracked wallet addresses
- Adjust scoring weights
- Set alert thresholds

## User's Wallet
`0xd19bff5dd8e408c55f8ef21facfb0d82632c8054`

## Discord Integration
- Agent: `polymarket-agent` (Grok 4.1)
- Channel: `1471501314939814018`
- requireMention: false

## Setup
1. Ensure Python 3 + `requests` module available
2. Add wallet addresses to `config.json`
3. Run scripts manually or via cron
4. Open `dashboard/index.html` or deploy to GitHub Pages

## Scripts Usage
```bash
# Fetch trades for all tracked wallets
python3 scripts/fetch_wallet_trades.py

# Fetch current portfolio positions
python3 scripts/fetch_portfolio.py

# Score all wallets
python3 scripts/score_wallets.py

# Generate today's daily report
python3 scripts/daily_report.py

# Generate weekly analysis
python3 scripts/weekly_report.py
```
