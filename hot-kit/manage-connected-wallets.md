# Manage connected wallets

The `HotConnector` class implements communication between different components of the library, maintaining the state of connected wallets, balances, and active transactions. Let's look at how to use it:

```typescript
const kit = new HotConnector({ ... }) 

// Use wallet.type to specify chain: WalletType.EVM/NEAR/TON/Stellar/Solana/Cosmos
kit.onConnect(({ wallet }) => {})

// if user disconnect wallet
kit.onDisconnect(({ wallet }) => {})

const wallet = await kit.connect() // or pass WalletType.EVM
await kit.disconnect(wallet) // or just WalletType.EVM/wallet.type
```

{% hint style="info" %}
HotConnector uses MobX to provide reactive state out of the box. In a separate article, you can learn how to integrate HOT Kit with React.
{% endhint %}

### Wallet class

Regardless of the connected chain or wallet, you receive a consistent interface, allowing you to:

```typescript
interface OmniWallet {  
  readonly address: string; // chain-specific address
  readonly publicKey?: string; // chain-specific publicKey (hex/base58)
  readonly omniAddress: string; // omni address (determistic from address)
  readonly type: WalletType; // Type of wallet (EVM/TON/Solana/NEAR/Stellar/Cosmos)
  
  // tx is chain-specific object, this method will execute transaction and return hash
  async sendTransaction(tx: any): Promise<string> 

  // How many network fee you should pay to make transfer tx
  async transferFee(token: Token, receiver: string, amount: bigint): Promise<ReviewFee>

  // Transfer transaction (common arguments for any chain)
  async transfer(args: { 
     token: Token; 
     receiver: string; 
     amount: bigint; 
     comment?: string; 
     gasFee?: ReviewFee 
  }): Promise<string>

  // Balances
  async fetchBalance(chain: number, address: string): Promise<bigint>
  async fetchBalances(chain: number): Promise<Record<string, bigint>>

  // Это методы нужны для работы с омни балансами, подробнее в Intents builder
  async signIntents(intents: Record<string, any>[], options?: { nonce?: Uint8Array; deadline?: number; signerId?: string }): Promise<Commitment>
  async auth<T = string>(intents?: Record<string, any>[], options?: { domain?: string; signerId?: string; customAuth?: (commitment: Commitment, seed: string) => Promise<T> }): Promise<T>
  async waitUntilOmniBalance(need: Record<string, bigint>, receiver = this.omniAddress, attempts = 0)
}
```
