# Calculate token omni address

When you deposit a token via HOT Bridge on NEAR Intents, its address will be deterministically altered from the original on-chain format. To compute the omni format, you can use the TypeScript library `@hot-labs/omni-sdk`.

<table><thead><tr><th width="148.28125">Types</th><th width="397.05859375">Format</th><th>How calculate</th></tr></thead><tbody><tr><td>Onchain</td><td>any chain-specific format</td><td></td></tr><tr><td>HOT Bridge</td><td><code>v2_1.omni.hot.tg:</code><strong><code>CHAIN</code></strong><code>_</code><strong><code>BASE58</code></strong></td><td><code>utils.toOmni</code></td></tr><tr><td>NEAR Intents</td><td><code>nep245:v2_1.omni.hot.tg:</code><strong><code>CHAIN</code></strong><code>_</code><strong><code>BASE58</code></strong></td><td><code>utils.toOmniIntent</code></td></tr></tbody></table>

## Stellar tokens

Stellar has two types of tokens:

1. **Classic Assets** (G-address issuer): Use `Asset.contractId()` to derive the Soroban contract ID
2. **Native Soroban Contracts** (C-address): Use the contract address directly

### Classic Assets (G-address issuer)

In the Stellar blockchain, there's an additional complexity. The blockchain features high-level Assets, accessible through an Issuer contract and symbol. However, HOT Bridge interacts with the token contract instead. This contract can be obtained via `asset.contractId(Networks.PUBLIC)` and this address must be converted to an HOT Bridge address.

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

### Native Soroban Contracts (C-address)

```typescript
import { utils, Network } from "@hot-labs/omni-sdk";

// Use the contract address directly
const contract = "CBIJBDNZNF4X35BJ4FFZWCDBSCKOP5NB4PLG4SNENRMLAPYG4P5FM6VN";
const omniAddress = utils.toOmniIntent(Network.Stellar, contract);
console.log(omniAddress);
```
