---
description: >-
  For now the main wallet, who support HOT MPC Wallet is HERE Wallet. This is
  the easiest way to start.
---

# 🔧 Wallet Connect

## @herewallet/core

In contrast to the synchronous signing of transactions in web near wallets, where the user is redirected to the wallet site for signing -- **HERE Wallet** provides the ability to sign transactions using async/await API calls.

```bash
npm i near-api-js@^0.44.2 --save
npm i @here-wallet/core --save
```

### Usage

```ts
import { HereWallet } from "@here-wallet/core";
const here = await HereWallet.connect();
const account = await here.signIn({ contractId: "social.near" });
console.log(`Hello ${account}!`);
```

**You can also login to the wallet without adding a key. For this you can call `signIn` without `contractId`**

### How it works

By default, all near-selector api calls that you make with this library run a background process and generate a unique link that the user can go to their mobile wallet and confirm the transaction. This is a link of the form: https://h4n.app/TRX\_PART\_OF\_SHA1\_IN\_BASE64

If a user has logged into your application from a phone and has a wallet installed, we immediately transfer him to the application for signing. In all other cases, we open a new window on the web.herewallet.app site, where the user can find information about installing the wallet and sign the transaction there.

All this time while user signing the transaction, a background process in your application will monitor the status of the transaction requested for signing.

### Sign in is optional!

You can generate a signing transaction without knowing your user's accountId (without calling signIn). There are cases when you do not need to receive a public key from the user to call your contract, but you want to ask the user to perform an action in your application once:

```ts
import { HereWallet } from "@here-wallet/core";
const here = await HereWallet.connect();
const tx = await here.signAndSendTransaction({
  actions: [{ type: "FunctionCall", params: { deposit: 1000 } }],
  receiverId: "donate.near",
});

console.log("Thanks for the donation!");
```

### Build Telegram App and connect HOT Telegram Wallet

```ts
import { HereWallet } from "@here-wallet/core";
const here = await HereWallet.connect({
  botId: "HOTExampleConnectBot/app", // Your bot MiniApp
  walletId: "herewalletbot/app", // HOT Wallet
});
```

### Login without AddKey

In order to use the wallet for authorization on the backend, you need to use the signMessage method. This method signs your message with a private full access key inside the wallet. You can also use this just to securely get your user's accountId without any extra transactions.

```ts
import { HereWallet } from "@here-wallet/core";
const here = await HereWallet.connect();

const nonce = Array.from(crypto.getRandomValues(new Uint8Array(32)));
const recipient = window.location.host;
const message = "Authenticate message";

const { signature, publicKey, accountId } = await here.signMessage({ recipient, nonce, message });

// Verify on you backend side, check NEP0413
const accessToken = await axios.post(yourAPI, { signature, accountId, publicKey, nonce, message, recipient });
console.log("Auth completed!");
```

Or you can verify signMessage on client side, just call:

```ts
try {
  const { accountId } = await here.authenticate();
  console.log(`Hello ${accountId}!`);
} catch (e) {
  console.log(e);
}
```

If you use js-sdk on your backend, then you do not need to additionally check the signature and key, the library does this, and if the signature is invalid or the key is not a full access key, then the method returns an error. Otherwise, on the backend, you need to verify the signature and message with this public key. And also check that this public key is the full access key for this accountId.

**It's important to understand** that the returned message is not the same as the message you submitted for signature. This message conforms to the standard: https://github.com/near/NEPs/pull/413

### Security

To transfer data between the application and the phone, we use our own proxy service. On the client side, a transaction confirmation request is generated with a unique request\_id, our wallet receives this request\_id and requests this transaction from the proxy.

**To make sure that the transaction was not forged by the proxy service, the link that opens inside the application contains a hash-sum of the transaction. If the hashes do not match, the wallet will automatically reject the signing request**
