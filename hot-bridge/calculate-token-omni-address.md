# Calculate token omni address

<table><thead><tr><th width="148.28125">Types</th><th width="397.05859375">Format</th><th>How calculate</th></tr></thead><tbody><tr><td>Onchain</td><td>any chain-specific format</td><td></td></tr><tr><td>HOT Bridge</td><td><code>v2_1.omni.hot.tg:</code><strong><code>CHAIN</code></strong><code>_</code><strong><code>BASE58</code></strong></td><td><code>utils.toOmni</code></td></tr><tr><td>NEAR Intents</td><td><code>nep245:v2_1.omni.hot.tg:</code><strong><code>CHAIN</code></strong><code>_</code><strong><code>BASE58</code></strong></td><td><code>utils.toOmniIntent</code></td></tr></tbody></table>

## Stellar tokens

В случае с блокчейном стеллар возникает дополнительная путаница. \
В блокчейне существуют высокоуровневые Asset'ы, доступ к которым осуществляется через контракт Issuer'а и символа. Но HOT Bridge работает не с ассетами, а с контрактом токена, который можно получить через `asset.contractId(Networks.PUBLIC)` . Уже этот адрес подлежит детерминированному форматированию в омни адрес.&#x20;

```typescript
import { utils, Network, HotBridge } from "@hot-labs/omni-sdk";
import { Asset, Networks } from "@stellar/stellar-sdk";

const main = async () => {
  // symbol + issuer
  const asset = new Asset("CETES", "GCRYUGD5NVARGXT56XEZI5CIFCQETYHAPQQTHO2O3IQZTHDH4LATMYWC");

  // Convert contractId to omni address (NOT ISSUER!)
  const omniAddress = utils.toOmniIntent(Network.Stellar, asset.contractId(Networks.PUBLIC));
  console.log(omniAddress);

  // How to convert omni address to asset?
  const hotBridge = new HotBridge({});
  const [stellarChainId, tokenContractAddress] = utils.fromOmni(omniAddress).split(":"); // 1100:ADDRESS

  // Get asset from contract id
  const assetFromOmniId = await hotBridge.stellar.getAssetFromContractId(tokenContractAddress);

  // Check if the asset is the same
  console.log(assetFromOmniId.contractId(Networks.PUBLIC) === asset.contractId(Networks.PUBLIC));
};

main();
```
