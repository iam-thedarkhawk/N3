# N3 Security Scanner - System Architecture

## High-Level Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           N3 SECURITY SCANNER                               |
│                    Template-Based Vulnerability Detection                   │
└─────────────────────────────────────────────────────────────────────────────┘

                                    ▼

┌─────────────────────────────────────────────────────────────────────────────┐
│                           CORE ENGINE (packages/core/)                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                    TEMPLATE LIBRARY (40+ YAML Files)                 │   │
│  ├──────────────────────────────────────────────────────────────────────┤   │
│  │                                                                      │   │
│  │  📁 SWC Templates (20+)          📁 DeFi Templates (10+)            │   │
│  │  ├─ reentrancy-001.yaml          ├─ flash-loan-001.yaml              │   │
│  │  ├─ integer-overflow-001.yaml    ├─ oracle-manipulation.yaml         │   │
│  │  ├─ access-control-001.yaml      ├─ frontrunning-001.yaml            │   │
│  │  ├─ unchecked-call.yaml          ├─ liquidity-drain.yaml             │   │
│  │  └─ ... (16 more)                └─ ... (6 more)                     │   │
│  │                                                                      │   │
│  │  📁 Smart Contract Templates (10+)                                	  │   │
│  │  ├─ upgrade-proxy-001.yaml                                           │   │
│  │  ├─ delegatecall-001.yaml                                            │   │
│  │  ├─ selfdestruct-001.yaml                                            │   │
│  │  └─ ... (7 more)                                                     │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                         SCANNING ENGINE                              │   │
│  ├──────────────────────────────────────────────────────────────────────┤   │
│  │  • Template Parser (YAML → Executable Rules)                         │   │
│  │  • Pattern Matcher (Solidity AST Analysis)                           │   │
│  │  • Risk Calculator (Severity × Impact × Exploitability)              │   │
│  │  • CVE Mapper (Known Vulnerabilities Database)                       │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

                                    ▼
                    ┌───────────────┴───────────────┐
                    │   DELIVERY PLATFORMS (3)      │
                    └───────────────┬───────────────┘
                                    ▼

        ┌───────────────────┬───────────────────┬───────────────────┐
        │                   │                   │                   │
        ▼                   ▼                   ▼                   ▼

┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   HARDHAT       │ │   BLOCKSCOUT    │ │     ENVIO       │ │   MCP SERVER    │
│   INTEGRATION   │ │   INTEGRATION   │ │   INTEGRATION   │ │   INTEGRATION   │
├─────────────────┤ ├─────────────────┤ ├─────────────────┤ ├─────────────────┤
│                 │ │                 │ │                 │ │                 │
│  Compilation    │ │  Widget UI      │ │  Real-Time      │ │  AI-Powered     │
│    Time Scan    │ │    Embedded     │ │    Indexing     │ │    Analysis     │
│                 │ │                 │ │                 │ │                 │
│ • Runs during   │ │ • Contract page │ │ • Event stream  │ │ • Natural lang. │
│   npx hardhat   │ │   security      │ │   monitoring    │ │   explanations  │
│   compile       │ │   badge         │ │ • GraphQL API   │ │ • Blockscout    │
│                 │ │                 │ │                 │ │   API proxy     │
│ • Blocks bad    │ │ • Live risk     │ │ • Analytics     │ │                 │
│   deployments   │ │   score         │ │   dashboard     │ │ • Security      │
│                 │ │                 │ │                 │ │   prompts       │
│ • Risk report   │ │ • Vulnerability │ │ • Alert system  │ │                 │
│   in terminal   │ │   list          │ │                 │ │                 │
│                 │ │                 │ │                 │ │                 │
└─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────┘
        │                   │                   │                   │
        └───────────────────┴───────────────────┴───────────────────┘
                                    │
                                    ▼
                        ┌───────────────────────┐
                        │   STORAGE LAYER       │
                        ├───────────────────────┤
                        │ • Smart Contract      │
                        │   (N3SecurityOracle)  │
                        │ • PostgreSQL DB       │
                        │ • Hasura GraphQL      │
                        └───────────────────────┘
