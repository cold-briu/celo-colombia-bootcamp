# Multi-Sig Wallet with Vaults using Solidity, Foundry, and Celo Alfajores

A minimal, step-by-step guide entirely via terminal. We’ll:

1. Scaffold a Foundry project
2. Write a multi-sig wallet contract supporting multiple vaults (split into focused snippets)
3. Test locally with Foundry
4. Deploy to Celo Alfajores
5. Build a simple Next.js front-end (split into focused snippets)

---

## 1. Scaffold Foundry Project

```bash
# Create project folder
d mkdir multisig-vault && cd multisig-vault
# Initialize Foundry
forge init .
```

Directory structure:

```
multisig-vault/
├── lib/
├── src/
│   └── MultiSigVault.sol
├── test/
└── foundry.toml
```

---

## 2. Multi-Sig Vault Contract Breakdown

### 2.1 License & Pragma

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.19;
```

* Declares no license and sets Solidity version to 0.8.19 for safety and features.

### 2.2 Contract Definition & State Variables

```solidity
contract MultiSigVault {
    uint public vaultCount;
    uint public required;
    mapping(uint => Vault) public vaults;

    struct Vault {
        address[] owners;
        mapping(address => bool) isOwner;
        uint confirmations;
        uint balance;
    }
```

* `vaultCount`: auto-incremented ID for each vault.
* `required`: number of confirmations needed to withdraw.
* `vaults`: maps each ID to a `Vault` struct, which holds owners list, a lookup, confirmation count, and balance.

### 2.3 Constructor

```solidity
    constructor(uint _required) {
        required = _required;
    }
```

* Initializes the contract with a global confirmation threshold (e.g., 2-of-3).
* Stored in `required` for use in withdrawals.

### 2.4 Vault Creation

```solidity
    function createVault(address[] calldata _owners) external {
        vaultCount++;
        Vault storage v = vaults[vaultCount];
        for (uint i; i < _owners.length; i++) {
            v.isOwner[_owners[i]] = true;
        }
        v.owners = _owners;
    }
```

* Increments `vaultCount` to get a new ID.
* Marks each address in `_owners` as valid via `isOwner` mapping.
* Stores the owner list for iteration/tracking.

### 2.5 Deposit Function

```solidity
    function deposit(uint vaultId) external payable {
        vaults[vaultId].balance += msg.value;
    }
```

* Accepts CELO (or ETH) via `msg.value`.
* Increases the corresponding vault’s balance.

### 2.6 Confirm & Withdraw

```solidity
    function confirmAndWithdraw(uint vaultId, address payable to, uint amount) external {
        Vault storage v = vaults[vaultId];
        require(v.isOwner[msg.sender], "Not owner");
        v.confirmations++;
        if (v.confirmations >= required) {
            v.balance -= amount;
            v.confirmations = 0;
            to.transfer(amount);
        }
    }
}
```

* Checks caller is an owner.
* Increments `confirmations`.
* When confirmations ≥ `required`, resets counter, deducts balance, and transfers funds.

---

## 3. Test with Foundry

*(unchanged)*

---

## 4. Deploy to Celo Alfajores

*(unchanged)*

---

## 5. Next.js Front-End with Reown AppKit

We’ll replace manual `ethers` calls with Reown AppKit for seamless wallet connections and UX.

### 5.1 Scaffold with AppKit CLI

```bash
# In a new folder 'ui'
npx @reown/appkit-cli
```

When prompted:

1. **Project Name**: `multisig-ui`
2. **Framework**: `Next.js`
3. **Blockchain Library**: `ethers` (via Wagmi)

This bootstraps a Next.js app with Reown AppKit, Wagmi adapters, and basic connect UI.

### 5.2 Install & Configure SDK

```bash
cd multisig-ui
npm install @reown/appkit @reown/appkit-adapter-wagmi @reown/appkit-networks ethers wagmi
```

Create `reown.config.js` in project root:

```js
import { WagmiAdapter } from '@reown/appkit-adapter-wagmi';
import { alfajores } from '@reown/appkit-networks';

export const projectId = 'YOUR_REOWN_PROJECT_ID';
export const networks = [alfajores];

export const wagmiAdapter = new WagmiAdapter({
  projectId,
  networks,
  storage: undefined, // defaults
  ssr: true,
});
```

### 5.3 Wrap App in AppKit Provider

Edit `pages/_app.js`:

```jsx
import '@/styles/globals.css';
import { AppKitProvider } from '@reown/appkit/react';
import { wagmiAdapter, networks, projectId } from '../reown.config';

export default function App({ Component, pageProps }) {
  return (
    <AppKitProvider
      adapters={[wagmiAdapter]}
      networks={networks}
      projectId={projectId}
    >
      <Component {...pageProps} />
    </AppKitProvider>
  );
}
```

* Wraps the entire app to provide connection modal and hooks via context.

### 5.4 Connect & Deposit UI

Edit `pages/index.js`:

```jsx
import { useAppKit } from '@reown/appkit/react';
import { createContract } from '@reown/appkit/utils';
import { abi } from '../src/abi';

export default function Home() {
  const { connectModal, address, provider } = useAppKit();
  const MULTISIG = '0x...';

  async function deposit() {
    const contract = createContract(provider, MULTISIG, abi);
    await contract.deposit(1, { value: '0.1' });
  }

  return (
    <div className="p-4">
      {address ? (
        <button onClick={deposit} className="btn">Deposit 0.1 CELO</button>
      ) : (
        <appkit-button />
      )}
    </div>
  );
}
```

* `<appkit-button />` toggles the Reown connect modal.
* `useAppKit` provides `connectModal`, `address`, and `provider`.
* `createContract` helper wraps `ethers.Contract` and handles unit conversions.

Run the dev server:

```bash
npm run dev
```

Navigate to `http://localhost:3000` to connect and deposit via Reown-powered UI.
