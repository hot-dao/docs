# Omni Tokens

## Abstract

**HOT Bridge** is a protocol for minting omni-assets within the **HOT OMNI Balance** smart contract, backed 1:1 by assets locked on native networks (e.g., Solana, Ethereum, TON). These assets can be transferred and swapped within inside **HOT OMNI Balance**, and redeemed for their original native tokens at any time.

### Architecture&#x20;

Omni bridge consists of:

* Locker contracts, one per each supported chain
  * Store native assets
  * Implement api for deposit validation by HOT Protocol validators
  * Process deposit/withdrawals of native assets
* HOT OMNI Balance, contract on NEAR Protocol
  * Store, Mint and Burn omni-assets.
  * Implement api for withdrawal validation by HOT Protocol validators
  * Implement NEP-254 Multitoken standard, so omni-assets can be used by any smart contract on NEAR Protocol. This is useful in a higher level tools like [Near Intents](https://docs.near.org/chain-abstraction/intents/overview)&#x20;
* HOT Bridge - list of instructions within client sdk for depositing and withdrawing assets on HOT Omni Balance.
* HOT Protocol - MPC networks of validators that validate cross-chain messages.

### Deposit flow



<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



1.  **Transfer liquidity to the locker contract**\
    How this done exactly depends a specific chain. It can be either token transfer to the contract address, or tokens attached to the call.&#x20;

    After deposit, locker contract save deposit data into the contract state  generate a unique `nonce`.&#x20;
2. **Generate Proof**\
   The user call HOT MPC networks, providing nonce and deposit arguments. Each MPC node call view method of locker contract to verify that this deposit actually exist.
3. **Mint omni-token**\
   The user execute `deposit`  method on Omni Balance contract, providing nonce, signature and deposit arguments. The Omni Balance contract verify that signature is valid and nonce have never been used before and mints omni-tokens.

### Withdraw flow

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* **Burn omni-asset**\
  After withdrawal, omni balance contract burns omni-token, save withdrawal data into the contract state  and generate a unique `nonce`.&#x20;
* **Generate Proof**\
  The user call HOT MPC networks, providing nonce and withdrawal arguments. Each MPC node call view method of omni balance contract to verify that this withdrawal actually exist.
* **Get token on target chain**\
  The user execute `withdraw`  method on locker contract, providing nonce, signature and withdrawal arguments. The locker contract verify that signature is valid and nonce have never been used before and transfers native tokens to receiver.

## **Omni Bridge API**

### `deposit`  method on `v2_1.omni.hot.tg` contract on NEAR Blockchain

Args:

```
receiver_data: AccountId/String
receiver_id: String
chain_id: u64
contract_id: String
amount: U128
nonce: U128
signature: Signature
```

* `signature` - MPC **ECDSA** signature of `hash(rlp_encode(receiver, chain_id, token_address, amount, nonce))`&#x20;
* `receiver`  - 32 bytes =  `hash(receiver_data)`. Some smart contracts work with a fixed par memory size, so receiver\_id is always 32 bytes. The receiver\_data format can be extended to work with the intents contract without requiring the data format on the locker contracts to be changed.
* `nonce`  - nonce, that have been generated inside Locker Contract
* `amount` - deposit amount
* `contract_id` - token contract address bytes in base58&#x20;
  * on TON:  `Cell(address).to_bytes()`
  * on Stellar: `Asset("USDC", "PUBLIC_NETWORK_PASSPHRASE").to_xdr_bytes()`
  * on EVM: `bytes.from_hex(contarct address)`
  * &#x20;on Solana: Token Mint Address
* `chain_id` - chain id
  * EVM chain id for all EVM Chain
  * TON = 1111
  * SOLANA = 1001
  * STELLAR = 1100
  * TRON = 999
