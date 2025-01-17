---
head:
  - - meta
    - property: og:title
      content: Local Accounts
  - - meta
    - name: description
      content: What is a Local Account?
  - - meta
    - property: og:description
      content: What is a Local Account?

---

# Local Accounts (Private Key, Mnemonic, etc)

A Local Account performs signing of transactions & messages with a private key **before** executing a method over JSON-RPC.

There are three types of Local Accounts in viem:

- [Private Key Account](/docs/accounts/privateKey)
- [Mnemonic Account](/docs/accounts/mnemonic)
- [Hierarchical Deterministic (HD) Account](/docs/accounts/hd)

Below are the steps to integrate a **Private Key Account**, but the same steps can be applied to **Mnemonic & HD Accounts**.

### 1: Initialize a Wallet Client

Before we set up our Account and start consuming Wallet Actions, we will need to set up our Wallet Client with the [`http` Transport](/docs/clients/transports/http):

```ts
import { createWalletClient, http } from 'viem'
import { mainnet } from 'viem/chains'

const client = createWalletClient({
  chain: mainnet,
  transport: http()
})
```

### 2: Set up your Local Account

Next, we will instantiate a Private Key Account using `privateKeyToAccount`:

```ts
import { createWalletClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts' // [!code focus]
import { mainnet } from 'viem/chains'

const client = createWalletClient({
  chain: mainnet,
  transport: http()
})

const account = privateKeyToAccount('0x...') // [!code focus:1]
```

### 3: Consume [Wallet Actions](/docs/actions/wallet/introduction)

Now you can use that Account within Wallet Actions that need a signature from the user:

```ts
import { createWalletClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

const client = createWalletClient({
  chain: mainnet,
  transport: http()
})

const account = privateKeyToAccount('0x...')

const hash = await client.sendTransaction({ // [!code focus:5]
  account,
  to: '0xa5cc3c03994DB5b0d9A5eEdD10CabaB0813678AC',
  value: parseEther('0.001')
})
```

### Optional: Hoist the Account

If you do not wish to pass an account around to every Action that requires an `account`, you can also hoist the account into the Wallet Client.

```ts
import { createWalletClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { mainnet } from 'viem/chains'

const account = privateKeyToAccount('0x...')

const client = createWalletClient({ // [!code focus:99]
  account, // [!code ++]
  chain: mainnet,
  transport: http()
})

const hash = await client.sendTransaction({
  account, // [!code --]
  to: '0xa5cc3c03994DB5b0d9A5eEdD10CabaB0813678AC',
  value: parseEther('0.001')
})
```

### Optional: Extend with Public Actions

When using a Local Account, you may be finding yourself using a [Public Client](/docs/clients/public) instantiated with the same parameters (`transport`, `chain`, etc) as your Wallet Client.

In this case, you can extend your Wallet Client with [Public Actions](/docs/actions/public/introduction) to avoid having to handle multiple Clients.

```ts {12}
import { createWalletClient, http, publicActions } from 'viem'
import { mainnet } from 'viem/chains'

const account = privateKeyToAccount('0x...')

const client = createWalletClient({
  account,
  chain: mainnet,
  transport: http()
}).extend(publicActions) // [!code ++]

const { request } = await client.simulateContract({ ... }) // Public Action
const { data } = await client.writeContract(request) // Wallet Action
```
