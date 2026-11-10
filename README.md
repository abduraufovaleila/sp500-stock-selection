# S&P 500 Stock Selection: Tail Risk and Pair Diversification

Two quantitative-finance analyses of the S&P 500 using daily log returns from 2024 to 2025. The first ranks individual stocks by tail risk; the second searches every pair of stocks for the best diversifying portfolio.

## Notebooks

| Notebook | Question | Method |
|---|---|---|
| `01_skewness_kurtosis_ranking.ipynb` | Which stocks have the most extreme left-tail behaviour? | Keep stocks with negative skewness and at least 90% data coverage, then rank by `0.5 * rank(kurtosis) + 0.5 * rank(-skewness)`. |
| `02_pair_diversification.ipynb` | Which two stocks diversify each other best? | Brute-force search over about 122,000 pairs using a rolling 63-day diversification gain, scored as mean minus half the standard deviation. |

## Data

- **Constituents:** scraped live from the [Wikipedia list of S&P 500 companies](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies).
- **Prices:** adjusted daily closes from Yahoo Finance via `yfinance`, for 2024-01-02 to 2025-12-31.
- **Filtering:** stocks with less than 90% coverage over the period are dropped (about 495 stocks remain).
- **Benchmark:** S&P 500 index (`^GSPC`), annualised mean log return of 18.14% over the same window.

## Analysis 1: skewness and kurtosis ranking

Skewness measures how asymmetric returns are, and kurtosis measures how heavy the tails are. Stocks with strongly negative skewness and very high kurtosis have returns dominated by a few large single-day drops.

The top of the ranking (score scaled to 0-1):

| Rank | Ticker | Skewness | Kurtosis |
|---|---|---|---|
| 1 | GL | -14.20 | 281.0 |
| 2 | FISV | -11.87 | 196.2 |
| 3 | EW | -8.18 | 139.1 |
| 4 | CNC | -7.93 | 124.8 |
| 5 | DXCM | -6.84 | 101.6 |

## Analysis 2: diversification gain

For each pair of stocks `A` and `B`, an equal-weight portfolio is built and, over rolling 63-day windows (about one trading quarter), the diversification gain is

```
DG = 1 - portfolio volatility / (0.5 * vol_A + 0.5 * vol_B)
```

A higher DG means the portfolio is less volatile than the average of its parts. Each pair is scored as `mean(DG) - 0.5 * std(DG)`, which rewards a gain that is both large and stable.

Best pair, which wins both overall and under the constraint that annualised return is at least that of the S&P 500:

| Pair | Mean DG | Std of DG | Min DG | Score | Annualised return |
|---|---|---|---|---|---|
| AMZN + CBOE | 40.1% | 5.8% | 24.8% | 37.2% | 19.49% |

37,269 of the roughly 122,000 pairs meet the return constraint.

## Limitations

- **Results depend on the run date.** The constituent list is scraped live, and Yahoo Finance data can be revised, so rankings may shift slightly if you rerun the notebooks.
- **In-sample only.** Both analyses describe 2024-2025. They do not show that a pair or ranking will hold in the future.
- **Survivorship bias.** Only current index members are used, so stocks that left the index during the period are missing. Three tickers (HONA, VYLR, FDXF) had no price data and were dropped.
- **Multiple comparisons.** Searching over 122,000 pairs makes it likely that the top pair benefits partly from chance.

## Run it yourself

```bash
git clone https://github.com/abduraufovaleila/sp500-stock-selection.git
cd sp500-stock-selection
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook
```

The pair search in notebook 02 takes roughly 7 minutes.

## Tech stack

Python, pandas, NumPy, yfinance, requests, tqdm.

## Author

Leila Abduraufova
