---
description: >-
  An overview of the approach to create cross-chain token bridges on top of HOT
  Protocol.
---

# 🔓 HOT Bridge



{% hint style="warning" %}
This is an early version of the documentation and it continues to be expanded and improved.
{% endhint %}

Overview

Bridges are a bundle of contracts on 2 networks that allows users to transfer tokens between blockchains. Typically, cross-chain messages are used to transfer assets. For example LayerZero does this, they send a transfer message between contracts on different networks.

### HOT approach

HOT offers a different approach. The bridge contract on each network is controlled by a programmable MPC wallet created on top of the HOT Protocol. These wallets have fixed rules by which a transaction can be signed. HOT Protocol validators call the view `hot_verify(raw_data)`  method where they pass the full transaction that the user wants to sign. The contract reconstructs the arguments, checks their validity and gives permission to sign the transaction.

The user sends the signed transaction by RPC to the networks where he wants to receive tokens and performs the transfer. This approach works instantly, is very gas-efficient and does not require the use of third-party oracles.

### Step-by-step bridge from BASE to NEAR

1. Sent token to Bridge smart contract on Base blockchain

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

2. Create and Sign "transfer" transaction from HOT MPC blockchain

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

3. Execute sign transaction on NEAR Protocol and receive tokens

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