```

---

## Detailed Component Architecture

### 1️⃣ CORE ENGINE - Template System

```
packages/core/
│
├── templates/                          ← 40+ YAML vulnerability definitions
│   ├── SWC/
│   │   ├── SWC-107-reentrancy.yaml
│   │   ├── SWC-101-integer-overflow.yaml
│   │   ├── SWC-105-unprotected-ether.yaml
│   │   └── ... (17 more)
│   │
│   ├── defi/
│   │   ├── flash-loan-attack.yaml
│   │   ├── oracle-manipulation.yaml
│   │   ├── frontrunning.yaml
│   │   └── ... (7 more)
│   │
│   └── smart-contract/
│       ├── delegatecall-injection.yaml
│       ├── selfdestruct-abuse.yaml
│       └── ... (8 more)
│
├── src/
│   ├── engine.ts                       ← Main scanning orchestrator
│   ├── parser.ts                       ← Solidity AST parser
│   ├── cve-scanner.ts                  ← Template matcher
│   ├── cve-parser.ts                   ← YAML → Rules converter
│   └── index.ts                        ← Public API
│
└── package.json
```

**Template Structure (YAML):**
```yaml
id: reentrancy-001
name: Reentrancy Vulnerability
severity: CRITICAL
category: SWC-107
description: Contract calls external address before state update
impact: Allows attacker to drain contract funds
remediation: Use checks-effects-interactions pattern

patterns:
  - pattern: call.value(...)
    before: storage_write
    risk: HIGH
    
  - pattern: delegatecall(...)
    context: external_contract
    risk: CRITICAL

test_cases:
  - vulnerable: |
      function withdraw() public {
        msg.sender.call.value(balance[msg.sender])("");
        balance[msg.sender] = 0;  // ⚠️ State change AFTER external call
      }
      
  - safe: |
      function withdraw() public {
        uint amount = balance[msg.sender];
        balance[msg.sender] = 0;  // ✅ State change BEFORE external call
        msg.sender.call.value(amount)("");
      }
```

---

### 2️⃣ HARDHAT INTEGRATION - Build-Time Scanning

```
packages/hardhat-plugin/
│
├── src/
│   ├── index.ts                        ← Hardhat plugin entry point
│   ├── tasks/
│   │   ├── compile-hook.ts             ← Intercepts compilation
│   │   ├── scan-task.ts                ← Manual scan command
│   │   └── report-task.ts              ← Generate reports
│   │
│   └── utils/
│       ├── contract-loader.ts          ← Load compiled artifacts
│       └── report-formatter.ts         ← Terminal output formatting
│
└── package.json

┌─────────────────────────────────────────────────────────────────┐
│                    HARDHAT WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Developer runs:                                                │
│  $ npx hardhat compile                                          │
│                                                                 │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────┐                                           │
│  │ Solidity Compile │                                           │
│  └──────────────────┘                                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────┐                                           │
│  │ N3 Plugin Hook   │  ← Runs automatically                     │
│  └──────────────────┘                                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Load Templates (40+ YAML)        │                           │
│  │ Parse Compiled Bytecode + AST    │                           │
│  │ Match Vulnerability Patterns     │                           │
│  │ Calculate Risk Score             │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────┐                                           │
│  │ Terminal Report  │                                           │
│  ├──────────────────┤                                           │
│  │  0 Critical      │                                           │
│  │  2 High          │                                           │
│  │  Risk: 65/100    │                                           │
│  └──────────────────┘                                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────┐                                           │
│  │ Block Deploy?    │  ← If risk > threshold                    │
│  └──────────────────┘                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Usage Example:**
```javascript
// hardhat.config.js
require('@n3/hardhat-plugin');

module.exports = {
  solidity: "0.8.20",
  n3: {
    enabled: true,
    threshold: 70,          // Fail compilation if score < 70
    templates: './templates', // Custom template directory
    output: './reports',    // Save scan reports
  }
};
```

