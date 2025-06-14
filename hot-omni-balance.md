# HOT Omni Balance

## Overview

**HOT Omni Balance** is a cross-chain representations of assets from various networks. It allow unified interaction with tokens across Ethereum-compatible chains, Solana, TON, Bitcoin, Zcash, and more.

HOT Omni Balance standardizes work with tokens secured by:

* **HOT Omni Bridge** – supporting EVM-compatible chains.
* **Lite Client Bridges** – for trustless Bitcoin and Zcash bridging.

Once bridged, all balances are stored in a **on-chain "database" smart contract**, hosted on the NEAR blockchain. This contract acts as the canonical source of truth for all Omni Token balances, enabling secure and composable interactions.

### Trustless Intents

**Intents** are signed user instructions that define specific actions to be executed on the HOT Omni Balance. Intents are signed off-chain by user and executed on-chain by any account, allowing trustless and gasless execution on behalf of the user.

### Intent Types

HOT Protocol currently supports the following core intent types for Omni Tokens:

| Intent Type   | Purpose        | Description                                                        |
| ------------- | -------------- | ------------------------------------------------------------------ |
| `transfer`    | Token transfer | Sends tokens from one user to another.                             |
| `token_diff`  | Swap           | Atomically swaps X of Token A for Y of Token B at a fixed rate.    |
| `mt_withdraw` | Withdrawal     | Withdraws tokens to their native chain via the appropriate bridge. |

All intent executions update the on-chain balance state and may trigger additional events or bridge logic.

***

### JSON Intent Examples

#### 1. Transfer

```json
{
  "intent": "transfer",
  "from": "alice.near",
  "to": "bob.near",
  "token": "nep141:usdc.omni.hot.tg:eth",
  "amount": "1000000",
  "nonce": "1749735276000003203"
}
```

#### 2. Token Swap (token\_diff)

```json
{
  "intent": "token_diff",
  "diff": {
    "nep141:usdc.omni.hot.tg:eth": "-1000000",
    "nep141:dai.omni.hot.tg:eth": "995000"
  },
  "referral": "intents.tg",
  "nonce": "1749735276000003204"
}
```

#### 3. Withdraw (mt\_withdraw)

```json
{
  "intent": "mt_withdraw",
  "token": "btc.omni.hot.tg:mainnet",
  "amount": "100000",
  "recipient": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
  "nonce": "1749735276000003205"
}
```

Each intent must be accompanied by a valid cryptographic signature using the signer’s chain-specific format.

***

### Supported Signature Standards

To enable seamless integration across chains, HOT Omni Balance supports native signature formats per network:

| Network        | Signature Format                   | Standard                |
| -------------- | ---------------------------------- | ----------------------- |
| Ethereum / EVM | `eth_sign`, `eth_signTypedData_v4` | EIP-191, EIP-712        |
| Solana         | Base58 Ed25519                     | Solana `Message`        |
| NEAR           | Ed25519 (NEP-413)                  | `signed_message`        |
| TON            | TL-B with `wallet v4/v5`           | Custom signature schema |
| Stellar        | XDR-auth message                   | SEP-0010                |

This allows users to sign intents using their native wallets without needing additional infrastructure.

***

### Swap Architecture

Swaps (e.g., `token_diff`) in HOT Protocol are designed to be flexible and modular. Execution can be handled in multiple ways:

1. **Solvers**\
   External entities (like market makers or bots) who scan the mempool or listen to intent APIs, validate signature + price, and fulfill the swap. They may:
   * Hedge externally (on CEX/DEX).
   * Route liquidity internally.
2. **On-chain AMM** _(Optional)_\
   A fallback mechanism for direct swaps without a solver. This provides a trustless execution path for certain pools.
3. **Intents-based Orderbooks** _(Experimental)_\
   Users can post signed limit orders (intents) to be matched by anyone — a novel decentralized orderbook using HOT Omni Tokens.

### Chain Abstraction dApps

Thanks to HOT Omni Balance intent model, developers can build **Chain Abstraction** apps that:

* Interact with assets from multiple chains via a single interface.
* Use one signer identity to control assets from Solana, Ethereum, TON, etc.
* Stay gasless: all fees can be subsidized or paid via token abstraction.



