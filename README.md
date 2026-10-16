# QFin trading bot

A live trading bot for the UWA QFin trading competition (October 2026), plus the research behind it. The bot traded a
simulated exchange with five fruits, an ETF on the fruit basket, and 5-minute and 60-minute UP/DOWN binary options on the
ETF. The score was mean daily P&L minus its standard deviation.

Over eight days the bot went through 25+ versions (v1 → v16). On the live board it averaged about $26k a day and sat
2nd on the score. Final rankings came from a separate end-of-competition simulation.

## What the bot does

- **Fruit and ETF market making.** Quotes inside the house market maker, centred on a joint Kalman-filter fair value for
  the five fruits (ETF fair = sum of fruit fairs), leaning against inventory. Small but steady edge (~0.3–0.6c a unit),
  and it earned in every regime.
- **Option pricing.** P(UP) = P(ETF mid at expiry + settle noise ≥ strike), with a time-of-day volatility profile, drift
  toward the basket, and the empirical settle-noise distribution (settles use the last ETF trade, not the mid).
- **Options trading.** Model takes, resting bids, an open-of-window UP buy, and a settle-second push that keeps our side
  of the strike (allowed by the organisers).
- **Regime switch (v15/v16).** The bot tracks how often UP has won recently, from the exchange's settled windows:
  buy UP when UP is winning, go flat in between, and short the 5m UP when DOWN is winning (caps chosen by replay).

## Layout

```
bot/                  final submission (v16)
  strategy.py         trading logic: Bot.on_second(snapshot) -> actions
  exchange_adapter.py live connection: builds a snapshot each second, sends orders
  models.py           Kalman fair values, option P(UP) model
  config.py           every parameter and switch, each commented with where it came from
  pair_trade.py       apples/oranges pair trade
  pnl_tracker.py      P&L by strategy
  backtest.py         historical replay
  tests/              132 unit tests
  tools/              replay and analysis scripts used to choose parameters
  CHANGELOG.md        what changed in every version and why (v1 → v16)
  DESIGN_NOTES.md     the original architecture notes
research/
  notebooks_scripts/  01–05 data studies, fair_value.py, 06_scripts/
  findings/           write-ups for each research step (02–15)
  charts/             charts referenced in the findings
```

## Research path

| Step | Topic |
|---|---|
| 01–02 | Data checks; fruits mean-revert, the ETF follows the basket, informed size-30 ETF bot |
| 03 | 5m options are mispriced: settles use the last trade, not the mid |
| 04 | Kalman fair values; the ETF's fair value is the sum of fruit fairs |
| 05 | Replay simulator scored like the competition |
| 06 | Strawberry size-150 buyer, activity U-shape, option skew and kurtosis |
| 07–09 | The close second, the hosted runtime, late-push fixes, where the option money is |
| 10–11 | Live reviews: shorting doesn't pay, resting-bid tuning (v12) |
| 14 | Fruit edge is set by adverse selection, not quote width |
| 15 | Oct 15 regime flip, v15/v16 regime switch and the DOWN-mode cap sweep |

## Running it

Requires Python 3.10+.

```
pip install -r requirements.txt
```

- **Tests:** the live adapter imports the competition's SDK (`exchange.py` from the QFin team kit), which is not included
  here. With the kit on your path:
  ```
  cd bot
  PYTHONPATH=/path/to/qfin-team-kit:. python -m unittest discover -s tests
  ```
- **Research scripts and replays** expect the competition data, which is not included. The `research/` scripts read
  `qfin_project_data/members/`; the replay tools in `bot/tools/` read the recorded live books from `$QFIN_LIVE_DATA`
  (default `data/live/`).

## Not included

The competition dataset and recorded live market data, the organisers' team kit and SDK, exchange credentials, and
leaderboard/other-team data.
