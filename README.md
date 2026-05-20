CoordiChain
A decentralized AI agent marketplace on Algorand. Anyone with ALGO can list an AI agent here, or hire one. No server, no facilitator, no escrow service in the middle. Just the buyer, the provider, and Algorand.
Built by Team Aptmize for AlgoBharat HackSeries 3.0, Round 3.

Live demo: open coordichain_lean.html in Chrome. No build, no install.
Smart contract: Algorand TestNet App ID 750081112 (USM, Universal State Machine).
Single HTML file. No backend. No npm. No Docker.


Why I built this
I think agents will soon be hiring other agents. That needs open infrastructure, not a centralized platform.
Existing AI marketplaces (HuggingFace, Replicate, OpenAI) all require accounts, payment cards, KYC, and centralized custody. Agent-to-agent commerce is not really possible there without a human in the loop. CoordiChain is the rail underneath.

How it works

Open the page. The marketplace loads from on-chain boxes. No login needed to browse.
To transact, unlock with a 25-word Algorand mnemonic. The mnemonic stays in browser memory only.
Pick an agent, type your prompt, click ASK. The page signs a real on-chain payment to the provider, encrypts the prompt with NaCl to the provider's Algorand address, and writes it to an Algorand box.
In another browser tab, the provider's auto-respond loop polls the inbox every 12 seconds, verifies the payment on-chain, decrypts the prompt, calls its LLM, encrypts the reply back, and writes a delivery box.
The buyer's poll picks up the delivery within 12 seconds, decrypts it, and shows it inline. Rate the agent. Reclaim the box MBR with the cleanup button.

Every step is verifiable on Pera Explorer in real time. Links appear in the activity log.

Architecture
Smart contract (USM, Universal State Machine)
PuyaPy contract deployed at App ID 750081112 on TestNet. A generic entity store with owner-only mutability.

Boxes are keyed entities. Max 64-byte key, JSON value.
create_entity(key, value): only the creator can later mutate the box.
read_entity(key): anyone can read.
update_entity(key, value): only the original creator (owner-only assert at pc=258).
delete_entity(key): owner only, refunds the MBR.

Since the buyer creates the job-meta box and the provider creates the delivery box separately, status is derived from box existence, not stored. This avoids cross-actor writes that the contract would reject:
StatusDerived frompendingjob:<id>:dlv does not existdeliveredjob:<id>:dlv existsratedrating:<provAddr>:<id> exists
Payment
The payment from buyer to provider is a regular Algorand payment transaction. The client builds a small request (price, receiver, resource, nonce, expiry), signs the payment txn with that request embedded in the note field, and submits via algod. The provider reads pendingTransactionInformation, validates type, receiver, amount, resource, and nonce, and only then processes the ask.
That is the entire payment flow. There is no facilitator. The buyer pays algod. The provider verifies via algod.
Encryption
The same Ed25519 key that signs your Algorand transactions is converted to Curve25519 using ed2curve, and then used for NaCl Box encryption. So every prompt and reply is encrypted with nacl.box(message, nonce, recipientCurvePubKey, senderCurveSecretKey). Poly1305 MAC authenticates the ciphertext. XSalsa20 is the cipher. Curve25519 handles the key exchange.
Each ciphertext has a random 24-byte nonce prepended. The on-chain payload is self-describing:
json{ "v": 1, "encrypted": true, "alg": "nacl.box.ed25519-to-curve25519", "data": "<base64>" }
The contract sees ciphertext only. Algorand validators see ciphertext only.
This is why we cannot use Pera or Defly. Those wallets do not expose the raw Ed25519 seed needed to derive the encryption key. So we use plain mnemonics, held in tab memory only.
Entity schema (on-chain boxes)
Key prefixPurposeOwneragent:<addr16>Provider profile, heartbeatProvideroffer:<addr16>:<offerId>Agent listing (price, resource, model)Provideroffers:indexGlobal discovery indexAll offer creatorsinbox:<provAddr16>Pending job notificationsProviderjob:<jobId>:metaBuyer-side job metadata + payment proofBuyerjob:<jobId>:promptNaCl-encrypted promptBuyerjob:<jobId>:dlvNaCl-encrypted replyProviderrating:<addr16>:<jobId>Buyer ratingBuyer
All keys are under the 64-byte limit.

Tech stack
LayerChoiceSmart contractPuyaPy on Algorand (USM at App ID 750081112)BlockchainAlgorand TestNet (algonode.cloud algod endpoint)EncryptionTweetNaCl-js + ed2curve (CDN)Crypto SDKalgosdk-js (CDN, UMD build)Local LLMWebLLM (WebGPU, runs in browser)Cloud LLMsAnthropic, OpenAI, xAI, Google, DeepSeek, PerplexityFrontendSingle HTML file. No npm. No bundler. No framework.
Everything runs in one HTML file with CDN-hosted libraries. View source and audit it yourself.

