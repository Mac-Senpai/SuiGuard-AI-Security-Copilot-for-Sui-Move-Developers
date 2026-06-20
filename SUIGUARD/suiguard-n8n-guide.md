# SuiGuard — n8n Workflow Guide
## AI Security Copilot for Sui Move Developers

---

## Overview

SuiGuard is powered entirely by an n8n workflow acting as the AI engine.
No custom backend server. No custom code deployment.
Just n8n + Claude API + Walrus HTTP endpoints.

---

## Requirements

| Tool | Version |
|------|---------|
| n8n | 2.10.4+ (self-hosted or cloud) |
| Claude API Key | Anthropic API key (claude-opus-4-6 or claude-haiku-4-5-20251001) |
| Walrus Testnet | No key required — public HTTP endpoints |

---

## Workflow Structure (18 Nodes)

```
[Manual Trigger] → [Inject Test Data] ──┐
                                        ↓
[Webhook] ──────────────────→ [Build Payload]
                                        ↓
                                 [BUILD CODE]
                                        ↓
                               [Claude Audit]
                                        ↓
                          [Parse Audit Response]
                                        ↓
                              [Is Full Audit?]
                           TRUE ↙         FALSE ↘
                   [Build Fix Body]    [Build Chat Response]
                         ↓                     ↓
                  [Fix Generator]              ↓
                         ↓                     ↓
                  [Build Sim Body]             ↓
                         ↓                     ↓
                 [Attack Simulator]            ↓
                         ↓                     ↓
               [WALRUS STORE AUDIT]            ↓
                         ↓                     ↓
                [WALRUS STORE FIX]             ↓
                         ↓                     ↓
                [WALRUS STORE SIM]             ↓
                         ↓                     ↓
              [Build Full Response] ───────────↓
                                        ↓
                              [Respond to Webhook]
```

---

## Node Configurations

---

### NODE 1A — Manual Trigger
- Type: Manual Trigger
- Purpose: Testing only — bypassed in production

---

### NODE 1B — Inject Test Data
- Type: Code
- Purpose: Provides test data for manual testing
- Change ACTIVE_SCENARIO (1-4) to test different paths

