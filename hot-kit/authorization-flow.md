---
icon: user-tag
---

# Authorization flow

#### JWT Authentication

The `auth()` method allows you to authenticate a wallet and receive a JWT token. This is useful for backend authentication and verifying wallet ownership.

**How it works:**

1. Opens a UI popup asking the user to sign a message
2. Generates a random seed and creates a cryptographic nonce
3. Signs intents (or empty array) with the nonce
4. Sends the signed commitment to the API
5. Returns a JWT token string

**Basic Usage:**

```typescript
// Get JWT token for authentication
const wallet = wibe3.priorityWallet; // or any connected wallet
if (wallet) {
  const jwt = await wallet.auth();
  console.log("JWT token:", jwt);

  // Use JWT for backend authentication
  // Example: send to your backend API
  await fetch("https://your-api.com/authenticate", {
    method: "POST",
    headers: {
      Authorization: `Bearer ${jwt}`,
      "Content-Type": "application/json",
    },
  });
}
```

**Auth with Intents (Optional):**

You can optionally pass intents to be signed during authentication:

```typescript
// Auth with intents (optional)
const intents = [{
   intent: "transfer",
   recipient: "petya.near",
   token: OmniToken.USDC,
   amount: 10,
}];

const jwt = await wibe3.priorityWallet.auth(intents);
console.log("JWT with signed intents:", jwt);
```

**Validate JWT Token:**

You can validate a JWT token using the API:

```typescript
import { api } from "@hot-labs/kit";

// Validate JWT token
const isValid = await api.validateAuth(jwt);
console.log("Token is valid:", isValid);
```

**Complete Example:**

```tsx
import { observer } from "mobx-react-lite";
import { HotConnector } from "@hot-labs/kit";

const wibe3 = new HotConnector({ ... });

const App = observer(() => {
  const handleAuthenticate = async () => {
    const wallet = wibe3.priorityWallet;
    if (!wallet) return alert("Please connect a wallet first");

    try {
      // Get JWT token
      const jwt = await wallet.auth();

      // Store JWT (e.g., in localStorage or send to backend)
      localStorage.setItem("authToken", jwt);

      // Use JWT for authenticated API calls
      const response = await fetch("https://your-api.com/user/profile", {
        headers: { Authorization: `Bearer ${jwt}` },
      });

      const userData = await response.json();
      console.log("User data:", userData);
      alert("Authentication successful!");
    } catch (error) {
      console.error("Authentication failed:", error);
      alert("Authentication failed");
    }
  };

  return (
    <div>
      <button onClick={handleAuthenticate}>Authenticate & Get JWT</button>
    </div>
  );
});
```

**Important Notes:**

* The `auth()` method opens a UI popup that requires user interaction to sign the message
* The JWT token is generated server-side and returned after successful signature verification
* The token can be used for backend authentication to verify wallet ownership
* The authentication process is safe - it only signs a message, not a transaction
* You can optionally pass intents to be signed during authentication
