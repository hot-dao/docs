# Transfer omni token

Если вы работаете с омни балансами, то два самых частых юзкейса это обмен и трансфер.  Давайте рассмотрим как можно реализовать отправку омни токена с вашего кошелька на другой адрес.

{% hint style="info" %}
Полностью рабочий пример вы можете найти здесь:\
[https://github.com/hot-dao/kit/blob/main/examples-node/transfer.ts](https://github.com/hot-dao/kit/blob/main/examples-node/transfer.ts)
{% endhint %}

#### 1. Connect wallet

Для начала, если вы используете **nodejs**, а не браузер, то вам необходимо инициализировать кошелек через приватный ключ. Например так:

```typescript
import { NearWallet } from "@hot-labs/kit/near";

const privateKey = Buffer.from(process.env.PRIVATE_KEY, 'hex')
const wallet = await NearWallet.fromPrivateKey(privateKey, process.env.ACCOUNT_ID);
```

Для приложения в браузере вам достаточно запросить кошелек у HOT Connector:

```typescript
import { HotConnector } from "@hot-labs/kit"
import { defaultConnectors } from "@hot-labs/kit/defaults"
const kit = new HotConnector({ connectors: defaultConnectors })
const wallet = await kit.connect(); // Open UI
```

#### 2. Recipient

Теперь создадим адрес получателя. Так как вы отправляете омни токены с одного аккаунта на другой, то вы не можете просто использовать ончейн адрес получателя. Омни балансы хранятся на адресах другого формата, поэтому для начала создадим объект Recipient:

```typescript
import { Recipient, WalletType } from "@hot-labs/kit/core";

// Real onchain evm address:
const recipient = await Recipient.fromAddress(WalletType.EVM, "0x...");
```

Recipient это удобный класс, который вычисляет omni адресс из вашего ончейн адреса. У такого класса есть три поля: `type`, `addres` и `omniAddress`&#x20;

#### 3. Build and execute intent

Теперь мы готовы собрать наш интент на трансфер и отправить omni NEAR с нашего кошелька на EVM кошелек (омни баланс):

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

У каждого кошелька в HOT Kit есть специальный IntentsBulder, с помощью которого вы можете сформировать любое действие с вашими омни балансом. Цепочка команд чаще всего заканчивается вызовом execute, который вызывает подпись сформированных интентов и затем отправляет их в блокчейн (бесплатно!).&#x20;

В результате мы получаем хеш транзакции, результат которой можно отследить в HOT Scan. Пример транзакции: [https://hotscan.org/transaction/4cYXDkgofecfPKWvjeAnqs1VtRP1PbnLeGLZnKdoTmnT](https://hotscan.org/transaction/4cYXDkgofecfPKWvjeAnqs1VtRP1PbnLeGLZnKdoTmnT)

#### Why I need transfer to omni?

Переводы внутри омни очень быстрые и полностью бесплатные, вам не нужно думать о том какой блокчейн у получателя или сколько денег нужно потратить на комиссии. Однако если вам все же нужно отправить омни токены сразу на конкретный ончейн кошелек, то вам поможет следующая глава про вывод омни токенов.
