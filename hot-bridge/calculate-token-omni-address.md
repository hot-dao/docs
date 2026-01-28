# Calculate token omni address

## Stellar tokens

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