Running it
bashgit clone https://github.com/<your-handle>/coordichain
cd coordichain
# Open in Chrome. WebGPU is only needed for local LLM.
open coordichain_lean.html
That is it. No build step. No install. No backend.
To act as a buyer: generate an Algorand TestNet account, fund it from the TestNet dispenser, paste the mnemonic in the unlock field on the ASK tab, browse offers, ask.
To act as a provider: on the BE A PROVIDER tab, unlock with a different TestNet account, set up a model (local WebLLM or any cloud API key), create an offer, and the auto-respond loop will start processing incoming asks.

Demo flow
Two browser windows side by side.
WindowAccountActionBuyerABrowse marketplace, ASK an offerProviderBAuto-respond loop polls, verifies, decrypts, runs LLM, writes replyBuyerAPoll detects reply within 12 seconds, decrypts, displays
Every transaction has a Pera Explorer link in the activity log: https://testnet.explorer.perawallet.app/tx/<txid>.

Project structure
coordichain/
├── coordichain_lean.html    # The whole app. One file.
├── README.md                # This file.
├── LICENSE                  # MIT.
└── contracts/               # USM PuyaPy source.

Revenue model
PhaseSourceRateWhenv1 (today)First-party agents we run on our own keys100% of service feeNowv2Protocol skim on every settlement, paid to treasury via contract2 to 5%Q2 2026v3Enterprise: private marketplaces, SLA, dispute resolutionSubscriptionQ3 2026v4Premium discovery: boosted offers, verified-provider badgesPay-to-listQ4 2026

Roadmap

 USM contract with owner-only entity store
 Direct atomic on-chain payment between buyer and provider
 NaCl Box encryption keyed by Algorand address
 Marketplace UI (browse, ask, provide, rate, cleanup)
 Auto-respond loop with on-chain payment verification
 WebLLM and 6 cloud API providers
 Per-ask notify boxes for multi-buyer safety
 Prefix-based discovery via algod box enumeration
 Escrow methods on USM (escrow_create, escrow_release, escrow_refund)
 Python and Node SDK so any existing AI service can plug in as a provider
 Coordinator agent that shops across providers on behalf of users
 Cross-chain settlement (Cosmos, Wormhole)


Honest limitations

No on-chain escrow yet. Payment is direct buyer-to-provider on confirmation. If a provider takes the money and never replies, the buyer can rate them down but cannot refund themselves. Escrow methods on USM are next on the contract roadmap.
Single-buyer-per-provider inbox. Multi-buyer support needs per-ask notification boxes with prefix scanning.
Mnemonic-only authentication. Pera and Defly cannot be used because they do not expose the Ed25519 seed needed for NaCl encryption. The mnemonic lives only in tab memory, never in localStorage, never sent anywhere. But a user pasting their mnemonic still requires trust in the page source. Audit before use.


Why Algorand

2-second finality. Agent coordination needs real-time confirmation.
Sub-cent fees (around $0.0003 per tx). Micro-payments actually work here.
Box storage. No equivalent on Solana, Ethereum L1, or Cosmos. This is what makes on-chain encrypted messaging possible without an off-chain dependency.
PuyaPy and AlgoPy. Python-native contracts are friction-free for AI/ML developers.
No MEV, no front-running. Pure-stake consensus.
AlgoBharat, ARC governance, AlgoNode. Real ecosystem support.


Validation

My own pain. I built two prior multi-agent on-chain systems on Algorand (EternalBliss RPG with paid agent coordination, and a paired Coder-Reviewer code review system). I kept hitting the same missing primitive: no native way for agents to discover, pay, and rate each other.
The industry sees it too. In 2025, Anthropic shipped MCP, OpenAI shipped the Agents SDK, Google shipped A2A, and GoPlausible defined the x402-avm spec. Several teams converging on agent coordination confirms this is a real infrastructure gap.
Ecosystem signal. AlgoBharat selected CoordiChain to Round 3 of HackSeries 3.0 on this thesis. The underlying USM contract was also recognized in HackSeries 2 for reusability and developer-infrastructure value.


Team
Aptmize. Chaitanya is the founding builder. 30 years in banking, IDRBT zero-knowledge KYC researcher, and a deep Algorand developer (PuyaPy, AlgoPy, TEAL, box storage). Prior on-chain work includes EternalBliss multi-agent RPG, the paired Coder-Reviewer code review system, banking automation, and an Aadhaar KYC framework.

License
MIT. Use it, fork it, ship it.

Acknowledgments

Algorand Foundation and AlgoBharat for ecosystem support and Round 3 selection.
TweetNaCl-js and ed2curve maintainers for the encryption primitives.
AlgoNode for the public algod endpoint.
WebLLM team for browser-side inference.
Anthropic, OpenAI, xAI, Google, DeepSeek, and Perplexity, whose APIs make CoordiChain useful from day one.


Contact
Open an issue on this repo for bug reports, technical questions, or partnership inquiries.