---

### 3️⃣ BLOCKSCOUT INTEGRATION - Explorer Widget

```
packages/blockscout-widget/
│
├── src/
│   ├── widget.tsx                      ← React widget component
│   ├── api-client.ts                   ← Blockscout API wrapper
│   └── index.ts                        ← Export bundle
│
├── rollup.config.js                    ← Build UMD bundle
└── package.json

┌─────────────────────────────────────────────────────────────────┐
│              BLOCKSCOUT WIDGET ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User views contract on Blockscout:                             │
│  https://eth.blockscout.com/address/0x123...                    │
│                                                                 │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │  Blockscout Contract Page        │                           │
│  │  ┌────────────────────────────┐  │                           │
│  │  │ Contract Details           │  │                           │
│  │  │ • Name: TetherToken        │  │                           │
│  │  │ • Verified:                │  │                           │
│  │  │ • Compiler: v0.4.18        │  │                           │
│  │  └────────────────────────────┘  │                           │
│  │                                  │                           │
│  │  ┌────────────────────────────┐  │                           │
│  │  │  N3 Security Widget        │  │  ← Embedded via <script>  │
│  │  ├────────────────────────────┤  │                           │
│  │  │ Risk Score: 45/100         │  │                           │
│  │  │                            │  │                           │
│  │  │   3 Critical Issues:       │  │                           │
│  │  │ • Reentrancy in withdraw() │  │                           │
│  │  │ • Missing access control   │  │                           │
│  │  │ • Integer overflow risk    │  │                           │
│  │  │                            │  │                           │
│  │  │ [View Full Report]         │  │                           │
│  │  │ [Request Audit]            │  │                           │
│  │  └────────────────────────────┘  │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Widget calls MCP Server API      │                           │
│  │ POST /api/analyze                │                           │
│  │ { contractAddress: "0x123..." }  │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ MCP Server:                      │                           │
│  │ 1. Fetch code from Blockscout    │                           │
│  │ 2. Run N3 template scan          │                           │
│  │ 3. Return JSON response          │                           │
│  └──────────────────────────────────┘                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Widget Integration:**
```html
<!-- On Blockscout contract page -->
<div id="n3-security-widget"></div>

<script src="https://cdn.n3security.io/widget.js"></script>
<script>
  N3Widget.render({
    containerId: 'n3-security-widget',
    contractAddress: '{{contract.address}}',
    network: '{{chain.name}}',
    theme: 'light'
  });
</script>
```

---

### 4️⃣ ENVIO INTEGRATION - Real-Time Monitoring

```
packages/envio-indexer/
│
├── config.yaml                         ← Network & contract config
├── schema.graphql                      ← GraphQL schema definition
│
├── src/
│   └── EventHandlers.ts                ← Event processing logic
│
├── abis/
│   └── n3-security-oracle-abi.json    ← Contract ABI
│
└── generated/                          ← Auto-generated TypeScript types
    ├── src/
    │   ├── Types.gen.ts
    │   └── Handlers.gen.ts
    └── package.json

