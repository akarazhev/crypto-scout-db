# Issue 2: Combine `bybit_spot_tickers_btc_usdt` and `bybit_spot_tickers_eth_usdt` tables into `bybit_spot_tickers`

In this `crypto-scout-db` project we are going to combine two tables `bybit_spot_tickers_btc_usdt` and 
`bybit_spot_tickers_eth_usdt` into one `bybit_spot_tickers` table.

## Roles

Take the following roles:

- Expert database engineer.

## Conditions

- Use the best practices and design patterns.
- Do not hallucinate.

## Tasks

- As the expert database engineer review the current `init.sql` script implementation in `crypto-scout-db` project and
  update it by combining the `bybit_spot_tickers_btc_usdt` and `bybit_spot_tickers_eth_usdt` tables into `bybit_spot_tickers` table.
- Define for the `bybit_spot_tickers` table indexes, retentions and compressions.
- Remove the `bybit_spot_tickers_btc_usdt` and `bybit_spot_tickers_eth_usdt` tables with related configuration of indexes, 
  retentions and compressions.
- Recheck your proposal and make sure that they are correct and haven't missed any important points.
- Rely on the sample of the data section.
- Update the documentation `issue-2-combine-spot-tickers-tables.md` with your results in the resolution section.

## Sample of the data

```json
{
  "ts": 1673853746003,
  "type": "snapshot",
  "cs": 2588407389,
  "data": {
    "symbol": "BTCUSDT",
    "lastPrice": "21109.77",
    "highPrice24h": "21426.99",
    "lowPrice24h": "20575",
    "prevPrice24h": "20704.93",
    "volume24h": "6780.866843",
    "turnover24h": "141946527.22907118",
    "price24hPcnt": "0.0196",
    "usdIndexPrice": "21120.2400136"
  }
}
```