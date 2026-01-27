---
icon: arrow-down-to-line
---

# Installation

{% hint style="info" %}
To fully utilize the features of HOT Kit, obtain a free API key from the platform at [pay.hot-labs.org/admin/api-keys](https://pay.hot-labs.org/admin/api-keys).
{% endhint %}

### Client-side setup

`npm install @hot-labs/kit`

HOT Kit require you to install **node-polyfills** and react to work, for **vite** you need to complete the following extra steps:

`npm install vite-plugin-node-polyfills @vitejs/plugin-react`

Then in your `vite.config.ts` add this plugins:

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { nodePolyfills } from "vite-plugin-node-polyfills";

export default defineConfig({
  plugins: [nodePolyfills(), react()],
});
```

Also HotConnector use React and ReactDOM to render UI, you should install this deps to start work:

```
npm install react react-dom
```

And now your can initialize connector:

```ts
import { HotConnector } from "@hot-labs/kit";
import { defaultConnectors } from "@hot-labs/kit/defaults";

const connector = new HotConnector({
  connectors: defaultConnectors,
  apiKey: "Get on https://pay.hot-labs.org/admin/api-keys for free",

  // optional get on https://dashboard.reown.com
  walletConnect: {
    projectId: "1292473190ce7eb75c9de67e15aaad99",
    metadata: {
      name: "Example App",
      description: "Example App",
      url: window.location.origin,
      icons: ["/favicon.ico"],
    },
  },
});
```

### Nodejs setup

```
npm install @hot-labs/kit
```

To work server-side, you cannot use the HotConnector class, but you can use components from the library available via `@hot-labs/kit/core`. Additional build configuration or polyfills are not needed. Instead of using a UI to connect an external wallet, you need to directly create a wallet using your private key.

```typescript
import { tokens, Network } from '@hot-labs/kit/core';
import { NearWallet } from '@hot-labs/kit/near';

// optional, refresh tokens and rates in background process
tokens.startTokenPolling()

// Force refresh tokens list
tokens.refreshTokens()

const privateKey = Buffer.from(process.env.PRIVATE_KEY, 'hex')
const wallet = await NearWallet.fromPrivateKey(privateKey, process.env.SIGNER_ID);

const usdc = tokens.get("native", Network.Base)
const balances = await wallet.fetchBalances() as Record<string, bigint>
console.log(`${usdc.float(balances[usdc.id])} ${usdc.symbol}) // 2.3 USDC

```