┌─────────────────────────────────────────────────────────────────┐
│                ENVIO REAL-TIME INDEXING FLOW                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Smart Contract emits event:                                    │
│  N3SecurityOracle.VulnerabilityDetected(...)                    │
│                                                                 │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Envio HyperIndex (Listening)     │                           │
│  │ • Monitors blockchain events     │                           │
│  │ • Decodes event parameters       │                           │
│  │ • Triggers handler function      │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ EventHandler.ts                  │                           │
│  │ VulnerabilityDetected.handler()  │                           │
│  │ {                                │                           │
│  │   contractAddress: "0x123...",   │                           │
│  │   vulnType: "reentrancy",        │                           │
│  │   severity: "CRITICAL",          │                           │
│  │   timestamp: 1698264523          │                           │
│  │ }                                │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ PostgreSQL Database              │                           │
│  │ ┌──────────────────────────────┐ │                           │
│  │ │ vulnerability_event table    │ │                           │
│  │ │ • id                         │ │                           │
│  │ │ • contract_address           │ │                           │
│  │ │ • vulnerability_type         │ │                           │
│  │ │ • severity                   │ │                           │
│  │ │ • detected_at                │ │                           │
│  │ │ • block_number               │ │                           │
│  │ └──────────────────────────────┘ │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Hasura GraphQL API               │                           │
│  │ Auto-generated from DB schema    │                           │
│  │ http://localhost:8080/v1/graphql │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Security Dashboard (Frontend)    │                           │
│  │ • Real-time vulnerability feed   │                           │
│  │ • Analytics charts               │                           │
│  │ • Alert notifications            │                           │
│  └──────────────────────────────────┘                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**GraphQL Query Example:**
```graphql
query GetCriticalVulnerabilities {
  VulnerabilityEvent(
    where: {
      severity: {_eq: "CRITICAL"},
      detectedAt: {_gte: "2025-10-01"}
    }
    order_by: {detectedAt: desc}
  ) {
    id
    contractAddress
    vulnerabilityType
    severity
    detectedAt
    blockNumber
  }
}

query GetSecurityMetrics {
  SecurityMetric(
    order_by: {criticalCount: desc}
    limit: 10
  ) {
    id
    totalScans
    criticalCount
    highCount
    mediumCount
    lowCount
  }
}
```

---

### 5️⃣ MCP SERVER - AI Integration + Blockscout Proxy

```
mcp-blockscout-server.mjs
│
├── Express HTTP Server (Port 3000)
│   │
│   ├── GET  /                          ← Health check
│   ├── POST /api/analyze               ← Enhanced analysis
│   ├── GET  /api/report/:address       ← Full security report
│   └── POST /api/prompt                ← MCP prompts
│
├── Blockscout API Client
│   │
│   ├── queryBlockscout()               ← Direct API calls
│   ├── getContractInfo()               ← Contract verification
│   └── getTransactionHistory()         ← Recent transactions
│
├── N3 Template Scanner
│   │
│   └── scanContract()                  ← Run vulnerability templates
│
└── MCP Protocol Handler
    │
    └── generatePrompt()                ← AI-friendly explanations

┌─────────────────────────────────────────────────────────────────┐
│                  MCP SERVER REQUEST FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Client (Widget/CLI/AI):                                        │
│  POST /api/analyze                                              │
│  { contractAddress: "0xdAC1..." }                               │
│                                                                 │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ MCP Server                       │                           │
│  │ Step 1: Fetch from Blockscout    │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Blockscout API                   │                           │
│  │ GET /api?module=contract&action= │                           │
│  │     getsourcecode&address=0x...  │                           │
│  │                                  │                           │
│  │ Returns:                         │                           │
│  │ • Source code                    │                           │
│  │ • Compiler version               │                           │
│  │ • Optimization settings          │                           │
│  │ • Contract name                  │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ MCP Server                       │                           │
│  │ Step 2: Run N3 Template Scan     │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ N3 Core Engine                   │                           │
│  │ • Load 40+ templates             │                           │
│  │ • Parse Solidity code            │                           │
│  │ • Match vulnerability patterns   │                           │
│  │ • Calculate risk score           │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ MCP Server                       │                           │
│  │ Step 3: Generate AI Prompt       │                           │
│  └──────────────────────────────────┘                           │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────┐                           │
│  │ Response:                        │                           │
│  │ {                                │                           │
│  │   contractAddress: "0xdAC1...",  │                           │
│  │   blockscout: {                  │                           │
│  │     verified: true,              │                           │
│  │     contractName: "TetherToken", │                           │
│  │     compiler: "v0.4.18"          │                           │
│  │   },                             │                           │
│  │   n3Security: {                  │                           │
│  │     riskScore: 45,               │                           │
│  │     vulnerabilities: [           │                           │
│  │       {                          │                           │
│  │         type: "reentrancy",      │                           │
│  │         severity: "CRITICAL",    │                           │
│  │         location: "withdraw()",  │                           │
│  │         remediation: "Use CEI"   │                           │
│  │       }                          │                           │
│  │     ]                            │                           │
│  │   },                             │                           │
│  │   aiPrompt: "This contract..."   │                           │
│  │ }                                │                           │
│  └──────────────────────────────────┘                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagram

```
┌─────────────┐
│  Developer  │
└──────┬──────┘
       │
       │ (1) Writes Solidity code
       │
       ▼
