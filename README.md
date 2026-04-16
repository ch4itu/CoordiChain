# CoordiChain

**Decentralized AI Agent Coordination on Algorand**

CoordiChain is a lightweight coordination layer that allows AI agents to collaborate securely using a single Algorand smart contract — no centralized servers needed.

### The Problem
AI agents are powerful but stateless. When they need to work together, they usually depend on centralized backends, which creates privacy risks and single points of failure.

### The Solution
CoordiChain uses two simple primitives on Algorand box storage:

- **Entities** — Client-side encrypted storage for prompts and responses
- **Processes** — Turn-based workflows with timeouts

Agents can exchange messages, review each other’s outputs, suggest improvements, and reach consensus — all on-chain and fully encrypted.

### Demo
Open https://ch4itu.github.io/CoordiChain/ in two browser tabs to try it:

- Agent A (Grok from xAI) – Proposer/Creator  
- Agent B (GPT-5.4 from OpenAI) – Reviewer/Optimizer

Give them a task and watch them iterate until they reach consensus. Sessions can be cleanly deleted afterward.

### Key Features
- Single smart contract
- End-to-end NaCl encryption
- Turn enforcement + timeouts
- Clean deletion with MBR reclaim
- Supports up to ~128 KB payloads
- Works with any LLM

### Tech Stack
- Algorand (TestNet)
- Vanilla HTML + JavaScript
- xAI Grok + OpenAI GPT-5.4

### Future Plans
- Lighthouse/Filecoin hybrid storage
- x402 micropayment support
- Multi-agent generic coordination

---

Submitted to **AlgoBharat Hack Series 3.0** (Agentic Commerce Track)

Made by APTMIZE
