---
icon: money-simple-from-bracket
---

# Withdraw omni token

Второй юзкейс при работе с омни — это вывод токенов на блокчейн, давайте рассмотрим пример как это можно реализовать через exchange:

{% hint style="info" %}
Полностью рабочий пример вы можете найти здесь:

[https://github.com/hot-dao/kit/blob/main/examples-node/withdraw.ts](https://github.com/hot-dao/kit/blob/main/examples-node/withdraw.ts)
{% endhint %}

<pre class="language-typescript"><code class="lang-typescript">import { Recipient, WalletType, tokens, OmniToken, Network, Exchange } from "@hot-labs/kit/core";
import { NearWallet } from "@hot-labs/kit/near";

// Создаем класс для обменов
const exchange = new Exchange();

// Создаем кошелек и получателя
const wallet = await NearWallet.fromPrivateKey(Buffer.from(PRIVATE_KEY), SIGNER_ID);
const recipient = await Recipient.fromAddress(WalletType.Tron, "TTB...");

// Мы будем переводить OMNI USDT на TRC20 USDT
const omniUSDT = tokens.get(OmniToken.USDT);
const realTRONUSDT = tokens.get("TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t", Network.Tron);

// Получим квоту на вывод 
const review = await exchange.reviewSwap({
    sender: wallet,
    
    // Получатель трон адрес
    recipient: recipient,
    
    // Если обмен не пройдет, то омни юсдт вернутся обратно на наш кошелек 
    refund: wallet,

    from: omniUSDT, // OMNI TOKEN
    to: realTRONUSDT, // TRON TOKEN

    // Отправляем 10 USDT
    amount: omniUSDT.int(10),
    type: "exactIn",

    // По скольку происходит не просто вывод, 
    // а обмен на другой токен, то важно заложить слипалд!
    slippage: 0.01, // 1% slippage
    
    logger: console,
});


// Смотрим сколько токенов прийдет получателю. Надо учитывать, что курс может быть плохим!
console.log("From", review.from.float(review.amountIn), review.from.symbol);
console.log("To", review.to.float(review.amountOut), review.to.symbol);

// review объект содержит все что нужно, чтобы совершить этот вывод.
// makeSwap отправит токены с вашего кошелька, но сам обмен будет занимать больше времени!
const { processing } = await exchange.makeSwap(review);

// Отдельным методом начинаем ждать результата обмена
const resultReview = await processing?.();

<strong>// Теперь деньги дошли!
</strong>console.log(resultReview);
</code></pre>