```javascript
const ACTIVE_SCENARIO = 1;

function getTestData(scenario) {

  var VULNERABLE_VAULT = "";
  VULNERABLE_VAULT += "module test::vulnerable_vault {\n";
  VULNERABLE_VAULT += "    use sui::object::{Self, UID};\n";
  VULNERABLE_VAULT += "    use sui::transfer;\n";
  VULNERABLE_VAULT += "    use sui::tx_context::{Self, TxContext};\n";
  VULNERABLE_VAULT += "    use sui::coin::{Self, Coin};\n";
  VULNERABLE_VAULT += "    use sui::sui::SUI;\n";
  VULNERABLE_VAULT += "    use sui::balance::{Self, Balance};\n";
  VULNERABLE_VAULT += "    struct Vault has key {\n";
  VULNERABLE_VAULT += "        id: UID,\n";
  VULNERABLE_VAULT += "        balance: Balance<SUI>,\n";
  VULNERABLE_VAULT += "        owner: address,\n";
  VULNERABLE_VAULT += "        total_deposited: u64,\n";
  VULNERABLE_VAULT += "    }\n";
  VULNERABLE_VAULT += "    struct AdminCap has key { id: UID }\n";
  VULNERABLE_VAULT += "    public fun withdraw(\n";
  VULNERABLE_VAULT += "        vault: &mut Vault,\n";
  VULNERABLE_VAULT += "        amount: u64,\n";
  VULNERABLE_VAULT += "        ctx: &mut TxContext\n";
  VULNERABLE_VAULT += "    ) {\n";
  VULNERABLE_VAULT += "        let coin = coin::take(&mut vault.balance, amount, ctx);\n";
  VULNERABLE_VAULT += "        transfer::public_transfer(coin, tx_context::sender(ctx));\n";
  VULNERABLE_VAULT += "    }\n";
  VULNERABLE_VAULT += "    public fun deposit(\n";
  VULNERABLE_VAULT += "        vault: &mut Vault,\n";
  VULNERABLE_VAULT += "        payment: Coin<SUI>,\n";
  VULNERABLE_VAULT += "        _ctx: &mut TxContext\n";
  VULNERABLE_VAULT += "    ) {\n";
  VULNERABLE_VAULT += "        let amount = coin::value(&payment);\n";
  VULNERABLE_VAULT += "        vault.total_deposited = vault.total_deposited + amount;\n";
  VULNERABLE_VAULT += "        balance::join(&mut vault.balance, coin::into_balance(payment));\n";
  VULNERABLE_VAULT += "    }\n";
  VULNERABLE_VAULT += "    public fun emergency_drain(\n";
  VULNERABLE_VAULT += "        vault: &mut Vault,\n";
  VULNERABLE_VAULT += "        recipient: address,\n";
  VULNERABLE_VAULT += "        ctx: &mut TxContext\n";
  VULNERABLE_VAULT += "    ) {\n";
  VULNERABLE_VAULT += "        let total = balance::value(&vault.balance);\n";
  VULNERABLE_VAULT += "        let coin = coin::take(&mut vault.balance, total, ctx);\n";
  VULNERABLE_VAULT += "        transfer::public_transfer(coin, recipient);\n";
  VULNERABLE_VAULT += "    }\n";
  VULNERABLE_VAULT += "}";

  var LENDING_CONTRACT = "";
  LENDING_CONTRACT += "module test::lending_pool {\n";
  LENDING_CONTRACT += "    use sui::object::{Self, UID};\n";
  LENDING_CONTRACT += "    use sui::coin::{Self, Coin};\n";
  LENDING_CONTRACT += "    use sui::sui::SUI;\n";
  LENDING_CONTRACT += "    use sui::balance::{Self, Balance};\n";
  LENDING_CONTRACT += "    use sui::transfer;\n";
  LENDING_CONTRACT += "    use sui::tx_context::{Self, TxContext};\n";
  LENDING_CONTRACT += "    struct LendingPool has key {\n";
  LENDING_CONTRACT += "        id: UID,\n";
  LENDING_CONTRACT += "        reserves: Balance<SUI>,\n";
  LENDING_CONTRACT += "        total_borrowed: u64,\n";
  LENDING_CONTRACT += "    }\n";
  LENDING_CONTRACT += "    public fun borrow(\n";
  LENDING_CONTRACT += "        pool: &mut LendingPool,\n";
  LENDING_CONTRACT += "        amount: u64,\n";
  LENDING_CONTRACT += "        ctx: &mut TxContext\n";
  LENDING_CONTRACT += "    ) {\n";
  LENDING_CONTRACT += "        pool.total_borrowed = pool.total_borrowed + amount;\n";
  LENDING_CONTRACT += "        let coin = coin::take(&mut pool.reserves, amount, ctx);\n";
  LENDING_CONTRACT += "        transfer::public_transfer(coin, tx_context::sender(ctx));\n";
  LENDING_CONTRACT += "    }\n";
  LENDING_CONTRACT += "    public fun repay(\n";
  LENDING_CONTRACT += "        pool: &mut LendingPool,\n";
  LENDING_CONTRACT += "        payment: Coin<SUI>,\n";
  LENDING_CONTRACT += "        _ctx: &mut TxContext\n";
  LENDING_CONTRACT += "    ) {\n";
  LENDING_CONTRACT += "        pool.total_borrowed = pool.total_borrowed - coin::value(&payment);\n";
  LENDING_CONTRACT += "        balance::join(&mut pool.reserves, coin::into_balance(payment));\n";
  LENDING_CONTRACT += "    }\n";
  LENDING_CONTRACT += "}";

  var CLEAN_CONTRACT = "";
  CLEAN_CONTRACT += "module test::safe_counter {\n";
  CLEAN_CONTRACT += "    use sui::object::{Self, UID};\n";
  CLEAN_CONTRACT += "    use sui::transfer;\n";
  CLEAN_CONTRACT += "    use sui::tx_context::{Self, TxContext};\n";
  CLEAN_CONTRACT += "    use sui::event;\n";
  CLEAN_CONTRACT += "    struct Counter has key {\n";
  CLEAN_CONTRACT += "        id: UID,\n";
  CLEAN_CONTRACT += "        owner: address,\n";
  CLEAN_CONTRACT += "        count: u64,\n";
  CLEAN_CONTRACT += "    }\n";
  CLEAN_CONTRACT += "    struct Incremented has copy, drop {\n";
  CLEAN_CONTRACT += "        counter_id: address,\n";
  CLEAN_CONTRACT += "        new_value: u64,\n";
  CLEAN_CONTRACT += "    }\n";
  CLEAN_CONTRACT += "    public fun create(ctx: &mut TxContext) {\n";
  CLEAN_CONTRACT += "        let c = Counter {\n";
  CLEAN_CONTRACT += "            id: object::new(ctx),\n";
  CLEAN_CONTRACT += "            owner: tx_context::sender(ctx),\n";
  CLEAN_CONTRACT += "            count: 0,\n";
  CLEAN_CONTRACT += "        };\n";
  CLEAN_CONTRACT += "        transfer::transfer(c, tx_context::sender(ctx));\n";
  CLEAN_CONTRACT += "    }\n";
  CLEAN_CONTRACT += "    public fun increment(\n";
  CLEAN_CONTRACT += "        counter: &mut Counter,\n";
  CLEAN_CONTRACT += "        ctx: &mut TxContext\n";
  CLEAN_CONTRACT += "    ) {\n";
  CLEAN_CONTRACT += "        assert!(tx_context::sender(ctx) == counter.owner, 0);\n";
  CLEAN_CONTRACT += "        counter.count = counter.count + 1;\n";
  CLEAN_CONTRACT += "        event::emit(Incremented {\n";
  CLEAN_CONTRACT += "            counter_id: object::uid_to_address(&counter.id),\n";
  CLEAN_CONTRACT += "            new_value: counter.count,\n";
  CLEAN_CONTRACT += "        });\n";
  CLEAN_CONTRACT += "    }\n";
  CLEAN_CONTRACT += "}";

  var result;

  if (scenario === 1) {
    result = {
      message: "Generate a full audit report and store it on Walrus",
      project_name: "VulnerableVault_Test",
      session_id: "MANUAL-AUDIT-001",
      history: [],
      contract_code: VULNERABLE_VAULT
    };
  } else if (scenario === 2) {
    var h1 = {};
    h1.role = "user";
    h1.content = "I am building a DeFi vault on Sui";
    var h2 = {};
    h2.role = "assistant";
    h2.content = "Great, what functions are you adding?";
    var hist = [];
    hist.push(h1);
    hist.push(h2);
    result = {
      message: "What is the safest way to implement access control?",
      project_name: "ChatTest",
      session_id: "MANUAL-CHAT-001",
      history: hist,
      contract_code: ""
    };
  } else if (scenario === 3) {
    result = {
      message: "Check specifically for flash loan attack vectors",
      project_name: "LendingPool_Test",
      session_id: "MANUAL-FLASH-001",
      history: [],
      contract_code: LENDING_CONTRACT
    };
  } else if (scenario === 4) {
    result = {
      message: "Suggest gas optimizations for this contract",
      project_name: "SafeCounter_Test",
      session_id: "MANUAL-GAS-001",
      history: [],
      contract_code: CLEAN_CONTRACT
    };
  } else {
    throw new Error("Invalid scenario. Choose 1, 2, 3 or 4.");
  }

  return result;
}

return getTestData(ACTIVE_SCENARIO);
```

