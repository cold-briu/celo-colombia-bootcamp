# Simple Time‑Locked Vault (Solidity + Foundry + Celo Alfajores)

## Prerequisites

* Node.js & npm
* Foundry (`forge` CLI)
* Celo CLI (`celocli`)
* A funded Alfajores account (use the faucet)

---

## 1. Initialize Foundry Project

**Explanation:** Create a fresh Foundry workspace for our vault.

```bash
forge init time-locked-vault
cd time-locked-vault
```

---

## 2. Build the Solidity Contract

### 2.1 Contract Overview

**Explanation:** We define a `Vault` struct and a `mapping` to store multiple vaults by ID. Functions let users create, view, and withdraw their vaults.

### 2.2 Contract Code Snippets (Step by Step)

#### 2.2.1 Struct & Storage

**Explanation:** Define the `Vault` struct and a mapping to hold multiple vaults by ID.

```solidity
pragma solidity ^0.8.0;

contract TimeLockVault {
    struct Vault {
        address owner;
        uint256 amount;
        uint256 unlockTime;
        bool withdrawn;
    }

    mapping(uint256 => Vault) public vaults;
```

#### 2.2.2 Constructor

**Explanation:** A simple, no-op constructor for initialization.

```solidity
    constructor() {}
```

#### 2.2.3 createVault

**Explanation:** Creates a new vault under a unique `id`, requires future unlock time, and stores funds.

```solidity
    function createVault(uint256 id, uint256 _unlockTime) external payable {
        require(vaults[id].owner == address(0), "Vault exists");
        require(_unlockTime > block.timestamp, "Unlock time must be in future");
        vaults[id] = Vault({
            owner: msg.sender,
            amount: msg.value,
            unlockTime: _unlockTime,
            withdrawn: false
        });
    }
```

#### 2.2.4 withdrawVault

**Explanation:** Allows the owner to withdraw once the unlock time has passed, marking it withdrawn.

```solidity
    function withdrawVault(uint256 id) external {
        Vault storage v = vaults[id];
        require(v.owner == msg.sender, "Not owner");
        require(!v.withdrawn, "Already withdrawn");
        require(block.timestamp >= v.unlockTime, "Locked");
        v.withdrawn = true;
        payable(v.owner).transfer(v.amount);
    }
```

#### 2.2.5 getVault

**Explanation:** Returns details for a given vault ID: owner, amount, unlock time, and withdrawal status.

```solidity
    function getVault(uint256 id) external view returns (
        address owner,
        uint256 amount,
        uint256 unlockTime,
        bool withdrawn
    ) {
        Vault storage v = vaults[id];
        return (v.owner, v.amount, v.unlockTime, v.withdrawn);
    }
}
```

### 2.3 Full Contract File

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TimeLockVault {
    struct Vault {
        address owner;
        uint256 amount;
        uint256 unlockTime;
        bool withdrawn;
    }

    mapping(uint256 => Vault) public vaults;

    constructor() {}

    function createVault(uint256 id, uint256 _unlockTime) external payable {
        require(vaults[id].owner == address(0), "Vault exists");
        require(_unlockTime > block.timestamp, "Unlock time must be in future");
        vaults[id] = Vault({
            owner: msg.sender,
            amount: msg.value,
            unlockTime: _unlockTime,
            withdrawn: false
        });
    }

    function withdrawVault(uint256 id) external {
        Vault storage v = vaults[id];
        require(v.owner == msg.sender, "Not owner");
        require(!v.withdrawn, "Already withdrawn");
        require(block.timestamp >= v.unlockTime, "Locked");
        v.withdrawn = true;
        payable(v.owner).transfer(v.amount);
    }

    function getVault(uint256 id) external view returns (
        address owner,
        uint256 amount,
        uint256 unlockTime,
        bool withdrawn
    ) {
        Vault storage v = vaults[id];
        return (v.owner, v.amount, v.unlockTime, v.withdrawn);
    }
}
```

---

## 3. Write and Explain Tests

### 3.1 Test Overview

**Explanation:** We test two vaults with different unlock times to ensure proper lock and release behavior.

### 3.2 Code Snippet: `test/TimeLockVault.t.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "forge-std/Test.sol";
import "../src/TimeLockVault.sol";

