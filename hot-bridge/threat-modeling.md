# 👮 Threat Modeling

This document provides a detailed threat model for the HOT Bridge deposit flow. For each step of the flow, we outline potential attack vectors and describe the defense mechanisms implemented to mitigate them. The withdrawal process mirrors the deposit flow and shares the same security architecture.

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### Overview

HOT Bridge is a cross-chain liquidity protocol that enables native asset deposits on supported blockchains into the HOT Omni Balance system. The protocol is governed by **HOT DAO**, a decentralized organization responsible for contract upgrades, validator approval, and cryptographic key rotation. All DAO members undergo KYC and background verification.

Validator infrastructure and HOT MPC nodes are distributed globally across trusted cloud providers, including **Amazon Web Services**, **Google Cloud**, and **Hetzner**, ensuring resilience, redundancy, and geopolitical diversity.

***

### 🔁 Deposit Flow Threat Model

***

#### 1. Deposit to Locker Contract

**🔸 Threat: Malformed or manipulated arguments**

* **Risk:** An attacker may craft invalid or manipulated inputs to bypass logic or trigger unexpected contract behavior.
* **Mitigation:** Client-side validation is enforced through HOT SDKs. Incorrectly formatted inputs will be rejected before submission.

**🔸 Threat: Smart contract bugs (e.g. unlimited deposits, unauthorized withdrawals)**

* **Stellar Network Mitigation:**
  * The Stellar locker smart contract has undergone internal review by the **Stellar Development Foundation (SDF)**.
  * HOT DAO retains exclusive control over contract upgrades.
  * Additional third-party audits are scheduled as liquidity grows.
* **EVM Network Mitigation:**
  * The locker smart contract has been audited by **Hacken**, a leading Web3 security firm.
  * Audit reports are published and publicly verifiable.
  * Upgradeability is gated through HOT DAO governance.

***

#### 2. Generate Signature Request

**🔸 Threat: Tampered deposit arguments**

* **Risk:** A user might modify the original deposit parameters to obtain unauthorized approval.
* **Mitigation:** All deposits are immutably recorded in the locker contract. During validation, the signature module cross-checks input arguments against the on-chain record. Any mismatch results in rejection.

**🔸 Threat: Argument collision via alternative serialization**

* **Risk:** Different inputs producing the same internal representation might allow ambiguous interpretation.
* **Mitigation:** HOT Bridge uses **RLP serialization**, which is designed to avoid collisions between distinct argument sets. This ensures deterministic and secure encoding.

***

#### 3–6. Authorization Lookup for MPC Key

**🔸 Threat: Unauthorized modification of key authorization methods**

* **Mitigation:**
  * The **HOT Key Registry** smart contract maintains a static list of valid authorization methods per MPC key.
  * It does **not** support dynamic mutation by external actors.
  * The registry contract has been internally reviewed and cross-audited by the **NEAR One**, **FAST Near**, and **Intents** teams.
  * Contract upgrades and configuration changes are governed by HOT DAO votes.

**🔸 Threat: Malicious RPC or node manipulation (NEAR)**

* **Risk:** An attacker with access to the local NEAR node could return falsified data to validators.
* **Mitigation:**
  * HOT validators run independent NEAR nodes and verify data across multiple trusted RPCs.
  * Infrastructure is hosted on **AWS**, **Google Cloud**, and **Hetzner** for geographical and technical diversity.
  * Tampering would require compromising the underlying infrastructure and simultaneously defeating threshold cryptography.

***

#### 7–10. Signature Validation via Locker Contract

**🔸 Threat: Local node rollback or inconsistency**

* **Risk:** Node state may be stale or incomplete due to chain re-orgs or RPC desynchronization.
* **Mitigation:**
  * Validators maintain their own nodes and cross-verify transactions against a quorum of trusted third-party RPC endpoints (e.g. **QuickNode**, **Ankr**, **Infura**).
  * They wait for block finality before processing.
  * If validators cannot reach consensus, no signature is generated, preventing unauthorized mints.

**🔸 Threat: Compromise of HOT MPC validator network (N > threshold)**

* **Mitigation:**
  * Validators are carefully selected KYB-verified organizations.
  * Nodes are distributed across **multiple countries**, **cloud providers**, and **jurisdictional zones**.
  * HOT DAO enforces onboarding and offboarding of validators through proposals and votes.
  * The MPC system enforces threshold-based signing, making it resilient to partial compromises.
  * HOT Labs is working on migrating signature validation into **Trusted Execution Environments (TEE)** to eliminate risks from compromised hosts.