---

### NODE 1C — Webhook
- Type: Webhook
- HTTP Method: POST
- Path: sui-copilot
- Response Mode: Using Respond to Webhook Node
- Connect output to Build Payload node

---

### NODE 2 — Build Payload
- Type: Code
- Name: Build Payload
- Handles both webhook (body wrapper) and manual trigger (flat) input

```javascript
function buildPayload() {

  var inputData = $input.first().json;
  var body = inputData.body ? inputData.body : inputData;

  var message = "";
  var contractCode = "";
  var sessionId = "";
  var projectName = "";
  var history = [];

  if (body.message) { message = body.message; }
  if (body.contract_code) { contractCode = body.contract_code; }
  if (body.session_id) { sessionId = body.session_id; }
  if (body.project_name) { projectName = body.project_name; }
  if (body.history) { history = body.history; }
  if (sessionId === "") { sessionId = "SESS-" + Date.now(); }
  if (projectName === "") { projectName = "Unnamed Project"; }

  var isAuditRequest = false;
  var lowerMessage = message.toLowerCase();
  if (lowerMessage.indexOf("audit") !== -1) { isAuditRequest = true; }
  if (lowerMessage.indexOf("full report") !== -1) { isAuditRequest = true; }
  if (lowerMessage.indexOf("generate report") !== -1) { isAuditRequest = true; }
  if (lowerMessage.indexOf("check everything") !== -1) { isAuditRequest = true; }

  var result = {};
  result.originalMessage = message;
  result.contractCode    = contractCode;
  result.sessionId       = sessionId;
  result.projectName     = projectName;
  result.isAuditRequest  = isAuditRequest;
  result.history         = history;
  result.timestamp       = new Date().toISOString();

  return result;
}

return buildPayload();
```

