# 🔑 Auth by Keys

Prerequisites

Ensure you have the following libraries installed:

```bash
pip install asyncio json sha256 base58 ed25519 httpx py_near
```

#### Import Necessary Modules

```python
import asyncio
import json
from hashlib import sha256
import base58
import ed25519
import httpx
from py_near.account import Account
from app import dao_nc
from configs import CONFIG
```

#### Define Keys and Wallet ID

```python
public_key = "[KEY]"
private_key = "...."
wallet_id = "EHGDUJ7GDASucUVU4ngsgUsTYZT1FpG91CTFG8iUmGQ5"
pk = ed25519.SigningKey(base58.b58decode(private_key))
```

#### Add Access

The following steps outline how to add access to a wallet.

1.  **Get Wallet Information**

    ```python
    wallets = await dao_nc.view_function(
        "keys.auth.hot.tg",
        "get_wallet",
        {"wallet_id": wallet_id},
    )
    ```
2.  **Prepare Access Data**

    ```python
    access = {"public_keys": [public_key], "rules": []}
    access_json = json.dumps(access).replace(" ", "")
    ```
3.  **Create and Sign the Message**

    ```python
    msg = access_json + f".{wallet_id}"
    msg_hash = sha256(msg.encode()).digest()
    signature_add = pk.sign(msg_hash)
    signature_add = base58.b58encode(signature_add).decode()
    ```
4.  **Call the `grant_assess` Function**

    ```python
    wallets = await dao_nc.function_call(
        "keys.auth.hot.tg",
        "grant_assess",
        {
            "wallet_id": wallet_id,
            "access": access,
            "signatures": [signature_add],
            "auth_method": 0,
        },
        amount=1,
        included=True,
    )
    print(wallets)
    ```

#### Revoke Access

The following steps outline how to revoke access from a wallet.

1.  **Create and Sign the Message for Revoking Access**

    ```python
    access_id = 0
    msg = "revoke-auth-{}-{}".format(wallet_id, access_json)
    msg_hash = sha256(msg.encode()).digest()
    signature = pk.sign(msg_hash)
    signature = base58.b58encode(signature).decode()
    ```
2.  **Call the `revoke_assess` Function**

    ```python
    wallets = await dao_nc.function_call(
        "keys.auth.hot.tg",
        "revoke_assess",
        {
            "wallet_id": wallet_id,
            "access_id": access_id,
            "signatures": [signature],
            "auth_method": 0,
        },
        amount=1,
        included=True,
    )
    print(wallets)
    ```

#### Execute Multiple Actions

The following steps outline how to execute multiple actions in a single call.

1.  **Create and Sign the Messages**

    ```python
    access_id = 0
    msg = "revoke-auth-{}-{}".format(wallet_id, access_json)
    msg_hash = sha256(msg.encode()).digest()
    signature = pk.sign(msg_hash)
    signature = base58.b58encode(signature).decode()
    ```
2.  **Call the `execute` Function**

    ```python
    wallets = await dao_nc.function_call(
        "keys.auth.hot.tg",
        "execute",
        {
            "actions": [
                {
                    "wallet_id": wallet_id,
                    "signatures": [signature_add],
                    "auth_method": 0,
                    "action": {"Add": access},
                },
                {
                    "wallet_id": wallet_id,
                    "signatures": [signature],
                    "auth_method": 0,
                    "action": {"Revoke": access_id},
                },
            ]
        },
        amount=1,
        included=True,
    )
    print(wallets)
    ```

#### Running the Function

To run the function, use the following:

```python
asyncio.run(f())
```

This documentation provides a clear example of how to manage wallet access using digital signatures and API calls in an asynchronous Python environment.