***

#### 11. Return of Signed Arguments

**🔸 Threat: Exposure of sensitive data**

* **Mitigation:** The signed message contains only public data derived from validated deposits. No private user information or sensitive protocol state is included.

***

#### 12. Omni-Token Minting

**🔸 Threat: Argument tampering or mismatch**

* **Mitigation:** The minting module validates that all arguments match the deposit record stored in the locker contract.

**🔸 Threat: Argument collision attack**

* **Mitigation:** The same RLP serialization protections from earlier steps apply here as well.

**🔸 Threat: Replay attack (re-use of a deposit nonce)**

* **Mitigation:** The Omni contract tracks used nonces and rejects any attempt to reuse a deposit for minting.

**🔸 Threat: Use of compromised non-MPC key**

* **Mitigation:**
  * Only HOT MPC signatures are accepted for minting.
  * Key rotation and signature policy updates are enacted via HOT DAO proposals.
  * All participants in the DAO are KYC’d and vetted before being granted voting rights.

**🔸 Threat: Smart contract bug enabling unlimited mints**

* **Mitigation:**
  * The Omni Balance smart contract is covered by over **92% unit test coverage**.
  * It has been internally reviewed by developers from **NEAR One**, **FAST Near**, and **Intents**.
  * Like other core contracts, it is upgradeable only via HOT DAO governance.
  * External audits are planned as part of the protocol’s scaling roadmap.

***

### 📜 Governance and Upgrade Policy

* HOT Bridge is governed by **HOT DAO**
* All smart contract upgrades, validator set changes, and key rotations must be approved by accounts, controled by Hardware wallets.
* DAO participants are subject to **KYC**, **background checks**, and **ongoing monitoring**.
* Audit reports are transparently published, and infrastructure decisions are publicly documented via governance proposals.



## 🧠 Cross-chain Threat Modeling

This section outlines the **threat model for cross-chain asset transfers** using the HOT Bridge Protocol. It focuses specifically on the **deposit and withdrawal** of native assets into and out of the **HOT Omni Balance** system.

***

### 🔒 Core Assumptions and Trust Boundaries

The security of HOT Bridge relies on **two critical points of trust**:

1. The **Locker Contract** on the origin chain (e.g. Ethereum, Solana, Base)
2. The **Omni Balance Contract** on NEAR

All minting and burning of omni-assets occurs **only** after successful validation through these two smart contracts.

Importantly, the architecture ensures **security isolation across chains**. For example:

> An exploit or misconfiguration affecting the **Base Locker Contract**, its RPC layer, or its validation process **can only affect omni-assets backed by Base**.\
> It **cannot impact** omni-assets backed by Ethereum, Solana, or other chains.

This modular isolation allows HOT DAO to **progressively increase security requirements** (e.g. more audits, stricter governance) **as liquidity on a specific chain grows** — without increasing risk to the rest of the ecosystem.

***

### 🔄 Token Swap Boundary

This document **does not cover** token-to-token swaps (e.g., `USDC(Base)` → `USDC(Stellar)`), as they happen outside the bridge on a **separate smart contract**: the **NEAR Intents**.

The Intents contract manages liquidity pools of omni-assets and enables atomic swaps, subject to available liquidity. It interacts with but is **not part of** the deposit/withdrawal bridge itself.

For an in-depth analysis of the **security model and architecture of Intents**, please refer to the article

HACKEN AUDIT FOR INTENTS CONTRACT

[https://www.datocms-assets.com/50156/1738583399-hacken\_aurora-labs-limited-sca-aurora-labs-defuse-contracts-dec2024\_p-2024-1418\_2\_20250127-10\_50.pdf](https://www.datocms-assets.com/50156/1738583399-hacken_aurora-labs-limited-sca-aurora-labs-defuse-contracts-dec2024_p-2024-1418_2_20250127-10_50.pdf)\
\
CROWDSOURCED AUDITING CHALLENGE FOR INTENTS CONTRACT\
[https://hackenproof.com/audit-programs/aurora-labs-sca-dualdefense](https://hackenproof.com/audit-programs/aurora-labs-sca-dualdefense)

{% embed url="https://github.com/near/intents/tree/d94771650382254b62cc034f293893e869be6604" %}

