

# **Requirements Document: SPHR Token & Uniswap Integration (Berachain/EVM)**

## **1. Objectives**
- Deploy the SPHR (ERC-20) token on Berachain (or Ethereum testnet).
- Allow **admin-only** functions to add liquidity to SPHR-BERA pools on Uniswap V2.
- Enable users to swap SPHR ↔ BERA via the pool.
- Lay groundwork for future cross-chain expansion (Cosmos/MANTRA).

---

## **2. Technical Scope**

### **A. Token Contract (SPHR.sol)**
- **ERC-20 Standard** with minting/burning.
- **Admin Controls**:
  - Only owner can mint tokens (for airdrops/liquidity).
  - Optionally renounce ownership after setup.
- **Initial Supply**: 1,000,000 SPHR (18 decimals).
- **Code**: Based on OpenZeppelin (see Phase 1 example).

### **B. Liquidity Pool (Uniswap V2)**
- **Admin-Only Liquidity Provisioning**:
  - Restrict `addLiquidity` to owner (to prevent rogue LPs).
  - Use `UniswapV2Router02` to create SPHR-BERA pool.
- **Steps**:
  1. Swap 500 USDC → BERA via Uniswap.
  2. Approve SPHR and BERA for router.
  3. Call `addLiquidity()` with 400,000 SPHR + equivalent BERA.

### **C. Swap Functionality**
- Public swaps via Uniswap interface.
- Users can trade SPHR ↔ BERA once liquidity exists.

### **D. Security**
- **Testnet First**: Deploy to Berachain Artio (testnet) or Sepolia.
- **Slippage Control**: Set min amounts in `addLiquidity` (e.g., 1% slippage).
- **Emergency Withdraw**: Admin can remove liquidity if needed.

---

## **3. Deliverables**
1. **Smart Contracts**:
   - `SPHR.sol` (ERC-20 with minting).
   - Optional helper contract for admin liquidity management.
2. **Scripts** (Hardhat/Foundry):
   - `deploy.js`: Deploy SPHR token.
   - `addLiquidity.js`: Fund pool (admin-only).
   - `swap.js`: Test token swaps.
3. **Test Cases**:
   - Verify minting restrictions.
   - Test liquidity addition and swaps.
4. **Documentation**:
   - Setup guide for testnet/mainnet deployment.
   - User instructions for swapping.

---

## **4. Tools & Libraries**
- **Development**:
  - **Hardhat/Foundry** (EVM framework).
  - **OpenZeppelin** (ERC-20 templates).
  - **Uniswap V2 Interfaces** (`IUniswapV2Router02`, `IUniswapV2Factory`).
- **Testing**:
  - Berachain Artio (testnet) or Sepolia.
  - Testnet USDC/BERA (use faucets).

---

## **5. User Flow**
1. **Admin**:
   - Deploys SPHR.
   - Mints 400,000 SPHR + swaps USDC → BERA.
   - Calls `addLiquidity()` to create SPHR-BERA pool.
2. **Users**:
   - Claim SPHR via airdrop (future phase).
   - Swap SPHR ↔ BERA on Uniswap.

---

## **6. Example Code Snippets**

### **SPHR Token (SPHR.sol)**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract SPHR is ERC20, Ownable {
    constructor() ERC20("SPHERE", "SPHR") {
        _mint(msg.sender, 1_000_000 * 10**18);
    }
    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
```

### **Adding Liquidity (addLiquidity.js)**
```javascript
const { ethers } = require("hardhat");
const UNISWAP_ROUTER = "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"; // Uniswap V2 Router

async function main() {
  const [admin] = await ethers.getSigners();
  const router = await ethers.getContractAt("IUniswapV2Router02", UNISWAP_ROUTER);

  // Approve SPHR and BERA for router
  await sphr.approve(router.address, "400000000000000000000000"); // 400k SPHR
  await bera.approve(router.address, BERA_AMOUNT);

  // Add liquidity
  await router.addLiquidity(
    sphr.address,
    bera.address,
    "400000000000000000000000", // 400k SPHR
    BERA_AMOUNT,
    0, 0, // min amounts (slippage tolerance)
    admin.address,
    Math.floor(Date.now() / 1000) + 300 // deadline
  );
}
```

---

## **7. Timeline & Milestones**
1. **Week 1**: Token deployment + basic swaps (testnet).
2. **Week 2**: Admin liquidity controls + tests.
3. **Week 3**: Audit prep + mainnet deployment (if testnet succeeds).

---

## **8. Next Steps (Post-This-Phase)**
- Cross-chain bridge to Cosmos (MANTRA).
- Airdrop contract (Merkle distributor).
- Frontend for swaps/claims.

---

### **Notes for the Developer**
- Prioritize **testnet validation** before mainnet.
- Use **OpenZeppelin Audited Contracts** for safety.
- Document all admin private keys/APIs securely.

Let me know if you’d like to adjust the scope or add more features!
