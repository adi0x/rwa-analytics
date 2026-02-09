# RWA Analytics — Real World Assets On-Chain

Tracking **$23.9B in tokenized real-world assets** across 136 protocols, 67 blockchains, and 800+ on-chain whale transfers.

**[Live Dashboard on Dune →](https://dune.com/adu0x/rwa-analytics)**

---

## What is this?

A Python ETL pipeline that extracts data from 3 APIs, transforms it into actionable analytics, and loads it into Dune for visualization.

2 years ago tokenized RWAs barely existed. Today:
- **BlackRock BUIDL:** $0 → $1.8B in 18 months
- **Ondo Finance:** $2.5B TVL (up 404% YoY)
- **Tokenized gold:** up 177% in 2025
- **40+ major financial institutions** now building tokenized products

Meanwhile 80% of the $307B stablecoin market earns nothing — tokenized Treasuries pay 3-4%.

---

## Dashboard Sections

| Section | What it shows |
|---------|--------------|
| **Market Overview** | Total TVL ($23B), protocol count (136), category breakdown |
| **Category Breakdown** | Treasuries (42%), Gold (27%), Credit (9%), Equities (7%) |
| **Chain Distribution** | Ethereum (31%), Arbitrum (11%), Solana (8%), 25+ others |
| **Whale Transfers** | Largest on-chain transfers — $55M USDY burn, $25M USYC mints |
| **Institutional Net Flow** | Mint vs burn analysis — USYC +$32M inflow, USDY -$55M outflow |
| **Token Market Caps** | Top RWA tokens ranked by market cap |

---

## Key Findings

### Institutional Rotation
- **$55M pulled out of USDY** in a single transaction (Feb 6, 2026)
- **$32M net inflow into USYC** the same week
- Institutions are rotating from Ondo → Hashnote

### Market Concentration
- Top 5 protocols control **50% of all RWA TVL**
- USYC has only **21 unique wallets** — extremely concentrated
- OUSG has **20 wallets**, STBT has **25 wallets**

### Category Trends
- Tokenized Treasuries: **$9.6B (42%)** — largest category
- Tokenized Gold: **$6.1B (27%)** — steady growth
- Tokenized Equities: **$1.5B** — up **110% in 7 days**
- Private Credit: **$2B** — declining (-5.5% weekly)

---

## Data Sources

| Source | What it provides | Endpoint |
|--------|-----------------|----------|
| **DefiLlama** | Protocol TVL, growth rates, chain data | `api.llama.fi/protocols` |
| **CoinGecko** | Token prices, market caps, volume, ATH | `api.coingecko.com/api/v3/coins/markets` |
| **Etherscan v2** | On-chain transfers, token supply, wallet data | `api.etherscan.io/v2/api` |

---

## Pipeline Architecture

```
DefiLlama API ─┐
                ├──→ Python ETL ──→ Transform ──→ 8 CSV files ──→ Dune Analytics
CoinGecko API ──┤      │
                │      ├── Clean null values
Etherscan API ──┘      ├── Classify protocols into 6 categories
                       ├── Calculate market share, concentration
                       ├── Track mint/burn flows
                       └── Identify whale wallets
```

### Output Files
| File | Records | Description |
|------|---------|-------------|
| `rwa_protocols.csv` | 156 | All RWA protocols with TVL, growth, chains |
| `rwa_categories.csv` | 6 | Aggregated by category type |
| `rwa_chains.csv` | 67 | TVL distribution across blockchains |
| `rwa_tokens.csv` | 50 | Token market data from CoinGecko |
| `rwa_yield_tokens.csv` | 7 | Yield-bearing tokens (BUIDL, USDY, USYC, etc.) |
| `rwa_gov_tokens.csv` | 8 | Governance tokens (ONDO, LINK, CFG, etc.) |
| `rwa_transfers.csv` | 800 | On-chain whale transfers from Etherscan |
| `rwa_supply.csv` | 8 | Token supply data |

---

## How to Run

```bash
# Clone the repo
git clone https://github.com/adi0x/rwa-analytics.git
cd rwa-analytics

# Install dependencies
pip install requests pandas

# Run the pipeline
python rwa_tracker.py
```

**Requirements:**
- Python 3.8+
- Free API keys: [Etherscan](https://etherscan.io/apis) (included in script for demo)
- No paid APIs needed — DefiLlama and CoinGecko free tiers are sufficient

---

## Dune Queries

All queries used in the dashboard:

```sql
-- Total RWA TVL
SELECT SUM(tvl) as total_rwa_tvl
FROM dune.adu0x.dataset_rwa_protocols

-- Category Breakdown
SELECT category_type, total_tvl, num_protocols
FROM dune.adu0x.dataset_rwa_categories
ORDER BY total_tvl DESC

-- Institutional Net Flow (Mint vs Burn)
SELECT 
    token,
    SUM(CASE WHEN type = 'MINT' THEN CAST(value AS DOUBLE) ELSE 0 END) -
    SUM(CASE WHEN type = 'BURN' THEN CAST(value AS DOUBLE) ELSE 0 END) as net_flow
FROM dune.adu0x.dataset_rwa_transfers
WHERE type IN ('MINT', 'BURN')
GROUP BY token
ORDER BY net_flow DESC
```

---

## Tech Stack

- **Extract:** Python `requests` library, 3 REST APIs
- **Transform:** `pandas` for data cleaning, aggregation, classification
- **Load:** CSV export → Dune Analytics upload
- **Visualize:** Dune SQL + built-in chart engine
- **Rate Limiting:** 0.3s delays for Etherscan free tier (5 calls/sec)

---

## What I Learned

- **API Version Migration:** Etherscan deprecated v1 mid-project — had to migrate to v2 API
- **Null Value Handling:** `p.get("tvl") or 0` pattern vs `p.get("tvl", 0)` — the latter fails when key exists with `None` value
- **Decimal Precision:** Different tokens use different decimals (USYC: 6, PAXG: 18) — requires per-token lookup
- **Data Quality:** No single API gives the full picture — joining 3 sources reveals insights none have alone

---

## About

Built by [@adi0x](https://twitter.com/adi0x) 
Other projects:
- [Hyperliquid Whale Tracker](https://dune.com/adu0x/hyperliquid-whale-tracker)
- [Aave Liquidation Monitor](https://github.com/adi0x)