┌──────────────────────────────────────┐
│   Smart Contract Source Code         │
│   • VulnerableBank.sol               │
│   • SecureBank.sol                   │
└──────┬───────────────────────────────┘
       │
       │ (2) npx hardhat compile
       │
       ▼
┌──────────────────────────────────────┐
│   Hardhat Compilation                │
│   ┌────────────────────────────────┐ │
│   │ N3 Plugin Hook (Automatic)     │ │
│   └────────────────────────────────┘ │
└──────┬───────────────────────────────┘
       │
       │ (3) Load templates & scan
       │
       ▼
┌──────────────────────────────────────┐
│   N3 Core Engine                     │
│   ┌────────────────────────────────┐ │
│   │ • Parse AST                    │ │
│   │ • Match 40+ templates          │ │
│   │ • Calculate risk score         │ │
│   └────────────────────────────────┘ │
└──────┬───────────────────────────────┘
       │
       ├──────────────────┬─────────────────┬─────────────────┐
       │                  │                 │                 │
       ▼                  ▼                 ▼                 ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  Terminal   │   │  Contract   │   │   Envio     │   │ MCP Server  │
│   Report    │   │  Deployment │   │  Indexer    │   │   API       │
├─────────────┤   ├─────────────┤   ├─────────────┤   ├─────────────┤
│ Risk: 45/100│   │ Deploy to   │   │ Listen for  │   │ Serve to    │
│  CRITICAL   │   │ Blockchain  │   │ Events      │   │ Blockscout  │
│             │   │             │   │             │   │   Widget    │
│ Vulnera-    │   │ Emit        │   │ Store in    │   │             │
│ bilities:   │   │ Security    │   │ PostgreSQL  │   │ Generate    │
│ • Reentr... │   │ Events      │   │             │   │ AI Prompts  │
└─────────────┘   └──────┬──────┘   └──────┬──────┘   └─────────────┘
                         │                 │
                         └────────┬────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │  N3SecurityOracle.sol    │
                    │  (On-Chain Storage)      │
                    ├──────────────────────────┤
                    │ Events:                  │
                    │ • VulnerabilityDetected  │
                    │ • SecurityScanCompleted  │
                    │ • ContractDeployed       │
                    └──────────────────────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │  Envio + Hasura          │
                    │  GraphQL API             │
                    ├──────────────────────────┤
                    │ query {                  │
                    │   VulnerabilityEvent {   │
                    │     contractAddress      │
                    │     severity             │
                    │   }                      │
                    │ }                        │
                    └──────────────────────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │  Security Dashboard      │
                    │  (Real-Time Analytics)   │
                    └──────────────────────────┘
