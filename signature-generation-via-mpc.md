# Signature generation via MPC

### Signature Generation

The primary use of the MPC network for us is secure signature generation.

Currently, we support the following signature formats:

* **ECDSA** — used in Bitcoin, EVM chains, BNB, Polygon
* **EdDSA** — used in Solana, NEAR, Polkadot

The MPC network is developed in collaboration with **NEAR One**.\
Some details can be found [here](https://docs.near.org/chain-abstraction/chain-signatures).

While on-chain smart contracts are used for signature generation on the NEAR blockchain, we focus primarily on the **off-chain signing API**, which offers higher bandwidth and lower latency.

### What is an HOT account?

As we work with chain agnostic identities, we have to explicitly define what identity is.

Identity is a `wallet_id: [u8; 32]` , some 32 bytes, which is a hash of _some_ `uid`&#x20;

<figure><picture><source srcset=".gitbook/assets/image (5).png" media="(prefers-color-scheme: dark)"><img src=".gitbook/assets/image (5).png" alt=""></picture><figcaption></figcaption></figure>

Suffice to say at the moment, that `uid` coming from a random source.

### Key Derivation Function&#x20;

We have to associate a key pair with an identity.

All accounts are created on top of MPC network. The MPC network has its own master key, from which all keys are derived. This process is described as Key Derivation Function:

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Thus, a set of public keys associated with `wallet_id` comes from `uid` too.

Important note: at any point of time no one knows MPC's `Secret Key` , nor `Derived Secret Key` , though `Public Key` and its derivations are well known.&#x20;

Nevertheless, it is still possible to sign a message with `Derived Secret Key` , thanks to MPC technology.&#x20;

But, if one wants to sign a message on behalf of specified `uid`,  they have to prove ownership of that `uid` – which is the same of proving ownership of `wallet_id`, which is the identity.

### Authorization of Signature Generation&#x20;

We must ensure that only messages **explicitly authorized by the user** are signed — not arbitrary requests.

Here's the abstract authorization flow, which can be used for any purpose. We will look at the specific examples (bridge use-case, mutlichain usage) later.

<figure><img src=".gitbook/assets/abstract signature authorization flow.svg" alt=""><figcaption></figcaption></figure>

1. **User sends a signature request to the MPC network**

The off-chain signing API accepts the following data:

```
JsonValue {
  uid: [u8; 32],
  message: [u8],
  proof: JsonValue
}
```

* `uid` – unique identifier for the user. Then we simply calculate `wallet_id = hash(uid)`
* `message` – message to be signed
* `proof` – data used to authorize the signature for the given uid\


2. **MPC Network before initiating a signing algorithm, starts the validation procedure**

Formally, the interpretation of `proof` is determined by each MPC node. In practice, all nodes follow a unified validation process

3-4. **Get authroization methods for the wallet**

We ask an account registry, which stores authorization methods for each `wallet_id`\
In theory, arbitrary logic can be placed in authorization method. In practice it's a contract on some blockchain which implements specific API.&#x20;

5. **Check each authorization method**

For each validation method we call a view method of the specified contract on its chain. In return we receive true/false as a result whether validation method passed.

6-7. **Proceed with the signature generation**

If all verifiaction methods succeeded, we move on to the cryptographic protocol for message signing.

8. **User get desired signature**
