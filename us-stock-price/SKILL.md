---
name: us-stock-price
description: >-
  Looks up the current United States stock price for a ticker from Yahoo
  Finance. Use when the user asks for a US stock price, share price, quote,
  or how a ticker is trading.
---

# US stock price

Get the current price of a United States stock. Replace `[SYMBOL]` in the chart URL with the ticker the user named.

## Assumption

The LLM has the ability to use the internet.

## URL

Request this URL with an HTTP GET. Put the same ticker in the path and in `symbol`:

https://query1.finance.yahoo.com/v8/finance/chart/[SYMBOL]?formatted=true&includeAdjustedClose=true&interval=1d&symbol=[SYMBOL]&lang=en-US&region=US

Write the ticker in uppercase. Keep a hyphen or dot the user included (`BRK-B`, `BRK.B`). For `AAPL`, the URL is:

https://query1.finance.yahoo.com/v8/finance/chart/AAPL?formatted=true&includeAdjustedClose=true&interval=1d&symbol=AAPL&lang=en-US&region=US

If the user names more than one ticker, request the URL once per ticker. If they name none, ask which symbol they want. Do not guess a symbol, and do not use a different host or path.

## Read the quote

Use the JSON body.

- Current price: `chart.result[0].meta.regularMarketPrice`
- Currency: `chart.result[0].meta.currency`
- Company: `chart.result[0].meta.longName`, or `shortName` when `longName` is missing
- Exchange: `chart.result[0].meta.fullExchangeName`
- Previous close: `chart.result[0].meta.chartPreviousClose`
- Change: `chart.result[0].meta.regularMarketChangePercent` (already in percent points; `6.191` means 6.191%)
- Quoted at: `chart.result[0].meta.regularMarketTime` (Unix seconds, shown in `America/New_York`)

`regularMarketPrice` is the current price. Keep the decimals Yahoo returned. Do not invent a price from `indicators` when `regularMarketPrice` is present.

If `chart.error` is not null, `chart.result` is missing or empty, or `regularMarketPrice` is missing, say the symbol was not found. Do not estimate a price.

## Reply

Lead with the current price. One short paragraph per ticker:

`{longName} ({symbol}) is {regularMarketPrice} {currency} on the {fullExchangeName}, up {regularMarketChangePercent}% from the previous close of {chartPreviousClose}, as of {time}.`

Use "down" and the absolute percent when the change is negative. Drop the change clause when previous close or change percent is missing. Use the figures from this response only. Do not add a recommendation, target, or news.