contract TimeLockVaultTest is Test {
    TimeLockVault vault;
    address user = address(0x1);

    function setUp() public {
        vm.deal(user, 1 ether);
        vm.prank(user);
        vault = new TimeLockVault();
        vault.createVault{value: 0.5 ether}(1, block.timestamp + 1 days);
        vault.createVault{value: 0.5 ether}(2, block.timestamp + 2 days);
    }

    function testCannotWithdrawEarly() public {
        vm.prank(user);
        vm.expectRevert("Locked");
        vault.withdrawVault(1);
    }

    function testWithdrawVault1AfterUnlock() public {
        vm.warp(block.timestamp + 1 days + 1);
        vm.prank(user);
        vault.withdrawVault(1);
        assertEq(user.balance, 0.5 ether);
    }

    function testWithdrawVault2AfterUnlock() public {
        vm.warp(block.timestamp + 2 days + 1);
        vm.prank(user);
        vault.withdrawVault(2);
        assertEq(user.balance, 0.5 ether);
    }
}
```

### 3.3 Run Tests

```bash
forge test
```

---

## 4. Configure & Deploy to Celo Alfajores

### 4.1 Configure CLI

**Explanation:** Point your Celo CLI to the Alfajores testnet.

```bash
npm install -g @celo/cli
celocli config:set --node https://alfajores-forno.celo-testnet.org
```

### 4.2 Deployment Script Snippet: `script/Deploy.s.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "forge-std/Script.sol";
import "../src/TimeLockVault.sol";

contract DeployTimeLock is Script {
    function run() external {
        vm.startBroadcast();
        new TimeLockVault(block.timestamp + 1 days){value: 0.1 ether}();
        vm.stopBroadcast();
    }
}
```

### 4.3 Deploy Command

```bash
export PRIVATE_KEY=<YOUR_ALFAJORES_KEY>
forge script script/Deploy.s.sol \
  --rpc-url https://alfajores-forno.celo-testnet.org \
  --private-key $PRIVATE_KEY \
  --broadcast
```

Copy the contract address from the logs.

---

## 5. Build the Next.js Frontend

### 5.1 Setup

**Explanation:** Scaffold a Next.js TypeScript app and install `ethers`.

```bash
npx create-next-app@latest frontend --typescript --src-dir
cd frontend
npm install ethers
```

### 5.2 Environment

```env
# frontend/.env.local
NEXT_PUBLIC_VAULT_ADDRESS=<YOUR_CONTRACT_ADDRESS>
```

### 5.3 UI Code Snippets (Step by Step)

#### 5.3.1 Import & ABI

**Explanation:** Load React hooks and ethers, and define the contract ABI.

```tsx
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

const abi = [
  'function createVault(uint256 id, uint256 _unlockTime) external payable',
  'function withdrawVault(uint256 id) external',
  'function getVault(uint256 id) view returns (address owner, uint256 amount, uint256 unlockTime, bool withdrawn)'
];
```

#### 5.3.2 Component & State Setup

**Explanation:** Initialize React component and state variables.

```tsx
export default function Home() {
  const [contract, setContract] = useState<ethers.Contract | null>(null);
  const [vaultId, setVaultId] = useState('');
  const [unlockTime, setUnlockTime] = useState('');
  const [status, setStatus] = useState<string | null>(null);
```

#### 5.3.3 Connect Wallet & Contract

**Explanation:** On mount, connect to Metamask and instantiate the contract.

```tsx
  useEffect(() => {
    if (typeof window !== 'undefined' && window.ethereum) {
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();
      const addr = process.env.NEXT_PUBLIC_VAULT_ADDRESS as string;
      setContract(new ethers.Contract(addr, abi, signer));
    }
  }, []);
```

#### 5.3.4 Create Vault Handler

**Explanation:** Send a transaction to `createVault`, passing ID, timestamp, and 0.1 CELO.

```tsx
  const handleCreate = async () => {
    if (!contract) return;
    const idNum = Number(vaultId);
    const unlockTs = Math.floor(new Date(unlockTime).getTime() / 1000);
    const tx = await contract.createVault(idNum, unlockTs, {
      value: ethers.utils.parseEther('0.1')
    });
    await tx.wait();
    setStatus(`Vault ${idNum} created, unlock at ${unlockTime}`);
  };
```

#### 5.3.5 Check Vault Status Handler

**Explanation:** Call `getVault` to fetch details and display them.

```tsx
  const handleStatus = async () => {
    if (!contract) return;
    const v = await contract.getVault(Number(vaultId));
    setStatus(
      `Owner: ${v.owner}
` +
      `Amount: ${ethers.utils.formatEther(v.amount)} CELO
` +
      `Unlock: ${new Date(v.unlockTime.toNumber() * 1000).toLocaleString()}
` +
      `Withdrawn: ${v.withdrawn}`
    );
  };
```

#### 5.3.6 Withdraw Handler

**Explanation:** Invoke `withdrawVault` once unlock time passed.

```tsx
  const handleWithdraw = async () => {
    if (!contract) return;
    const tx = await contract.withdrawVault(Number(vaultId));
    await tx.wait();
    setStatus(`Vault ${vaultId} withdrawn.`);
  };
```

#### 5.3.7 Render UI

**Explanation:** Build the form inputs, buttons, and status display.

```tsx
  return (
    <div style={{ maxWidth: 400, margin: '2rem auto', fontFamily: 'sans-serif' }}>
      <h1>Time-Locked Vault</h1>
      <label>Vault ID:</label>
      <input
        type="number"
        value={vaultId}
        onChange={e => setVaultId(e.target.value)}
        style={{ width: '100%', marginBottom: '1rem' }}
      />
      <label>Unlock Time:</label>
      <input
        type="datetime-local"
        value={unlockTime}
        onChange={e => setUnlockTime(e.target.value)}
        style={{ width: '100%', marginBottom: '1rem' }}
      />
      <button onClick={handleCreate} style={{ width: '100%', marginBottom: '0.5rem' }}>
        Create Vault (0.1 CELO)
      </button>
      <button onClick={handleStatus} style={{ width: '100%', marginBottom: '0.5rem' }}>
        Check Vault Status
      </button>
      <button onClick={handleWithdraw} style={{ width: '100%' }}>
        Withdraw
      </button>
      {status && (
        <pre style={{ background: '#f4f4f4', padding: '1rem', marginTop: '1rem' }}>
          {status}
        </pre>
      )}
    </div>
  );
}
```

### 5.4 Full Frontend File

```tsx
// src/pages/index.tsx
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

