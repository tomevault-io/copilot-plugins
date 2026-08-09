## binance-connector-js

> import { HttpsProxyAgent } from 'https-proxy-agent';

# WebSocket Agent Configuration

```typescript
import { HttpsProxyAgent } from 'https-proxy-agent';
import { Alpha, ALPHA_WS_STREAMS_PROD_URL } from '@binance/alpha';

const configurationWebsocketStreams = {
    wsURL: ALPHA_WS_STREAMS_PROD_URL,
    agent: new HttpsProxyAgent('your-proxy-url'),
};
const client = new Alpha({ configurationWebsocketStreams });

client.websocketStreams.connect().then(console.log).catch(console.error);
```

---
> Source: [binance/binance-connector-js](https://github.com/binance/binance-connector-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
