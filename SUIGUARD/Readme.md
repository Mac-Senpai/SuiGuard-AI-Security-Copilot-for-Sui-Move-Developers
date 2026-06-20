# SuiGuard — Hackathon Submission
## Sui Overflow 2026 | DeepSurge

---

## Project Name
SuiGuard — AI Security Copilot for Sui Move Developers

## One-Line Description
Real-time AI security partner for Sui Move developers — scan contracts, detect vulnerabilities, generate fixes, simulate attacks, and store permanent audit reports on Walrus.

## Live Demo
https://suiguard.vercel.app

## Track Selection
- Primary: **The Agentic Web** — SuiGuard is an autonomous AI agent that coordinates multiple Claude AI calls, makes security decisions, and stores verifiable outputs on Walrus without human involvement
- Secondary: **Special — Walrus** — Every audit report, fixed contract, and attack simulation is stored permanently as a Walrus blob with a shareable retrieval link

---

## Problem Statement

Every Sui Move smart contract needs security auditing before deployment. Professional audits cost $10,000–$50,000 and take weeks. The Cetus DEX hack in May 2025 resulted in a $220 million loss — exploiting vulnerabilities that existed between audit cycles.

Existing tools are either:
- Locked inside a single IDE (ChainIDE AI assistant)
- Enterprise-only and expensive (MoveBit, BitsLabAI)
- Slow and inaccessible to indie developers (manual audit firms)

No free, instant, conversational Move security tool exists that any developer can use right now.

---

## Solution

SuiGuard is an AI Security Copilot that sits beside the Sui Move developer in real time. It provides:

1. **Move Contract Scanner** — Accepts any Move contract via paste or upload
2. **Vulnerability Detection** — Identifies CRITICAL/HIGH/MEDIUM/LOW severity issues
3. **AI Security Reviewer** — Conversational chat for security questions
4. **Automated Fix Suggestions** — Returns a fully corrected contract with inline SUIGUARD FIX comments
5. **Audit Report Generator** — Structured report stored permanently on Walrus
6. **Attack Simulation Engine** — Proves each exploit with step-by-step PoC code

---

## How It Works

```
Developer pastes Move contract → asks security question or requests full audit
                ↓
        n8n webhook receives request
                ↓
        Claude AI analyzes contract (Security Audit)
                ↓
        IF full audit requested:
          Claude generates Fixed Contract
          Claude simulates Attack Scenarios
          All 3 outputs stored on Walrus
          3 permanent blob links returned
        ELSE:
          Fast conversational security answer returned
                ↓
        Frontend displays results across 4 tabs:
          💬 Copilot Chat
          📋 Audit Report
          🔧 Fixed Contract
          💀 Attack Simulation
```

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| AI Engine | n8n (self-hosted, v2.10.4) |
| AI Model | Anthropic Claude API (claude-opus-4-6 / claude-haiku-4-5) |
| Decentralized Storage | Walrus Testnet |
| Frontend | HTML/CSS/JS deployed on Vercel |
| Backend | n8n webhook workflow (18 nodes) |
| Network | Sui Testnet |

---

## Walrus Integration

SuiGuard stores 3 separate Walrus blobs per full audit:

| Blob | Content | Purpose |
|------|---------|---------|
| Audit Report | Full security analysis | Permanent verifiable proof of audit |
| Fixed Contract | Corrected Move code | Shareable secure version |
| Attack Simulation | Exploit PoC scenarios | Proof vulnerabilities are real |

Each blob returns a permanent retrieval link:
```
https://aggregator.walrus-testnet.walrus.space/v1/blobs/<BLOB_ID>
```

Developers can include their Walrus audit link in their project README as transparent proof of security review — creating an on-chain verifiable audit trail.

---

## Sample Walrus Blob IDs (From Live Testing)

These are real blobs stored during development testing:

| Report | Blob ID |
|--------|---------|
| Audit Report | T0TH2N0rst8rByTzEtgWanjdCz_JGQsli-K68XcIIPk |
| Fixed Contract | Od3w5YiGD3IG5V_ska89ZBCefSfSAKsD-Bp-vTB7csI |
| Attack Sim | XrV729dqBZLJkVhdockRyMdJ8UrmnDcM-5mxqDC4Dk8 |

---

## Agentic Web Track Justification

SuiGuard qualifies as an AI agent because:

- It **acts autonomously** — receives input, makes decisions via IF routing, coordinates 3 sequential Claude API calls without human involvement
- It **transacts** — stores data on Walrus via PUT requests triggered automatically
- It **coordinates** — orchestrates Audit Agent → Fix Agent → Attack Simulation Agent in sequence
- It **uses Sui's composability** — Walrus blobs are linked to Sui testnet epochs and storage objects

The n8n workflow IS the agent — an autonomous 18-node decision-making system.

---

## Why SuiGuard Wins

| Criteria | SuiGuard |
|----------|---------|
| Working demo | ✅ Live at suiguard.vercel.app |
| Genuine Sui Stack usage | ✅ Walrus testnet storage with real blob IDs |
| Real-world problem | ✅ $220M lost in Cetus hack — security tooling is critical |
| No hollow UI | ✅ Full 18-node n8n backend, 3 real Claude API calls |
| Differentiator | ✅ Only free, instant, Walrus-native Move security tool |
| Extensible | ✅ Can add DeepBook security module, mainnet support |

---

## Live Demo Instructions

1. Go to **https://suiguard.vercel.app**
2. Enter a project name in the sidebar
3. Paste any Sui Move contract in the code area
4. Click **"Quick security scan"** for a fast result (5-10 seconds)
5. Click **"Full audit + Walrus"** for the complete 6-function audit (30-60 seconds)
6. Check all 4 tabs — Chat, Audit Report, Fixed Contract, Attack Sim
7. Click any Walrus link to view the permanently stored report on-chain

---

## n8n Workflow Structure

```
[Manual Trigger / Webhook]
        ↓
[Build Payload] — parses request
        ↓
[BUILD CODE] — assembles Claude API body
        ↓
[Claude Audit] — security analysis
        ↓
[Parse Audit Response] — detects audit vs chat
        ↓
[IF: Full Audit?]
    TRUE ↓                    FALSE ↓
[Build Fix Body]          [Build Chat Response]
[Fix Generator]                   ↓
[Build Sim Body]          [Respond to Webhook]
[Attack Simulator]
[Walrus Store Audit]
[Walrus Store Fix]
[Walrus Store Sim]
[Build Full Response]
        ↓
[Respond to Webhook]
```

---

## What's Next (Post-Hackathon Roadmap)

- **DeepBook Security Module** — Specialized auditing for contracts integrating DeepBook orderbook
- **Mainnet Support** — Walrus mainnet storage for production audit trails
- **VS Code Extension** — Inline security warnings as Move code is written
- **Sui Wallet Integration** — Sign audit reports with developer's Sui wallet
- **SuiGuard Score** — On-chain security score for Move contracts browsable on Suiscan

---

## Team
Mac Senpai — AI Automation Expert, Builder

---

## Repository
[https://github.com/Mac-Senpai/SuiGuard-AI-Security-Copilot-for-Sui-Move-Developers]

## Demo Video
[Add 2-minute demo video link here]

## Contact
[mail : chineduezeukoma@gmail.com]
