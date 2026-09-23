# AccessControlRegistry

A secure, immutable role-based access control registry deployed on **Arc Testnet** (Circle's USDC-native blockchain). It lets a single owner manage named permissions for any address on-chain, hold a USDC treasury, and pause all role mutations in an emergency.

---

## What it does

- **Role management** — grant or revoke any `bytes32` role (e.g. `keccak256("OPERATOR")`) to/from any address. Every change emits a fully indexed event for off-chain monitoring.
- **USDC treasury** — the contract can hold USDC. Only the owner may withdraw it, protected by a reentrancy guard and `SafeERC20`.
- **Pause circuit breaker** — the owner can pause the contract at any time, blocking all role changes until unpaused.
- **Two-step ownership transfer** — the current owner nominates a new owner; the new owner must explicitly accept. `renounceOwnership` is disabled so the contract can never be left ownerless.

---

## Tech stack

| Layer | Choice |
|---|---|
| Language | Solidity `^0.8.20` (compiled with 0.8.28) |
| Framework | Foundry (forge build / forge test) |
| Libraries | OpenZeppelin 5.1.0 (`Ownable2Step`, `Pausable`, `ReentrancyGuard`, `SafeERC20`) |
| Chain | Arc Testnet (Chain ID: 5042002, EVM Paris) |
| Gas token | USDC (native on Arc) |
| Package manager | Bun |

---

## Project structure

```
contracts/
  AccessControlRegistry.sol        # Main contract
  AccessControlRegistry-design.md  # Full design doc (goals, threat model, testing)
  test/
    AccessControlRegistry.t.sol    # Foundry unit tests
  contract-metadata/
    AccessControlRegistry.json     # Deployment metadata (address, network, ABI path)
  out/
    AccessControlRegistry.sol/
      AccessControlRegistry.json   # Compiled artifact
src/                               # React + Vite frontend (not part of this contract)
foundry.toml                       # Foundry config (evm_version = "paris")
remappings.txt                     # Import remappings
```

---

## Quick start

```bash
# Install dependencies
bun install

# Build the contract
bun run contracts:build      # runs: forge build

# Run unit tests
bun run contracts:test       # runs: forge test -vvv

# Start the dev server (frontend)
bun run dev
```

---

## Contract interface

### Read

| Function | Returns | Description |
|---|---|---|
| `hasRole(bytes32 role, address account)` | `bool` | Check whether an address holds a given role |
| `owner()` | `address` | Current owner |
| `pendingOwner()` | `address` | Nominated next owner (zero if none pending) |
| `paused()` | `bool` | Whether the contract is currently paused |
| `usdcToken()` | `address` | USDC token address used by this registry |

### Write (owner only)

| Function | Description |
|---|---|
| `grantRole(bytes32 role, address account)` | Assign a role to an address (reverts if paused) |
| `revokeRole(bytes32 role, address account)` | Remove a role from an address (reverts if paused) |
| `pause()` | Pause all role mutations |
| `unpause()` | Resume role mutations |
| `withdrawUsdc(address to, uint256 amount)` | Withdraw USDC from the contract treasury |
| `transferOwnership(address newOwner)` | Nominate a new owner (two-step) |

### Write (pending owner only)

| Function | Description |
|---|---|
| `acceptOwnership()` | Complete the two-step ownership transfer |

### Events

```solidity
event RoleGranted(bytes32 indexed role, address indexed account, address indexed sender);
event RoleRevoked(bytes32 indexed role, address indexed account, address indexed sender);
event UsdcWithdrawn(address indexed to, uint256 amount);
```

---

## Security properties

- No non-owner can grant, revoke, or withdraw.
- Zero addresses cannot be granted roles.
- All role changes are blocked when paused.
- USDC withdrawal uses `SafeERC20` and is protected by `nonReentrant`.
- Ownership can never be renounced to the zero address.
- Two-step ownership transfer prevents accidental key loss.

See `contracts/AccessControlRegistry-design.md` for the full threat model, security table, and trust analysis.

---

## Deployment

Deployed via Circle Smart Contract Platform to **Arc Testnet**.  
Deployment metadata: `contracts/contract-metadata/AccessControlRegistry.json`

To get testnet USDC for interacting with the contract, use the **Get test USDC** button in the Arc Studio sidebar.

---

## License

MIT