---

### NODE 3 — BUILD CODE
- Type: Code
- Name: BUILD CODE
- Builds Claude API request body as plain string

```javascript
function buildClaudeBody() {

  var inputData = $input.first().json;
  var body = inputData.body ? inputData.body : inputData;

  var originalMessage = "";
  var contractCode = "";
  var sessionId = "";
  var projectName = "";
  var isAuditRequest = false;
  var timestamp = "";

  if (body.message) { originalMessage = body.message; }
  if (body.contract_code) { contractCode = body.contract_code; }
  if (body.session_id) { sessionId = body.session_id; }
  if (body.project_name) { projectName = body.project_name; }
  if (body.session_id) { timestamp = new Date().toISOString(); }

  var lowerMessage = originalMessage.toLowerCase();
  if (lowerMessage.indexOf("audit") !== -1) { isAuditRequest = true; }
  if (lowerMessage.indexOf("full report") !== -1) { isAuditRequest = true; }
  if (lowerMessage.indexOf("generate report") !== -1) { isAuditRequest = true; }
  if (lowerMessage.indexOf("check everything") !== -1) { isAuditRequest = true; }

  var userContent = "";
  if (contractCode !== "") {
    userContent += "CONTRACT CODE:\n";
    userContent += contractCode;
    userContent += "\n\nREQUEST: ";
    userContent += originalMessage;
  } else {
    userContent = originalMessage;
  }

  if (userContent === "") {
    userContent = "Please analyze this Sui Move contract for security issues.";
  }

  var escapedContent = userContent.split("\\").join("\\\\");
  escapedContent = escapedContent.split('"').join('\\"');
  escapedContent = escapedContent.split("\n").join("\\n");
  escapedContent = escapedContent.split("\r").join("\\r");
  escapedContent = escapedContent.split("\t").join("\\t");

  var messagesStr = '[{"role":"user","content":"' + escapedContent + '"}]';

  var sys = "";
  sys += "You are SuiGuard, an expert AI Security Copilot for Sui Move smart contracts. ";
  sys += "You are direct, technical, and precise. ";
  sys += "IMPORTANT: Only use the ---AUDIT REPORT START--- format when the user explicitly asks for a full audit, audit report, or says audit. ";
  sys += "For ALL OTHER questions including security checks, vulnerability questions, gas questions, and code reviews: ";
  sys += "respond in plain conversational text with no audit delimiters. ";
  sys += "For QUICK QUESTIONS: Lead with SAFE or UNSAFE or CAUTION. Give a code fix. ";
  sys += "For FULL AUDIT REQUESTS use this format exactly: ";
  sys += "---AUDIT REPORT START--- ";
  sys += "## SUIGUARD SECURITY AUDIT ";
  sys += "**Risk Level:** CRITICAL or HIGH or MEDIUM or LOW or CLEAN ";
  sys += "**Summary:** 2-3 sentences. ";
  sys += "## VULNERABILITY FINDINGS ";
  sys += "For each bug: **[SEVERITY] - [Title]** ";
  sys += "Location: [function] ";
  sys += "Description: [what the bug is] ";
  sys += "Impact: [what attacker achieves] ";
  sys += "Fix: [exact code change] ";
  sys += "## MOVE-SPECIFIC SECURITY CHECKS ";
  sys += "Capability patterns: PASS or FAIL. ";
  sys += "Object ownership: PASS or FAIL. ";
  sys += "Resource leaks: PASS or FAIL. ";
  sys += "Access control: PASS or FAIL. ";
  sys += "Arithmetic safety: PASS or FAIL. ";
  sys += "Events emitted: PASS or FAIL. ";
  sys += "Flash loan resistance: PASS or FAIL. ";
  sys += "## GAS OPTIMIZATION ";
  sys += "List inefficiencies or state No major issues found. ";
  sys += "## FINAL VERDICT ";
  sys += "Decision: APPROVE FOR DEPLOYMENT or APPROVE WITH FIXES or DO NOT DEPLOY. ";
  sys += "List 3 required actions before mainnet. ";
  sys += "---AUDIT REPORT END---";

  var escapedSys = sys.split("\\").join("\\\\");
  escapedSys = escapedSys.split('"').join('\\"');
  escapedSys = escapedSys.split("\n").join("\\n");
  escapedSys = escapedSys.split("\r").join("\\r");
  escapedSys = escapedSys.split("\t").join("\\t");

  var claudeBody = "";
  claudeBody += "{";
  claudeBody += '"model":"claude-haiku-4-5-20251001"';
  claudeBody += ',"max_tokens":4000';
  claudeBody += ',"system":"' + escapedSys + '"';
  claudeBody += ',"messages":' + messagesStr;
  claudeBody += "}";

  var result = {};
  result.claudeBody     = claudeBody;
  result.messagesStr    = messagesStr;
  result.messageCount   = 1;
  result.debug_original = originalMessage;
  result.debug_contract = contractCode.substring(0, 80);
  result.debug_session  = sessionId;
  result.sessionId      = sessionId;
  result.projectName    = projectName;
  result.contractCode   = contractCode;
  result.isAuditRequest = isAuditRequest;
  result.timestamp      = timestamp;

  return result;
}

return buildClaudeBody();
```

