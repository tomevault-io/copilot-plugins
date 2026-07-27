## binance-connector-python

> from binance_common.configuration import ConfigurationWebSocketStreams

# Agent

```python
import asyncio
import ssl
import logging

from binance_common.configuration import ConfigurationWebSocketStreams
from binance_sdk_derivatives_trading_coin_futures.derivatives_trading_coin_futures import DerivativesTradingCoinFutures

logging.basicConfig(level=logging.INFO)

configuration_ws_streams = ConfigurationWebSocketStreams(
    https_agent=ssl.create_default_context(),
)

client = DerivativesTradingCoinFutures(config_ws_streams=configuration_ws_streams)


async def all_book_tickers_stream():
    connection = None
    try:
        connection = await client.websocket_streams.create_connection()

        stream = await connection.all_book_tickers_stream()
        stream.on("message", lambda data: print(f"{data}"))
        await asyncio.sleep(5)
    except Exception as e:
        logging.error(f"all_book_tickers_stream() error: {e}")
    finally:
        if connection:
            await connection.close_connection(close_session=True)


if __name__ == "__main__":
    asyncio.run(all_book_tickers_stream())
```

---
> Source: [binance/binance-connector-python](https://github.com/binance/binance-connector-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
