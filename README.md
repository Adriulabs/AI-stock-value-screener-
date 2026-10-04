# AI Stock Value Screener

A web page where you enter a stock ticker and get a valuation verdict
(**strong buy / buy / hold / sell / avoid**) based on several valuation models,
plus a separate quality rating of the business.

It replaces a manually filled Excel sheet: the financial figures are fetched
automatically, the models are calculated instantly, and every input can still be
edited by hand.

**Live:** https://ai-stock-value-screener.vercel.app
(manual input works for everyone; automatic data fetching requires an access code)

## How it works

1. Enter a ticker (e.g. `KO`, `MSFT`, `JPM`) and your access code, then click **Fetch data**. Automatic fetching works for US-listed tickers, for other markets, type the figures in by hand (all models still work).
2. A serverless function (`api/data.js`) fetches the figures from Financial Modeling Prep.
   The API key stays on the server and is never visible in the page.
3. The page fills in the input fields and calculates all models in the browser.
4. Change any input (growth, discount rate, fair P/E, ...) and the results update live.

## Models

### Non-financial companies

| Model | What it does |
|-------|--------------|
| Fair Value P/E | EPS × fair P/E (median P/E of the last 5 years) |
| Graham Number | √(22.5 × EPS × BVPS); skipped when P/B > 5 |
| PEG | P/E divided by EPS growth |
| FCF Yield | 3-year average free cash flow per share / price |
| Gordon Growth | Dividend-based value; skipped when dividend yield < 1.5% |
| DCF (10 years) | EPS growth for 5 years (capped at 15%), then fading linearly to terminal growth |
| ROIC | Shown separately as a **quality** rating |

### Banks and other financials

FCF, ROIC and Gordon do not make sense for banks, so a different set is used:

| Model | What it does |
|-------|--------------|
| Justified P/B | (ROE − g) / (k − g) × book value per share |
| Residual Income | Book value + present value of earnings above the required return |
| DCF (FCFE) | Earnings minus the capital needed to grow, discounted |
| Fair P/E, Graham, PEG | Same as above |
| ROE | Shown separately as a **quality** rating |

The required return is CAPM-based with an adjustable minimum (default 8%), so
low-beta stocks do not get unrealistically cheap discount rates.

## Scores

**Price score (0–100)** answers "is the price attractive?" and is a weighted
average of the applicable models. Models that do not fit a stock are excluded
and the remaining weights are renormalized.

| Non-financials | Weight | Financials | Weight |
|----------------|--------|------------|--------|
| DCF | 25 | Justified P/B | 20 |
| Fair P/E | 15 | Fair P/E | 20 |
| PEG | 15 | DCF (FCFE) | 20 |
| FCF Yield | 10 | Residual Income | 15 |
| Graham | 10 | Graham | 15 |
| Gordon | 5 | PEG | 10 |

| Score | Verdict |
|-------|---------|
| 85–100 | STRONG BUY |
| 75–84 | BUY |
| 60–74 | HOLD |
| 45–59 | SELL |
| below 45 | AVOID |

**Quality** (ROIC, or ROE for financials) is shown separately, so you can tell
"great company, expensive price" apart from "cheap but low quality".

The page also shows how many models say buy / hold / sell, three **DCF scenarios**
(pessimistic / base / optimistic) and a **sensitivity table** (discount rate × growth)
so you can see how much the result depends on the assumptions.

## Limitations

- **Data is not real-time** and comes from the Financial Modeling Prep free plan.
  Each lookup uses 7 API requests (about 250 per day on the free plan).
  The free plan is intended for personal use; a public service needs a paid plan.
- **EPS growth is historical**, not an analyst forecast. DCF results depend heavily on it
  and on the discount rate, which is why scenarios and a sensitivity table are included.
- **Banks:** ROE uses total equity, not tangible equity (ROTCE), so high-quality banks
  can look more expensive than they are. Banks are best compared with each other.
- **Not handled yet:** REITs (EPS is not the right measure), insurers as a separate case,
  and companies with negative earnings.
- Reported earnings can include one-off items that distort growth rates.

## Disclaimer

This is not investment advice. Results depend on the accuracy of the input data and
the assumptions of each model. Always do your own research.