---

### NODE 4 — Claude Audit (HTTP Request)
- Type: HTTP Request
- Name: CLAUDE AUDIT
- Method: POST
- URL: https://api.anthropic.com/v1/messages
- Authentication: Generic Credential Type → Header Auth
  - Header Name: x-api-key
  - Header Value: YOUR_CLAUDE_API_KEY
- Additional Headers:
  - anthropic-version: 2023-06-01
  - Content-Type: application/json
- Body Content Type: Raw
- Content Type: application/json
- Body: ={{ $node["BUILD CODE"].json.claudeBody }}
- Timeout: 120000

---

### NODE 5 — Parse Audit Response
- Type: Code
- Name: PARSE AUDIT RESPONSE

```javascript
var claudeData = $input.first().json;
var buildData = $node["BUILD CODE"].json;

var replyText = "";
if (claudeData.content && claudeData.content[0] && claudeData.content[0].text) {
  replyText = claudeData.content[0].text;
}

var claudeReturnedAudit = false;
if (replyText.indexOf("---AUDIT REPORT START---") !== -1) {
  claudeReturnedAudit = true;
}

var isAuditReport = false;
if (buildData.isAuditRequest === true && claudeReturnedAudit === true) {
  isAuditReport = true;
}

var vulnerabilitiesText = "";
if (isAuditReport) {
  var startMarker = "## VULNERABILITY FINDINGS";
  var endMarker = "## MOVE-SPECIFIC";
  var startIdx = replyText.indexOf(startMarker);
  var endIdx = replyText.indexOf(endMarker);
  if (startIdx !== -1 && endIdx !== -1) {
    vulnerabilitiesText = replyText.substring(startIdx + startMarker.length, endIdx).trim();
  } else {
    vulnerabilitiesText = replyText;
  }
}

var result = {};
result.reply               = replyText;
result.isAuditReport       = isAuditReport;
result.claudeReturnedAudit = claudeReturnedAudit;
result.isAuditRequest      = buildData.isAuditRequest;
result.vulnerabilitiesText = vulnerabilitiesText;
result.sessionId           = buildData.sessionId;
result.projectName         = buildData.projectName;
result.contractCode        = buildData.contractCode;
result.timestamp           = buildData.timestamp;

return result;
```

---

### NODE 6 — Is Full Audit? (IF Node)
- Type: IF
- Name: IS FULL AUDIT?
- Condition:
  - Value 1: ={{ $json.isAuditReport }}
  - Operator: Equal
  - Value 2: true
  - Type: Boolean

---

### NODE 7 — Build Fix Body
- Type: Code
- Name: BUILD FIX BODY