const abi = [
  'function createVault(uint256 id, uint256 _unlockTime) external payable',
  'function withdrawVault(uint256 id) external',
  'function getVault(uint256 id) view returns (address owner, uint256 amount, uint256 unlockTime, bool withdrawn)'
];

export default function Home() {
  const [contract, setContract] = useState<ethers.Contract | null>(null);
  const [vaultId, setVaultId] = useState('');
  const [unlockTime, setUnlockTime] = useState('');
  const [status, setStatus] = useState<string | null>(null);

  useEffect(() => {
    if (typeof window !== 'undefined' && window.ethereum) {
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      const signer = provider.getSigner();
      const addr = process.env.NEXT_PUBLIC_VAULT_ADDRESS as string;
      setContract(new ethers.Contract(addr, abi, signer));
    }
  }, []);

  const handleCreate = async () => {
    if (!contract) return;
    const idNum = Number(vaultId);
    const unlockTs = Math.floor(new Date(unlockTime).getTime() / 1000);
    const tx = await contract.createVault(idNum, unlockTs, { value: ethers.utils.parseEther('0.1') });
    await tx.wait();
    setStatus(`Vault ${idNum} created, unlock at ${unlockTime}`);
  };

  const handleStatus = async () => {
    if (!contract) return;
    const v = await contract.getVault(Number(vaultId));
    setStatus(
      `Owner: ${v.owner}
Amount: ${ethers.utils.formatEther(v.amount)} CELO` +
      `
Unlock: ${new Date(v.unlockTime.toNumber() * 1000).toLocaleString()}` +
      `
Withdrawn: ${v.withdrawn}`
    );
  };

  const handleWithdraw = async () => {
    if (!contract) return;
    const tx = await contract.withdrawVault(Number(vaultId));
    await tx.wait();
    setStatus(`Vault ${vaultId} withdrawn.`);
  };

  return (
    <div style={{ maxWidth: 400, margin: '2rem auto', fontFamily: 'sans-serif' }}>
      <h1>Time-Locked Vault</h1>
      <label>Vault ID:</label>
      <input type="number" value={vaultId} onChange={e => setVaultId(e.target.value)} style={{ width: '100%', marginBottom: '1rem' }} />
      <label>Unlock Time:</label>
      <input type="datetime-local" value={unlockTime} onChange={e => setUnlockTime(e.target.value)} style={{ width: '100%', marginBottom: '1rem' }} />
      <button onClick={handleCreate} style={{ width: '100%', marginBottom: '0.5rem' }}>Create Vault (0.1 CELO)</button>
      <button onClick={handleStatus} style={{ width: '100%', marginBottom: '0.5rem' }}>Check Vault Status</button>
      <button onClick={handleWithdraw} style={{ width: '100%' }}>Withdraw</button>
      {status && <pre style={{ background: '#f4f4f4', padding: '1rem', marginTop: '1rem' }}>{status}</pre>}
    </div>
  );
}
```

```tsx
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

const abi = [
  'function createVault(uint256 id, uint256 _unlockTime) external payable',
  'function withdrawVault(uint256 id) external',
  'function getVault(uint256 id) view returns (address owner, uint256 amount, uint256 unlockTime, bool withdrawn)'
];

export default function Home() {
  // ... (same as code snippet above) ...
}
```

---

## 6. Run Your Frontend

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000), connect your wallet, and interact with your vaults!
