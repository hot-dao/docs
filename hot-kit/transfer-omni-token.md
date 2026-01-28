---
icon: money-bill-transfer
---

# Transfer omni token

When working with Omni balances, the two most common use cases are exchange and transfer. Let’s take a look at how to implement sending an Omni token from your wallet to another address.

{% hint style="info" %}
You can find a fully working example here:\
[https://github.com/hot-dao/kit/blob/main/examples-node/transfer.ts](https://github.com/hot-dao/kit/blob/main/examples-node/transfer.ts)
{% endhint %}

#### 1. Connect wallet

First, if you are using Node.js rather than a browser, you need to initialize the wallet using a private key. For example:

```typescript
import { NearWallet } from "@hot-labs/kit/near";

const privateKey = Buffer.from(process.env.PRIVATE_KEY, 'hex')
const wallet = await NearWallet.fromPrivateKey(privateKey, process.env.ACCOUNT_ID);
```

For a browser-based application, it is sufficient to request a wallet from the HOT Connector:

```typescript
import { HotConnector } from "@hot-labs/kit"
import { defaultConnectors } from "@hot-labs/kit/defaults"
const kit = new HotConnector({ connectors: defaultConnectors })
const wallet = await kit.connect(); // Open UI
```

#### 2. Recipient

Now let’s create the recipient address. Since you are sending Omni tokens from one account to another, **you cannot simply use the recipient’s on-chain address**. Omni balances are stored on addresses of a different format, so first, we need to create a Recipient object:

```typescript
import { Recipient, Network } from "@hot-labs/kit/core";

// Real onchain evm address:
const recipient = await Recipient.fromAddress(Network.Eth, "0x...");
```

Recipient is a convenient class that computes the Omni address from your on-chain address. This class has three fields: `type`, `address`, and `omniAddress`.

#### 3. Build and execute intent

Now we are ready to create our **transfer intent** and send **OMNI NEAR** from our wallet to an EVM wallet (in Omni balance):

```typescript
import { OmniToken } from "@hot-labs/kit/core";

const hash = await wallet
  .intents() // create Intents Builder
  .transfer({
    recipient: recipient.omniAddress,
    token: OmniToken.NEAR, // ID like -4:omniTokenAddress
    amount: 10,
  })
  .execute(); // execute transfer intent

console.log("10 NEAR Transfer Hash:", `https://hotscan.org/transaction/${hash}`);
```

Each wallet in **HOT Kit** has a special **IntentsBuilder**, which allows you to create any action with your OMNI balance. The commands chain usually ends with a call to **execute**, which signs the created intents and then sends them to the blockchain (**for free!**).

As a result, you receive a transaction hash, which can be tracked on HOT Scan. Example transaction: [https://hotscan.org/transaction/4cYXDkgofecfPKWvjeAnqs1VtRP1PbnLe](https://hotscan.org/transaction/4cYXDkgofecfPKWvjeAnqs1VtRP1PbnLeGLZnKdoTmnT)

### Why I need transfer to omni?

Transfers within OMNI are **very fast and completely free**. You don’t need to worry about which blockchain the recipient uses or how much to spend on fees. However, if you need to send OMNI tokens directly to a specific on-chain wallet, the next chapter on withdrawing OMNI tokens will guide you.
