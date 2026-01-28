---
icon: money-simple-from-bracket
---

# Withdraw omni token

The second use case when working with Omni is withdrawing tokens to the blockchain. Let’s look at an example of how this can be implemented via an exchange.

{% hint style="info" %}
You can find a complete working example here:

[https://github.com/hot-dao/kit/blob/main/examples-node/withdraw.ts](https://github.com/hot-dao/kit/blob/main/examples-node/withdraw.ts)
{% endhint %}

<pre class="language-typescript"><code class="lang-typescript">import { Recipient, tokens, OmniToken, Network, Exchange } from "@hot-labs/kit/core";
import { NearWallet } from "@hot-labs/kit/near";

const exchange = new Exchange();
const wallet = await NearWallet.fromPrivateKey(Buffer.from(PRIVATE_KEY), SIGNER_ID);
const recipient = await Recipient.fromAddress(Network.Tron, "TTB...");

// We want to exchange OMNI USDT to TRC20 USDT
const omniUSDT = tokens.get(OmniToken.USDT);
const realTRONUSDT = tokens.get("TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t", Network.Tron);

// Get qoute for exchange
const review = await exchange.reviewSwap({
    sender: wallet,
    
    // Who receive TRC20 USDT
    recipient: recipient,
    
    // If the exchange does not occur, USDT will automatically be returned to our wallet.
    refund: wallet,

    from: omniUSDT, // OMNI TOKEN
    to: realTRONUSDT, // TRON TOKEN

    // Send 10 USDT
    amount: omniUSDT.int(10),
    type: "exactIn",
    
    // Since it's not just an output but an exchange for another token, 
    // it's important to account for slippage!
    slippage: 0.01, // 1% slippage
    
    logger: console,
});


// Check how many tokens the recipient will receive. 
// Remember to consider that the exchange rate might be unfavorable!
console.log("From", review.from.float(review.amountIn), review.from.symbol);
console.log("To", review.to.float(review.amountOut), review.to.symbol);

// The review object includes everything needed to proceed with this output. 
// The makeSwap function will transfer tokens from your wallet, 
// but note that the exchange process will take more time to complete.
const { processing } = await exchange.makeSwap(review);

// Start a separate method to monitor and wait for the exchange result.
const resultReview = await processing?.();

<strong>// Funds have now arrived!
</strong>console.log(resultReview);
</code></pre>
