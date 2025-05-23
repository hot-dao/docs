# Overview

### What is a HOT Protocol?

The HOT Protocol is a combination of smart contracts and off-chain services that enables the creation of logic to improve the user experience for crypto asset holders.

We focus on the following areas:

* Chain Abstraction
* Identity Management

Although the protocol is chain-agnostic, its core logic is deployed on the NEAR blockchain due to the convenience of its infrastructure

### What is Chain Abstraction?

Chain abstraction refers to the idea of hiding the complexities and specifics of individual blockchains from users or developers, enabling seamless interaction with multiple blockchains as if they were one unified system

This achieved by implementing OmniBridge to be later used in Near intents.

#### What is OmniBridge?

OmniBridge enables **interoperability** between blockchains by acting as a trusted messenger and token custodian. Instead of deploying new tokens or relying on centralized exchanges, users can move existing tokens across chains, maintaining their value and utility in different blockchain environments.

#### What is Near Intents?

> In NEAR, an `intent` is a high level declaration of what a user wants to achieve. Think of it as telling the blockchain "what" you want to do, not "how" to do it. For example, instead of manually:
>
> * Finding the best DEX for a token swap
> * Calculating optimal routes
> * Executing multiple transactions
>
> You simply express: "I want to swap Token A for Token B at the best price."

[https://docs.near.org/chain-abstraction/intents/overview](https://docs.near.org/chain-abstraction/intents/overview)

### Identity Management

Currently, there are two main options for storing your assets — and by extension, your "identity":

1. **Centralized exchanges (custodial wallets)**, such as Binance, Bybit, etc.\
   You have access to your account, which implies you _may_ have the right to manage your assets. In practice, custodial wallets are subject to regulatory restrictions. On the plus side, they’re as easy to use as logging into your Google Account — often providing a sense of chain abstraction.
2. **Self-hosted wallets** (seed-phrase based).\
   You have full control over your assets, but you bear the responsibility of securely storing your seed phrase (e.g., 12 words written on a piece of paper). This is far less intuitive compared to modern authentication flows we’re used to in everyday apps.

Think of these as the two ends of a spectrum.

We propose a solution that lies in the middle:

* No external "governance" or KYC over your assets
* Full control remains in your hands
* Flexible identity authorization logic (e.g., password + 2FA)
* Transferable identity: for instance, you could sell full access to an account for $20, after which the seller permanently loses control

This approach is implemented using **Multi-Party Computation (MPC)** services.

### What is MPC?&#x20;

**Multi-Party Computation (MPC)** is a cryptographic technique that allows multiple parties to **jointly compute a function over their inputs** without revealing those inputs to each other.

#### Core Idea

* Several parties each hold private input data.
* They want to compute a result (e.g., a signature, sum, or encrypted value) based on all their inputs.
* MPC enables this computation **without any party revealing their individual input**, and without requiring a trusted third party.

#### Simple Example

Three people want to calculate the **average of their salaries** without disclosing their individual amounts.\
MPC allows them to compute the correct average **without exposing any single salary**.

#### Our application

* **Threshold cryptography**: Splitting private keys across multiple parties (e.g., 2-of-3 signing).
* **Key management**: Secure signing without ever reconstructing the full key.
* **Secure wallets**: non-custodial key control with improved UX and no seed phrase.

\