```

---

## Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                      TECHNOLOGY STACK                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   CORE ENGINE                                                   │
│  ├─ TypeScript 5.2                                              │
│  ├─ Solidity Parser (@solidity-parser/parser)                   │
│  ├─ YAML Parser (js-yaml)                                       │
│  └─ AST Walker (Custom implementation)                          │
│                                                                 │
│   HARDHAT INTEGRATION                                           │
│  ├─ Hardhat 3.0.9                                               │
│  ├─ @nomicfoundation/hardhat-ethers 4.0.2                       │
│  ├─ Ethers.js 6.8.0                                             │
│  └─ Node.js 22.21.0                                             │
│                                                                 │
│   BLOCKSCOUT INTEGRATION                                        │
│  ├─ React 18.2                                                  │
│  ├─ TypeScript 5.2                                              │
│  ├─ Rollup (UMD bundler)                                        │
│  ├─ Blockscout API (REST)                                       │
│  └─ Express 5.1.0 (MCP Server)                                  │
│                                                                 │
│   ENVIO INTEGRATION                                             │
│  ├─ Envio CLI 2.31.0                                            │
│  ├─ PostgreSQL 17.5                                             │
│  ├─ Hasura GraphQL 2.43.0                                       │
│  ├─ Docker Compose                                              │
│  └─ TypeScript Event Handlers                                   │
│                                                                 │
│   MCP SERVER                                                    │
│  ├─ @modelcontextprotocol/sdk 1.20.2                            │
│  ├─ Express 5.1.0                                               │
│  ├─ Axios (HTTP client)                                         │
│  └─ CORS + Body-parser                                          │
│                                                                 │
│   BLOCKCHAIN                                                    │
│  ├─ Solidity 0.8.20                                             │
│  ├─ Hardhat Local Node                                          │
│  ├─ Ethers.js 6.8.0                                             │
│  └─ OpenZeppelin Contracts                                      │
│                                                                 │
│   PACKAGE MANAGEMENT                                            │
│  ├─ pnpm 10.19.0 (Workspace)                                    │
│  ├─ npm 10.9.4                                                  │
│  └─ Monorepo Structure                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCTION DEPLOYMENT                        │
└─────────────────────────────────────────────────────────────────┘

                          ┌──────────────┐
                          │   USERS      │
                          └──────┬───────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
        ┌─────────────────────┐   ┌─────────────────────┐
        │  Blockscout Widget  │   │  Developer CLI      │
        │  (Browser)          │   │  (Local Machine)    │
        └──────────┬──────────┘   └──────────┬──────────┘
                   │                         │
                   │ HTTPS                   │ npx n3
                   │                         │
                   ▼                         ▼
        ┌──────────────────────────────────────────────┐
        │         MCP SERVER (Cloud)                   │
        │         https://api.n3security.io            │
        │                                              │
        │  ┌─────────────────────────────────────────┐ │
        │  │ Load Balancer (Nginx)                   │ │
        │  └─────────────────────────────────────────┘ │
        │                    │                         │
        │       ┌────────────┴────────────┐            │
        │       │                         │            │
        │       ▼                         ▼            │
        │  ┌─────────┐              ┌─────────┐        │
        │  │ Node.js │              │ Node.js │        │
        │  │ Server  │              │ Server  │        │
        │  │ (Pod 1) │              │ (Pod 2) │        │
        │  └─────────┘              └─────────┘        │
        └────────────────┬─────────────────────────────┘
                         │
            ┌────────────┴────────────┐
            │                         │
            ▼                         ▼
  ┌───────────────────┐    ┌───────────────────┐
  │ Blockscout API    │    │ Blockchain RPC    │
  │ (External)        │    │ (Infura/Alchemy)  │
  └───────────────────┘    └───────────────────┘
            │
            │
            ▼
  ┌───────────────────┐
  │ N3 Core Engine    │
  │ (Container)       │
  │ • Template Cache  │
  │ • Scan Workers    │
  └───────────────────┘
            │
            ▼
  ┌───────────────────┐
  │ PostgreSQL        │
  │ (Managed DB)      │
  │ • Scan Results    │
  │ • Analytics       │
  └───────────────────┘
            │
            ▼
  ┌───────────────────┐
  │ Hasura GraphQL    │
  │ (Cloud)           │
  │ • Public API      │
  │ • Real-time Subs  │
  └───────────────────┘
```

---

