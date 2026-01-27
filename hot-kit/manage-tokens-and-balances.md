---
icon: badge-dollar
---

# Manage tokens and balances

### Token class

```typescript
import { tokens, chains, Network } from '@hot-labs/kit'

// В репозитории chains находится информация о всех популярных чейнах
const baseChain = chains.get(Network.Base)

// В репозитории tokens находится список главный токенов, 
// которые поддерживаются для обмена и отображения в портфолио
const ethOnBase = tokens.get("native", Network.Base)

// Переводит int в число используя decimals токена
ethOnBase.float(10_000_000n)

// Переводит число в int используя decimals 
ethOnBase.int(10)

// Актуальный курс к долллару
`USD: ${ethOnBase.float(10_000n) * ethOnBase.usd}`

// IMPORTANT:
// ID of token is combination of chain id and address
// Any native token has address === 'native'
token.id === `${token.chain}:${token.address}`
```

