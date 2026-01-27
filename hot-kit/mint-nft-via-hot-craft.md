---
hidden: true
icon: hexagon-vertical-nft
---

# Mint NFT via HOT Craft

#### NFT Mint Example (Omni Chain / Intents)

Mint NFTs using `authCall` intent. This example shows how to mint multiple NFTs in a batch:

```typescript
interface MintMsg {
  msg: string; // trading address
  token_owner_id: string;
  token_id: string;
  token_metadata: {
    reference?: string;
    description: string;
    title: string;
    media: string;
  };
}

interface NFT {
  title: string;
  description: string;
  image: string;
  reference?: string;
}

const kit = new HotConnector({ ... })

async function mintNFTs(collection: string, nfts: NFT[], totalSupply: number) {
  const wallet = kit.priorityWallet
  if (!wallet) throw new Error("No wallet connected");
  
  const tradingAddress = wallet.omniAddress;
  const builder = kit.intentsBuilder(wallet)

  // Calculate total storage deposit needed for all NFTs
  let totalDeposit = 0n;
  const intents: any[] = [];

  for (let i = 0; i < nfts.length; i++) {
    const nft = nfts[i];
    const msg: MintMsg = {
      msg: tradingAddress,
      token_owner_id: "intents.near",
      token_id: (totalSupply + i).toString(),
      token_metadata: {
        reference: nft.reference || undefined,
        description: nft.description || "",
        title: nft.title,
        media: nft.image,
      },
    };

    // Calculate deposit size based on metadata size
    // Formula: (JSON string length * 8 bits) / 100,000 * 10^24 yoctoNEAR
    const metadataSize = JSON.stringify(msg.token_metadata).length;
    const size = BigInt((metadataSize * 8) / 100_000) * BigInt(10 ** 24);
    totalDeposit += size;

    // Create auth_call intent
    builder.authCall({
      attachNear: size.toString(),
      contractId: collection,
      msg: JSON.stringify(msg),
      tgas: 50,
    });
  }

  // Execute all intents
  return await builder.execute();
}

// Usage example
const nfts: NFT[] = [
  {
    title: "My NFT #1",
    description: "First NFT in collection",
    image: "https://example.com/nft1.png",
    reference: "https://example.com/nft1.json",
  },
  {
    title: "My NFT #2",
    description: "Second NFT in collection",
    image: "https://example.com/nft2.png",
  },
];

// Deploy contract with 100 max supply and mint 2 nft
await mintNFTs("my-nft-collection.near", nfts, 100);
```

#### NFT UI Recommendation: Trade On HOT Craft

When working with NFTs in your UI, it's **recommended to add a "Trade On HOT Craft" button** that links to the HOT Craft marketplace:

```typescript
import { observer } from "mobx-react-lite";

const NFTComponent = observer(() => {
  return (
    <div>
      {/* Your NFT display */}
      <div>
        <img src={nft.image} alt={nft.title} />
        <h3>{nft.title}</h3>
        <p>{nft.description}</p>
      </div>

      {/* Recommended: Trade On HOT Craft button */}
      <a href="https://hotcraft.art/" target="_blank" rel="noopener noreferrer">
        Trade On HOT Craft
      </a>
    </div>
  );
});
```

**Why add this button?**

* Provides users with a marketplace to trade their NFTs
* Improves user experience by offering trading functionality
* Connects your app with the HOT Craft ecosystem

<br>