```javascript
var auditData = $node["PARSE AUDIT RESPONSE"].json;

var contractCode = "";
var vulnerabilitiesText = "";
var sessionId = "";
var projectName = "";
var timestamp = "";

if (auditData.contractCode) { contractCode = auditData.contractCode; }
if (auditData.vulnerabilitiesText) { vulnerabilitiesText = auditData.vulnerabilitiesText; }
if (auditData.sessionId) { sessionId = auditData.sessionId; }
if (auditData.projectName) { projectName = auditData.projectName; }
if (auditData.timestamp) { timestamp = auditData.timestamp; }

var sys = "";
sys += "You are SuiGuard AutoFix - an expert Sui Move developer who rewrites vulnerable contracts. ";
sys += "Return the COMPLETE fixed contract with ALL vulnerabilities patched. ";
sys += "Add inline comments on every fix using: SUIGUARD FIX: [reason]. ";
sys += "Return ONLY this structure: ";
sys += "---FIXED CONTRACT START--- ";
sys += "COMPLETE FIXED MOVE CONTRACT CODE HERE ";
sys += "---FIXED CONTRACT END--- ";
sys += "Then add: ## FIXES APPLIED with a numbered list of every change made and why.";

var userContent = "";
userContent += "ORIGINAL CONTRACT:\n";
userContent += contractCode;
userContent += "\n\nVULNERABILITIES FOUND:\n";
userContent += vulnerabilitiesText;
userContent += "\n\nGenerate the fully fixed contract now.";

var escapedSys = sys.split("\\").join("\\\\");
escapedSys = escapedSys.split('"').join('\\"');
escapedSys = escapedSys.split("\n").join("\\n");
escapedSys = escapedSys.split("\r").join("\\r");
escapedSys = escapedSys.split("\t").join("\\t");

var escapedContent = userContent.split("\\").join("\\\\");
escapedContent = escapedContent.split('"').join('\\"');
escapedContent = escapedContent.split("\n").join("\\n");
escapedContent = escapedContent.split("\r").join("\\r");
escapedContent = escapedContent.split("\t").join("\\t");

var messagesStr = '[{"role":"user","content":"' + escapedContent + '"}]';

var claudeBody = "";
claudeBody += "{";
claudeBody += '"model":"claude-haiku-4-5-20251001"';
claudeBody += ',"max_tokens":4000';
claudeBody += ',"system":"' + escapedSys + '"';
claudeBody += ',"messages":' + messagesStr;
claudeBody += "}";

var result = {};
result.claudeBody          = claudeBody;
result.sessionId           = sessionId;
result.projectName         = projectName;
result.contractCode        = contractCode;
result.vulnerabilitiesText = vulnerabilitiesText;
result.timestamp           = timestamp;

return result;
```

---

### NODE 8 — Fix Generator (HTTP Request)
- Type: HTTP Request
- Name: FIX GENERATOR
- Same settings as Claude Audit
- Body: ={{ $node["BUILD FIX BODY"].json.claudeBody }}
- Timeout: 120000

---

### NODE 9 — Build Sim Body
- Type: Code
- Name: BUILD SIM BODY

```javascript
var auditData = $node["PARSE AUDIT RESPONSE"].json;

var contractCode = "";
var vulnerabilitiesText = "";
var sessionId = "";
var projectName = "";
var timestamp = "";

if (auditData.contractCode) { contractCode = auditData.contractCode; }
if (auditData.vulnerabilitiesText) { vulnerabilitiesText = auditData.vulnerabilitiesText; }
if (auditData.sessionId) { sessionId = auditData.sessionId; }
if (auditData.projectName) { projectName = auditData.projectName; }
if (auditData.timestamp) { timestamp = auditData.timestamp; }

var sys = "";
sys += "You are SuiGuard AttackSim - an elite Sui Move exploit researcher. ";
sys += "Simulate real attacks against vulnerable contracts to prove exploits are possible. ";
sys += "For each vulnerability produce a simulation block: ";
sys += "---SIMULATION START--- ";
sys += "## ATTACK SIMULATION REPORT ";
sys += "Target Contract: [module name] ";
sys += "### EXPLOIT #[N]: [Vulnerability Name] ";
sys += "Severity: CRITICAL or HIGH or MEDIUM or LOW ";
sys += "Attack Type: [type] ";
sys += "Setup: Attacker wallet and target details ";
sys += "Attack Sequence: Step by step ";
sys += "Proof of Concept: Move pseudo-code ";
sys += "Result: EXPLOIT SUCCESSFUL or CONDITIONAL or MITIGATED ";
sys += "Funds at risk: [amount] ";
sys += "Difficulty: LOW or MEDIUM or HIGH ";
sys += "Mitigation: fix summary ";
sys += "## SIMULATION SUMMARY ";
sys += "Total exploits: [N] ";
sys += "Recommended action: IMMEDIATE FIX REQUIRED or PATCH BEFORE MAINNET ";
sys += "---SIMULATION END---";

var userContent = "";
userContent += "CONTRACT:\n";
userContent += contractCode;
userContent += "\n\nVULNERABILITIES:\n";
userContent += vulnerabilitiesText;
userContent += "\n\nSimulate attacks now.";

var escapedSys = sys.split("\\").join("\\\\");
escapedSys = escapedSys.split('"').join('\\"');
escapedSys = escapedSys.split("\n").join("\\n");
escapedSys = escapedSys.split("\r").join("\\r");
escapedSys = escapedSys.split("\t").join("\\t");

var escapedContent = userContent.split("\\").join("\\\\");
escapedContent = escapedContent.split('"').join('\\"');
escapedContent = escapedContent.split("\n").join("\\n");
escapedContent = escapedContent.split("\r").join("\\r");
escapedContent = escapedContent.split("\t").join("\\t");

var messagesStr = '[{"role":"user","content":"' + escapedContent + '"}]';

var claudeBody = "";
claudeBody += "{";
claudeBody += '"model":"claude-haiku-4-5-20251001"';
claudeBody += ',"max_tokens":3000';
claudeBody += ',"system":"' + escapedSys + '"';
claudeBody += ',"messages":' + messagesStr;
claudeBody += "}";

var result = {};
result.claudeBody          = claudeBody;
result.sessionId           = sessionId;
result.projectName         = projectName;
result.contractCode        = contractCode;
result.vulnerabilitiesText = vulnerabilitiesText;
result.timestamp           = timestamp;

return result;
```