## Security & Privacy Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   SECURITY CONSIDERATIONS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   DATA PRIVACY                                                  │
│  ├─  Source code analysis is stateless (no storage)             │
│  ├─  Results cached with TTL (24 hours)                         │
│  ├─  No private keys stored                                     │
│  └─  Public blockchain data only                                │
│                                                                 │
│   API SECURITY                                                  │
│  ├─  Rate limiting (100 req/min per IP)                         │
│  ├─  API key authentication (premium tier)                      │
│  ├─  CORS policies enforced                                     │
│  └─  HTTPS only in production                                   │
│                                                                 │
│   TEMPLATE INTEGRITY                                            │
│  ├─  YAML schema validation                                     │
│  ├─  Template signatures (community submissions)                │
│  ├─  Version control (Git history)                              │
│  └─  Peer review process                                        │
│                                                                 │
│   SCAN ACCURACY                                                 │
│  ├─  Multiple template sources (SWC, CVE, DeFi hacks)           │
│  ├─  Conservative risk scoring (minimize false negatives)       │
│  ├─  Template test cases (vulnerable vs safe code)              │
│  └─  Disclaimer: Tool assistance, not audit guarantee           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Scalability Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCALABILITY STRATEGY                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   HORIZONTAL SCALING                                            │
│  ├─ MCP Server: Kubernetes pods (auto-scale)                    │
│  ├─ Scan Workers: Queue-based (RabbitMQ/Redis)                  │
│  ├─ Database: PostgreSQL read replicas                          │
│  └─ CDN: CloudFlare for widget bundle                           │
│                                                                 │
│   PERFORMANCE OPTIMIZATION                                      │
│  ├─ Template caching (in-memory Redis)                          │
│  ├─ Contract code caching (24h TTL)                             │
│  ├─ Lazy template loading (load on-demand)                      │
│  └─ Parallel pattern matching (Worker threads)                  │
│                                                                 │
│   DATA MANAGEMENT                                               │
│  ├─ Scan results: 90-day retention                              │
│  ├─ Analytics: Time-series DB (InfluxDB)                        │
│  ├─ Logs: Centralized (ELK stack)                               │
│  └─ Backups: Daily snapshots                                    │
│                                                                 │
│   MONITORING                                                    │
│  ├─ Prometheus metrics                                          │
│  ├─ Grafana dashboards                                          │
│  ├─ Error tracking (Sentry)                                     │
│  └─ Uptime monitoring (UptimeRobot)                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Future Architecture Extensions

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROADMAP ARCHITECTURE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Q1 2026 - Enhanced Analysis                                   │
│  ├─ Machine Learning integration                                │
│  │  ├─ Pattern discovery (unsupervised learning)                │
│  │  ├─ False positive reduction                                 │
│  │  └─ Exploit prediction models                                │
│  │                                                              │
│  ├─ Multi-language support                                      │
│  │  ├─ Vyper contracts                                          │
│  │  ├─ Rust (Solana/NEAR)                                       │
│  │  └─ Move (Aptos/Sui)                                         │
│  │                                                              │
│  └─ Advanced static analysis                                    │
│     ├─ Data flow analysis                                       │
│     ├─ Control flow graphs                                      │
│     └─ Symbolic execution                                       │
│                                                                 │
│   Q2 2026 - Multi-Chain Expansion                               │
│  ├─ L2 Support                                                  │
│  │  ├─ Optimism                                                 │
│  │  ├─ Arbitrum                                                 │
│  │  ├─ Polygon zkEVM                                            │
│  │  └─ Base                                                     │
│  │                                                              │
│  ├─ Alt-L1 Support                                              │
│  │  ├─ Solana                                                   │
│  │  ├─ Avalanche                                                │
│  │  └─ BNB Chain                                                │
│  │                                                              │
│  └─ Cross-chain security analysis                               │
│                                                                 │
│   Q3 2026 - Ecosystem Integration                               │
│  ├─ GitHub Actions integration                                  │
│  ├─ VS Code extension                                           │
│  ├─ Remix IDE plugin                                            │
│  ├─ Foundry integration                                         │
│  └─ CI/CD pipelines (CircleCI, GitLab)                          │
│                                                                 │
│   Q4 2026 - Enterprise Features                                 │
│  ├─ Custom template marketplace                                 │
│  ├─ White-label solutions                                       │
│  ├─ SLA guarantees                                              │
│  ├─ Dedicated support                                           │
│  └─ On-premise deployment                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

