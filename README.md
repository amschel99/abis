### Admin Dashboard Requirements Document

---

#### **1. Introduction**
This document outlines requirements for an admin dashboard interacting with SPHR Token & Exchange APIs. The dashboard must enable secure monitoring/management of token operations, exchange parameters, role-based access, and high-risk contract functions.

---

#### **2. Functional Requirements**

##### **2.1 Authentication & Authorization**
- **Secure Admin Login**  
  - Role-based access (DEFAULT_ADMIN, PAUSER, UPGRADER, etc.)  
  - Integration with MetaMask or Web3 wallet for transaction signing (recommended) or secure input for private keys (with warnings).  
  - Session management with JWT or OAuth2 (if backend supports).

##### **2.2 SPHR Token Management**
- **Real-Time Metrics**  
  - Display total supply, transferable status, paused state, decimals, symbol.  
  - Role assignments (Minter, Burner, Pauser) with searchable address lists.  
- **Admin Actions**  
  - Mint/Burn tokens (with address/amount inputs).  
  - Toggle transferability/pause state.  
  - Grant/Revoke roles (input: role hash, target address, admin private key).  
  - Contract upgrades (UUPS) with confirmation dialogs.  

##### **2.3 SPHR Exchange Management**
- **Exchange Metrics**  
  - SPHR/WBERA reserves, current/minimum exchange rates, decay factor.  
  - Claim cooldown period, pending rewards per user.  
- **Admin Actions**  
  - Add/Remove operators.  
  - Update decay factor, minimum rate, or reward reserves.  
  - Withdraw SPHR/WBERA from the contract.  
  - Toggle decaying rewards.  

##### **2.4 Role & Security Management**
- **Role Explorer**  
  - View all roles (DEFAULT_ADMIN, OPERATOR, etc.) with assigned addresses.  
  - Check role permissions for any address.  
- **Audit Logs**  
  - Track role changes, contract upgrades, and critical transactions.  

##### **2.5 Contract Lifecycle Controls**
- **High-Risk Operations Section**  
  - Initialize/Upgrade contracts (with multi-step confirmation).  
  - Emergency pause/unpause for both Token and Exchange.  

##### **2.6 User & Claim Management**
- **User Lookup**  
  - Search by address to view SPHR balance, allowances, last burn timestamp, and claim eligibility.  
- **Manual Overrides**  
  - Admin-initiated token transfers (e.g., recover stuck funds).  

---

#### **3. Non-Functional Requirements**

##### **3.1 Security**
- **Private Key Handling**  
  - Transient storage (in-memory only) for keys used in transactions.  
  - Clear warnings about key exposure risks.  
- **Rate Limiting**  
  - Prevent abuse of sensitive endpoints (e.g., minting).  
- **HTTPS**  
  - Enforce encrypted communication with the backend.  

##### **3.2 Performance**
- **Real-Time Updates**  
  - Auto-refresh metrics every 30 seconds.  
- **Caching**  
  - Cache static data (e.g., role hashes, token addresses).  

##### **3.3 Usability**
- **Role-Based UI**  
  - Hide unauthorized functions (e.g., only UPGRADERs see contract upgrade buttons).  
- **Input Validation**  
  - Validate Ethereum addresses, hex-encoded roles, and token amounts.  
- **Error Handling**  
  - Display transaction revert reasons from APIs.  

##### **3.4 Compatibility**
- **Browser Support**  
  - Chrome, Firefox, Safari (latest versions).  
- **Mobile Responsiveness**  
  - Critical functions accessible on tablets/mobile.  

---

#### **4. API Integration Details**

##### **4.1 SPHR Token Endpoints**
- **Read Operations**  
  - Balance, allowances, role checks, contract state.  
- **Write Operations**  
  - Use POST `/admin/set-transferable`, `/admin/grant-role`, etc., with private keys in request bodies.  

##### **4.2 SPHR Exchange Endpoints**
- **Read Operations**  
  - Reserves, decay status, claim info.  
- **Write Operations**  
  - POST `/admin/update-decay-factor`, `/admin/transfer-sphr`, etc.  

##### **4.3 Edge Cases**
- Handle API downtime with graceful error messages.  
- Validate SPHR/WBERA amounts against decimals (e.g., 18 decimals for SPHR).  

---

#### **5. Security Considerations**
- **Critical**  
  - Avoid storing private keys in the frontend. Recommend backend signing via MetaMask.  
  - Log all admin actions for audit trails.  
- **Warnings**  
  - Highlight irreversible actions (e.g., contract upgrades) with confirmation modals.  

---

#### **6. Mockups (Optional)**
- **Dashboard Layout**  
  - Sidebar with sections: Token, Exchange, Roles, Lifecycle.  
  - Metrics dashboard with cards for key stats (total supply, reserves, paused status).  
- **Forms**  
  - Input fields for addresses, amounts, role hashes, and private keys (masked).  

---

#### **7. Deliverables**
1. React/Next.js frontend with TypeScript.  
2. Integration tests for critical API calls.  
3. Deployment scripts for Vercel/Netlify.  
4. Documentation for role-based access setup.  

---

**Approvals**  
*Stakeholder sign-off required before development.*