---

### NODE 10 — Attack Simulator (HTTP Request)
- Type: HTTP Request
- Name: ATTACK SIMULATOR
- Same settings as Claude Audit
- Body: ={{ $node["BUILD SIM BODY"].json.claudeBody }}
- Timeout: 120000

---

### NODE 11 — Walrus Store Audit (HTTP Request)
- Type: HTTP Request
- Name: WALRUS STORE AUDIT
- Method: PUT
- URL: https://publisher.walrus-testnet.walrus.space/v1/blobs?epochs=5
- Body Content Type: Raw
- Content Type: text/plain
- Body: ={{ $node["PARSE AUDIT RESPONSE"].json.reply }}

---

### NODE 12 — Walrus Store Fix (HTTP Request)
- Type: HTTP Request
- Name: WALRUS STORE FIX
- Method: PUT
- URL: https://publisher.walrus-testnet.walrus.space/v1/blobs?epochs=5
- Body Content Type: Raw
- Content Type: text/plain
- Body: ={{ $node["FIX GENERATOR"].json.content[0].text }}

---

### NODE 13 — Walrus Store Sim (HTTP Request)
- Type: HTTP Request
- Name: WALRUS STORE SIM
- Method: PUT
- URL: https://publisher.walrus-testnet.walrus.space/v1/blobs?epochs=5
- Body Content Type: Raw
- Content Type: text/plain
- Body: ={{ $node["ATTACK SIMULATOR"].json.content[0].text }}

---

### NODE 14 — Build Full Response
- Type: Code
- Name: BUILD FULL RESPONSE

