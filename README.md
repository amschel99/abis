

# **SPHR Token Requirements Document**

## **1. Core Features**
### **A. Transfer Restrictions**
- The SPHR token **cannot be transferred directly** between users (e.g., via `transfer()` or `transferFrom()`).
- Transfers are **only allowed**:
  1. When interacting with Uniswap (e.g., swaps, liquidity provisioning).
  2. When initiated by the `admin` (e.g., for airdrops, liquidity setup).
- All other transfers are blocked.

### **B. Uniswap Integration**
- Users can **swap SPHR ↔ BERA** via Uniswap V2/V3.
- Users can **add/remove liquidity** through Uniswap (but liquidity provisioning can be restricted to admin if desired).

### **C. Admin Controls**
- **Upgradable Logic**: Admin can modify transfer rules (e.g., allow trading on other DEXs later).
- **Whitelist Management**: Admin can add/remove addresses exempt from transfer restrictions (e.g., future DEX routers).

---

## **2. Technical Implementation**

### **A. Modified ERC-20 Contract**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract RestrictedSPHR is ERC20, Ownable {
    address public immutable uniswapRouter;
    mapping(address => bool) public whitelist;

    constructor(address _uniswapRouter) ERC20("SPHERE", "SPHR") {
        uniswapRouter = _uniswapRouter;
        _mint(msg.sender, 1_000_000 * 10**18); // Initial supply to admin
        whitelist[_uniswapRouter] = true; // Uniswap router is whitelisted
        whitelist[msg.sender] = true; // Admin is whitelisted
    }

    // Override transfer functions with restrictions
    function _transfer(address sender, address recipient, uint256 amount) internal override {
        require(
            whitelist[sender] || whitelist[recipient],
            "SPHR: Transfers restricted to Uniswap/admin"
        );
        super._transfer(sender, recipient, amount);
    }

    // Admin functions
    function updateWhitelist(address _address, bool _status) external onlyOwner {
        whitelist[_address] = _status;
    }
}
```

### **B. Upgradeability (Optional)**
Use the **Transparent Proxy Pattern** (OpenZeppelin) to allow the admin to upgrade the contract logic later:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";

// Deploy this proxy contract, pointing to the initial RestrictedSPHR logic.
// Admin can upgrade to a new contract later.
```

---

## **3. Workflow**
### **A. Admin Setup**
1. Deploy `RestrictedSPHR` with the Uniswap router address (e.g., `0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D`).
2. Add liquidity to Uniswap (admin-only):
   - Approve SPHR and BERA for the router.
   - Call `addLiquidityETH` or `addLiquidity` via Uniswap.

### **B. User Swaps**
1. User approves Uniswap router to spend their SPHR.
2. User swaps via Uniswap interface (transfer allowed because router is whitelisted).

### **C. Upgrading Rules**
1. Deploy a new version of `RestrictedSPHR` with updated logic.
2. Admin calls `upgradeTo(newLogicContract)` on the proxy contract.

---

## **4. Key Checks**
### **Test Cases**
1. **Block Regular Transfers**:
   - User A tries to send SPHR to User B → Reverts.
2. **Allow Uniswap Swaps**:
   - User swaps SPHR → BERA via Uniswap → Succeeds.
3. **Admin Liquidity Provisioning**:
   - Admin adds liquidity → Succeeds.
4. **Upgrade Test**:
   - Deploy new logic allowing direct transfers → Verify upgrade works.

---

## **5. Security Considerations**
- **Immutable Router**: The Uniswap router address is set at deployment and cannot be changed (prevents rug-pulls).
- **Proxy Admin**: Use a multisig or timelock for upgradeability to prevent abuse.
- **Whitelist Audits**: Ensure only trusted addresses (e.g., Uniswap) are whitelisted.

---

## **6. Deliverables**
1. `RestrictedSPHR.sol` (ERC-20 with transfer restrictions).
2. Proxy deployment scripts (if using upgradeability).
3. Test suite (Hardhat/Foundry) covering transfer rules.
4. Deployment guide for Berachain/Uniswap.

---

## **7. Example User Flow**
1. **Admin**:
   - Deploys `RestrictedSPHR` and proxy (if upgradable).
   - Adds SPHR-BERA liquidity on Uniswap.
2. **User**:
   - Buys SPHR via Uniswap (allowed via router whitelist).
   - Cannot send SPHR to another wallet directly (blocked by contract).

---
