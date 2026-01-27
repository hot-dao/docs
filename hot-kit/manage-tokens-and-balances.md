---
icon: badge-dollar
---

# Manage tokens and balances

HOT Kit significantly simplifies token management, providing access to token information across various networks, current exchange rates in USD, and balances of connected wallets. Let's take a closer look at the tokens manager:

### Tokens repository

Each token has a unique ID, which is structured as `chainID:tokenAddress`. You can retrieve a specific token's class using `tokens.get`. This class offers several useful methods for formatting the balance according to the token's decimals.

```typescript
import { tokens, chains, Network } from '@hot-labs/kit'

// The repository chains contains data on all popular blockchains.
const baseChain = chains.get(Network.Base)

// The tokens repository contains a list of primary tokens, 
// which are supported for exchange and display in portfolios.
const ethOnBase = tokens.get("native", Network.Base)

// Converts an int to a float using the token's decimals.
ethOnBase.float(10_000_000n)

// Converts a number to an integer using decimals.
ethOnBase.int(10)

// Actual 0.01 ETH in dollars
console.log(`USD: ${ethOnBase.float(10n ** BigInt(ethOnBase.decimals - 2)) * ethOnBase.usd}`)

// IMPORTANT:
// ID of token is combination of chain id and address
// Any native token has address === 'native'
token.id === `${token.chain}:${token.address}`
```

### Refresh tokens and rates&#x20;

If you are working with the `HotConnector` class, the current list of tokens and balances are automatically updated. If you received a token through `tokens.get`, its USD rate will be updated automatically, meaning any access to `token.usd` will give you the current rate within a 5-minute interval.&#x20;

If you are working with Node.js or do not wish to use `HotConnector`, you can explicitly initialize automatic balance updates or request a forced update of the entire repository.

```typescript
tokens.startTokenPolling(interval?: number) // default inverval 2 minutes
tokens.refreshTokens() // or force refresh
```