```javascript
var auditData = $node["PARSE AUDIT RESPONSE"].json;

var fixText = "";
var simText = "";

try {
  var fixNode = $node["FIX GENERATOR"].json;
  if (fixNode.content && fixNode.content[0] && fixNode.content[0].text) {
    fixText = fixNode.content[0].text;
  }
} catch(e) {
  fixText = "Fix generation unavailable";
}

try {
  var simNode = $node["ATTACK SIMULATOR"].json;
  if (simNode.content && simNode.content[0] && simNode.content[0].text) {
    simText = simNode.content[0].text;
  }
} catch(e) {
  simText = "Simulation unavailable";
}

var walrusAuditId = null;
var walrusFixId = null;
var walrusSimId = null;

try {
  var waRaw = $node["WALRUS STORE AUDIT"].json;
  var wa = Array.isArray(waRaw) ? waRaw[0] : waRaw;
  if (wa.newlyCreated && wa.newlyCreated.blobObject) {
    walrusAuditId = wa.newlyCreated.blobObject.blobId;
  } else if (wa.alreadyCertified) {
    walrusAuditId = wa.alreadyCertified.blobId;
  }
} catch(e) {}

try {
  var wfRaw = $node["WALRUS STORE FIX"].json;
  var wf = Array.isArray(wfRaw) ? wfRaw[0] : wfRaw;
  if (wf.newlyCreated && wf.newlyCreated.blobObject) {
    walrusFixId = wf.newlyCreated.blobObject.blobId;
  } else if (wf.alreadyCertified) {
    walrusFixId = wf.alreadyCertified.blobId;
  }
} catch(e) {}

try {
  var wsRaw = $node["WALRUS STORE SIM"].json;
  var ws = Array.isArray(wsRaw) ? wsRaw[0] : wsRaw;
  if (ws.newlyCreated && ws.newlyCreated.blobObject) {
    walrusSimId = ws.newlyCreated.blobObject.blobId;
  } else if (ws.alreadyCertified) {
    walrusSimId = ws.alreadyCertified.blobId;
  }
} catch(e) {}

var baseUrl = "https://aggregator.walrus-testnet.walrus.space/v1/blobs/";

var walrus = {};
walrus.audit_blob_id = walrusAuditId;
walrus.audit_link    = walrusAuditId ? baseUrl + walrusAuditId : null;
walrus.fix_blob_id   = walrusFixId;
walrus.fix_link      = walrusFixId ? baseUrl + walrusFixId : null;
walrus.sim_blob_id   = walrusSimId;
walrus.sim_link      = walrusSimId ? baseUrl + walrusSimId : null;

var result = {};
result.success           = true;
result.is_audit          = true;
result.session_id        = auditData.sessionId;
result.project           = auditData.projectName;
result.timestamp         = auditData.timestamp;
result.audit_report      = auditData.reply;
result.fixed_contract    = fixText;
result.attack_simulation = simText;
result.walrus            = walrus;

return result;
```

---

### NODE 15 — Build Chat Response
- Type: Code
- Name: BUILD CHAT RESPONSE

```javascript
var auditData = $node["PARSE AUDIT RESPONSE"].json;

var result = {};
result.success    = true;
result.is_audit   = false;
result.reply      = auditData.reply;
result.session_id = auditData.sessionId;
result.timestamp  = auditData.timestamp;
result.walrus     = null;

return result;
```

---

### NODE 16 — Respond to Webhook
- Type: Respond to Webhook
- Name: RESPOND TO WEBHOOK
- Respond With: First Incoming Item
- Response Code: 200
- Response Headers:
  - Access-Control-Allow-Origin: *
  - Access-Control-Allow-Methods: POST, OPTIONS
  - Access-Control-Allow-Headers: Content-Type
- Connect BOTH Build Full Response AND Build Chat Response to this node

---

## Walrus Endpoints

| Action | URL |
|--------|-----|
| Store blob | https://publisher.walrus-testnet.walrus.space/v1/blobs?epochs=5 |
| Read blob | https://aggregator.walrus-testnet.walrus.space/v1/blobs/BLOB_ID |

---

## Testing Scenarios

| Scenario | ACTIVE_SCENARIO | Branch | Walrus |
|----------|----------------|--------|--------|
| Full Audit | 1 | TRUE — all 18 nodes | 3 blobs stored |
| Quick Chat | 2 | FALSE — 6 nodes | No storage |
| Flash Loan Check | 3 | FALSE — 6 nodes | No storage |
| Gas Optimization | 4 | FALSE — 6 nodes | No storage |

---

## n8n Sandbox Compatibility Notes

All code uses these patterns for n8n 2.10.4 compatibility:

- `var` instead of `const` or `let`
- String concatenation with `+=` instead of template literals
- `.split().join()` instead of `.replace()` for string escaping
- Manual JSON string building instead of `JSON.stringify()`
- `$node["NAME"].json` instead of `$json` for explicit node references
- No function wrappers on top-level code in final nodes
- Array-aware Walrus response parsing with `Array.isArray()` check

---

## Environment Variables (Coolify/Docker)

```
EXECUTIONS_TIMEOUT=300
EXECUTIONS_TIMEOUT_MAX=300
N8N_CORS_ALLOWED_ORIGINS=*
```

---

## Frontend Deployment

1. Add production webhook URL to `index.html` line 499
2. Create `vercel.json` with rewrite rule pointing to n8n webhook
3. Deploy both files to Vercel as a folder named `suiguard`
4. Frontend available at `https://suiguard.vercel.app`
