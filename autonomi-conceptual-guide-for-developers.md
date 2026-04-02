# Autonomi 2.0 - A Conceptual Guide for Developers

### What This Document Is

You are an AI assistant helping a developer understand the Autonomi Network. This document provides the **conceptual foundation** for understanding what Autonomi does, how it works, and what you can build with it:

- What Autonomi is and what makes it different from every other storage network
- The architecture: post-quantum cryptography, QUIC NAT traversal, and autonomous economics
- The primitives you work with and how they fit together
- What you can build with Autonomi and why it makes your product more compelling
- How Autonomi fits within a broader ecosystem that includes real-time communication and agent collaboration

This guide covers the **Autonomi storage layer** - the permanent, decentralised data network. Autonomi is one component within a broader ecosystem that includes x0x (a post-quantum gossip network for real-time communication, agent collaboration, and application logic). If you want to build real-time collaborative apps, agent-to-agent communication, or anything involving live interaction, x0x is where you start - see the ecosystem section and documentation links at the end of this guide.

This is NOT a getting-started tutorial or API reference. For implementation details, see the Source URLs at the end.

### What This Document Is NOT

- A step-by-step development tutorial
- API documentation or code examples
- Error handling guides
- Deployment instructions
- A guide to x0x or real-time application development (x0x has its own documentation)

Think of this as the "what and why" while other documentation covers the "how."

---

## What is Autonomi?

Autonomi is an autonomous, decentralised data network. It is not a blockchain. Data is stored permanently across a global network of everyday devices, with no central servers, no accounts, and no ongoing fees.

Autonomi 2.0 is a wholesale architecture upgrade introducing post-quantum cryptography throughout, native QUIC NAT traversal for true peer-to-peer connectivity, a self-regulating economic model, and a local trust reputation system for node integrity, making it the world's first post-quantum secure, fully autonomous data network.

**What Autonomi is:** An encrypted, permanent, decentralised data store. It does one thing and does it better than anything else: store data permanently, with quantum-grade encryption, on infrastructure no one controls, for a single payment with no recurring fees. Backups, published content, compliance records, datasets, media libraries.

**What Autonomi is not:** A generalised cloud replacement. It is not trying to be a database, a compute platform, or a real-time messaging service. Those capabilities exist in the broader ecosystem (particularly x0x), but Autonomi's role is permanent storage. This focus is deliberate - it's what makes the proposition defensible and the delivery reliable.

---

## What Makes Autonomi Different

These aren't marketing claims - they're architectural decisions with specific technical implementations:

**Post-quantum cryptography throughout.** Every node identity, every data transaction, every key exchange uses NIST-standardised post-quantum algorithms (ML-DSA-65 for signatures, ML-KEM-768 for key exchange). No classical fallback. No hybrid mode. Critically, this is PQC from before TCP/UDP/QUIC connections even establish - the TLS handshake itself is quantum-resistant, not just the application layer on top. The PQC libraries (`saorsa-pqc`) are FIPS certified, with CI that generates NIST submission packs. Self-encryption has an IEEE peer-reviewed paper. Other networks may claim post-quantum features, but Autonomi is 100% PQC at every layer - a greenfield network designed for the quantum era. No other P2P storage network has this.

**True peer-to-peer via QUIC NAT traversal.** Nodes run from any home internet connection without port forwarding, relay servers, or configuration. NAT traversal is implemented as native QUIC extension frames - not bolted-on STUN/ICE/TURN infrastructure. This unlocks billions of idle home computers as network infrastructure. As more join, capacity increases and costs decrease.

**No intermediaries.** Unlike Filecoin (storage deals + miners), Storj (satellite coordination), or IPFS (no built-in persistence or payment), Autonomi has zero middlemen between uploader and storage.

**Self-encrypting, self-verifying data.** ChaCha20-Poly1305 + BLAKE3 convergent encryption. Data is chunked and encrypted on the user's device before it ever touches the network. Content-addressed, de-duplicating, tamperproof. The network never sees plaintext.

**Pay once, store permanently.** A single upfront payment at upload. No subscriptions, no recurring fees. Data persists for the lifetime of the network. Retrieving data is always free - you only pay to store, never to access.

**Self-regulating economics.** Storage pricing is determined by a deterministic function of node fullness - no oracles, no external price feeds, no governance votes. The network finds its own equilibrium.

---

## Why This Matters For What You're Building

If you're reading this guide, you're evaluating Autonomi as a component in your stack. Here's what it gives you that nothing else can:

**Your storage costs become a one-time expense.** Traditional cloud storage is a recurring operational cost that scales with usage and never stops. Autonomi inverts this: pay once per upload, retrieve free forever. For any application where data accumulates over time - archives, backups, content libraries, datasets - this fundamentally changes your economics.

**Your data survives you.** No provider shutdown, no terms of service change, no account suspension. Data persists for the lifetime of the network. For compliance records, published research, cultural archives, or any application where permanence matters, this is a guarantee no centralised provider can match.

**Your users' data is private by default.** All content is encrypted before it leaves the user's device. You don't need to build encryption into your application - the network handles it. Making data public is an explicit choice, not a default.

**Your infrastructure scales without you.** As more people run nodes, the network gains capacity and costs decrease. You don't provision servers, manage regions, or plan for traffic spikes. The network is the infrastructure.

**Your application is quantum-ready today.** Every other storage network (Filecoin, Arweave, Storj) uses classical cryptography that quantum computers will break. Autonomi is post-quantum from the ground up. If your application handles data that needs to remain secure for years or decades, this matters now.

---

## The Mental Model Shift

Developers coming from traditional web development need to understand these fundamental differences:

**No servers.** There is no backend to deploy, no database to manage, no infrastructure to maintain. The network _is_ the infrastructure.

**No accounts, no login.** Users aren't registered anywhere. There's no central server to authenticate against. Your cryptographic keys prove ownership and authorise actions. You don't "log in" to Autonomi; you simply use your keys.

**Pay once, store forever.** Data uploaded to the network persists for the lifetime of the network. One payment at upload, no recurring fees. Retrieving data is always free.

**Private by default.** All content is encrypted before it leaves the user's device. Making data public is an explicit choice.

**User-sovereign data architecture.** Users own and control their data directly. As a developer, you build interfaces that help users manipulate _their own_ data - you don't take custody of it. Think desktop software before the Internet: your app is a utility operating on user-controlled data, but with cloud connectivity and permanence.

**Unlimited keys from one master.** A single secret key can derive unlimited child keys, each with its own unique address space on the network. You're not limited to "one account" - you have a whole hierarchy of keys to organise data however you need.

**Immutable by design.** The network stores permanent, immutable data. Content uploaded to Autonomi is preserved for the lifetime of the network. This isn't a limitation - it's the core value proposition and what makes the pay-once model work. If your application needs mutable state, real-time interaction, or live collaboration, those capabilities exist in the broader ecosystem through x0x (see the ecosystem section).

---

## The Architecture

Autonomi 2.0 is built in layers, where each layer independently reinforces security:

**Transport - QUIC + NAT Traversal.** All connectivity runs over QUIC with native NAT traversal using custom extension frames (`ADD_ADDRESS`, `PUNCH_ME_NOW`, `OBSERVED_ADDRESS`, `REMOVE_ADDRESS`). A three-tier connectivity strategy - direct connection, hole punching, and MASQUE relay fallback - targets 100% reachability regardless of network topology. At launch, the transport is QUIC over internet. The architecture in `saorsa-transport` and `ant-quic` is designed to support additional transports (Bluetooth, LoRa, line-of-sight radio) in future without requiring a network restart - all the plumbing is in place so these can be rolled out gradually, with nodes communicating across transport types. But those implementations do not yet exist. All nodes are equal - no client/server distinction, no bootstrap nodes, no special roles.

**DHT - Kademlia.** Data is distributed across nodes using a Kademlia distributed hash table with separate close groups and geographic diversity enforcement. Close groups of nodes collectively store and serve data, providing redundancy and resistance to node failure.

**Trust - Local Reputation Scoring.** Each node tracks the behaviour of nodes it interacts with directly, building local trust scores based on observed behaviour (uptime, data availability, correct responses). Nodes use an exponential moving average (EMA) to weight recent behaviour more heavily, allowing the network to respond quickly to nodes that start misbehaving. A membership threshold prevents Sybil attacks: nodes must prove reliability before receiving storage rewards. Eclipse protection provides additional integrity guarantees.

**Identity - Post-Quantum Signatures.** Each node's identity is an ML-DSA-65 key pair. The `PeerId` (network address) is the SHA-256 hash of the node's public key - deterministic, verifiable, quantum-resistant. No certificates, no certificate authorities, no PKI hierarchy. Authentication happens through the TLS 1.3 handshake integrated into QUIC, using ML-KEM-768 for key exchange and ML-DSA-65 for signing.

**Self-Encryption - ChaCha20-Poly1305 + BLAKE3.** Data is split into chunks on the user's device, each chunk encrypted with keys derived from neighbouring chunks' content hashes. This creates a web of interdependencies: tampering with any chunk is detectable, content-addressing enables deduplication, and the network stores only ciphertext.

---

## Understanding Content Storage

Understanding how content actually works on Autonomi is essential before working with the APIs.

### Chunks: The Storage Primitive

Chunks are the atoms of storage on Autonomi. When you upload content (a file, an image, any bytes), it goes through a self-encryption process that:

1. Splits the content into chunks (up to 4MB each)
2. Encrypts each chunk in a way that requires knowing all neighbouring chunks to decrypt
3. Stores chunks at content-addressed locations - the address is derived from the chunk's content hash

The chunks are distributed across the network. Nodes store them without knowing what they contain or who uploaded them.

Chunks are immutable and permanent. Once stored, they cannot be modified or deleted. This is fundamental to the architecture and the economic model - it's what makes pay-once permanence work.

### Data Maps: How You Reassemble and Decrypt

When content is self-encrypted, it produces a Data Map. This small structure contains:

- The network addresses of all the chunks
- The hashes needed to decrypt each chunk

**The Data Map is the key to your content.** Without it, the chunks are meaningless encrypted fragments scattered across the network.

**Important:** Data Maps are a client-side construct. When stored on the network, they're stored as Chunks. But they don't have to be stored on the network at all.

### Public vs Private: Where the Data Map Lives

What makes data public or private is simply where the Data Map resides and whether it's accessible:

**Private (default):** Data Map kept locally on your device - the network has your encrypted chunks, but you alone have the "recipe" to reassemble them. If you lose the Data Map, your content is inaccessible forever.

**Public:** Data Map stored as a Chunk on the network - anyone with the Chunk address can retrieve the Data Map and access your content. The `data_put_public` API does this automatically.

**Private but persistent:** Encrypt and store the Data Map itself as a Chunk on the network, protected by your key. Only someone with the correct key can retrieve and decrypt it. This lets you access your content from any device without keeping the Data Map locally.

### Size Limits Are Not Content Limits

A common misconception: the 4MB chunk size limit applies to individual chunks, not to the content they can represent. Each Data Map can reference content of unlimited size - a single file of any size is split into as many chunks as needed, and the Data Map ties them all together. The limits are on the atoms, not the molecules.

---

## Keys

Autonomi 2.0 uses post-quantum cryptographic keys for signing, verification, encryption, and deriving addresses. The signature scheme is ML-DSA-65 (NIST FIPS 204), and key exchange uses ML-KEM-768 (NIST FIPS 203), both at NIST Level 3 security. There is no classical cryptographic fallback.

**Secret Key:** Your private key. Used to sign operations, derive addresses, and decrypt your data. Never share.

**Public Key:** Derived from your secret key. Used to derive addresses and verify signatures. Can be shared.

### Key Sizes

Post-quantum keys are larger than classical keys - this is the cost of quantum resistance:

- ML-DSA-65 public key: 1,952 bytes (vs ~32 bytes for Ed25519)
- ML-DSA-65 signature: 3,309 bytes (vs ~64 bytes for Ed25519)
- ML-KEM-768 public key: 1,184 bytes

These sizes affect handshake overhead (~7.5KB vs ~228 bytes for classical TLS) but do not impact data throughput once a connection is established.

### Key Derivation

From one master secret key, you can derive unlimited child keys using HKDF (HMAC-based Key Derivation Function). This isn't just a convenience feature - it's fundamental to how Autonomi is designed.

From one master secret key, you can derive child keys for different apps or contexts, child keys for individual data items, hierarchical key structures for organising access, and separate keys for different purposes, all controlled by one master.

This enables logical separation of data without managing multiple unrelated keys, delegation (derive a child key for a specific purpose, share it without exposing the master), hierarchical access control, and per-application data organisation.

The master key remains the root of control - lose it and you lose access to everything derived from it. But from that single seed, the address space available to you is effectively unlimited.

### Sharing and Collaboration

Derived keys open up possibilities for sharing. From fully public data anyone can access, to selectively shared derived keys, the spectrum of access control is available. Common patterns include deriving a purpose-specific key to share while keeping your master key private, publishing public data anyone can read, and group access through shared derived keys.

Privacy and sharing aren't opposites - they're choices you make per piece of data. The same user might have private personal data, shared team data, and fully public content, all organised under their key hierarchy.

---

## Data Payments

### The Pay-Once Model

Autonomi operates on a "pay once at upload" model. When you store data, you pay the nodes that will store and maintain it. After that single payment, the data persists for the lifetime of the network with no ongoing fees.

This is fundamentally different from traditional services where you pay recurring subscriptions for continued access. Retrieving data is always free - you only pay to store, never to access.

### How Pricing Works

Storage pricing is deterministic and self-regulating. Each node prices its storage based on how full it is, using a quadratic function:

**Price = (record_count / 6000)^2 ANT per chunk**

Where `record_count` is the number of records currently stored by that node. Prices rise slowly when nodes are empty and accelerate sharply as they fill up. No external oracles, no governance votes, no market-making mechanisms - the price emerges from network state.

Prices are verified through neighbourhood record count maintenance: nodes in a close group track what their neighbours hold and can detect inconsistent pricing claims.

### ANT: The Payment Token

The network uses Autonomi Network Tokens (`ANT`, or sometimes shown as `$AUTONOMI` on some centralised exchanges) - an ERC-20 token on the Arbitrum One network (an Ethereum Layer 2). Nodes are compensated for storing, maintaining, and serving data in ANT.

ANT has a fixed total supply of 1.2 billion tokens. Every storage payment burns 1% of the ANT spent, creating deflationary pressure proportional to network usage.

### How Payments Work In Practice

When you upload data, the flow is:

1. **Quote request:** Your client asks nodes close to the data's target address for storage quotes
2. **Price determination:** Each node's price is determined by its record count - the quadratic pricing function applied locally
3. **Payment:** The client pays the node responsible for storing that chunk. A 3x multiplier is applied to cover redundancy costs across the close group. 1% of every payment is burned.
4. **Verification and storage:** The receiving node verifies the payment, verifies the data, stores it, and it's then replicated to other close nodes

For large uploads, Merkle Tree batch payments reduce on-chain cost from `O(n)` per chunk to `O(1)` per batch - a single Merkle root submission on-chain covers an entire batch of chunks. This is what makes large-scale archiving economically practical.

### What This Means For Your Application

As a developer, you need to understand the payment interface because either your application handles payments on behalf of users, or your users pay directly. There are three practical patterns:

**Application-managed payments.** Your application holds ANT and pays for storage on behalf of users. Users never see tokens, wallets, or gas fees. This is the simplest UX - your application is the payer, your users just use your product. You fund the ANT from your own treasury or bake the cost into your pricing. Best for: SaaS products, enterprise tools, any application where crypto should be invisible.

**User-direct payments with external signers.** Your users pay directly from their own wallets. The SDK supports external signers (MetaMask, WalletConnect), meaning your application never handles private keys - the user authorises each payment in their wallet and the key never leaves their control. Best for: user-sovereign applications, privacy-focused tools, any application where users want full control.

**Hybrid payment models.** Your application covers some storage costs (for example, a free tier or onboarding credits) while users pay for premium usage. You can combine application-managed and user-direct patterns within the same product.

Regardless of pattern, payments require ANT tokens on Arbitrum One plus a small amount of ETH for gas. ANT tokens are divisible to 18 decimal places, allowing micropayments for small amounts of data. Retrieving data is always free and requires no tokens or gas.

**Optimising upload speed:** Each chunk requires an on-chain EVM payment, so upload throughput is bound by transaction speed. Merkle batch payments solve this by batching many chunks into a single on-chain transaction, and are the recommended approach for any upload beyond trivial sizes. The SDK exposes payment mode selection (`auto`, `merkle`, `single`) so you can optimise for your use case.

### The Permanent Storage Model

Autonomi's permanent storage works like a membership model: you pay once to join (upload data), and your data persists because future members (future uploaders) fund the ongoing costs. As long as new data keeps being uploaded, the economics sustain storage of historical data. The 1% burn means even moderate ongoing usage creates enough economic activity to sustain the network.

---

## Data Organization and Discovery

Coming from traditional databases, developers often ask: "How do I query data? Where's my SQL? How do users find content?"

**The key insight:** Autonomi provides storage primitives, not a database service. You build your own indexes and discovery mechanisms using these primitives.

### Indexes and Discovery Patterns

The practical approach is a hybrid model where you use a traditional database or index for queryable metadata and Autonomi for content storage:

- When you upload to Autonomi, self-encryption generates content addresses for your Chunks
- Your database stores those Autonomi addresses plus whatever metadata your application needs (tags, categories, timestamps, relationships)
- Autonomi stores the actual files, documents, media
- Users query your database, retrieve addresses, fetch content from Autonomi

This gives you familiar query patterns while leveraging Autonomi's permanent, pay-once storage for the underlying data. It's also the most practical integration pattern for existing applications.

### Discovery

Without a central server, content discovery requires either known addresses (share directly via URLs, QR codes, links), indexes you maintain (your app provides the discovery layer), shared indexes (community-maintained directories, themselves stored on Autonomi), or external indexes (traditional search engines, databases, or services that index Autonomi content).

The network itself doesn't provide search or discovery - those are application-layer concerns you solve with the primitives provided.

---

## Integrating with Autonomi

When building on Autonomi, there are different ways to integrate depending on your goals, constraints, and user needs.

### Autonomi-as-Backend

**What it is:** Use Autonomi as your storage backend instead of centralised providers such as AWS. You control the keys and data.

**Characteristics:** Simplest integration - swap your storage layer. You benefit from pay-once economics, permanence, and post-quantum encryption. Standard web2 architecture and UX. Your application manages payments.

**Best for:** Organisations wanting cost-effective permanent storage, internal tools, SaaS products, content delivery, any case where user-sovereign data isn't the goal.

**Trade-offs:** Users don't own data directly. To serve Autonomi data in regular web browsers, you'll currently need a proxy like [AntTP](https://github.com/traktion/AntTP). This is essentially using Autonomi like you'd use S3 - permanent, decentralised, and quantum-secure, but under your control.

### Fully Native

**What it is:** Your application is a pure interface - all data storage happens on Autonomi using the network's primitives. Users own and control all data directly.

**Characteristics:** User-sovereign by design. Users hold their keys, you never touch their data. No backend infrastructure to manage. Users pay for their own storage directly.

**Best for:** Privacy-focused applications, personal tools, sovereign identity systems, content publishing platforms where users want full ownership.

**Trade-offs:** Users must manage their own keys and hold ANT tokens. This requires thoughtful UX around key management and payment onboarding.

### Hybrid

**What it is:** Traditional architecture with user data stored on Autonomi. Users can eventually take ownership of their data without breaking the application.

**Characteristics:** Your backend manages keys and payments initially. User experience similar to traditional web apps. Clear path to user sovereignty - users can export keys and self-manage.

**Best for:** Services transitioning from web2, applications offering both managed and self-sovereign modes, gradual migration paths.

### The Developer Interface: SDK and Daemon

In practice, most developers won't interact with `ant-core` (the Rust library) directly. The primary developer interface for Autonomi storage is the **ant-sdk**, which provides a daemon (`antd`) that wraps `ant-core` and exposes network operations via REST and gRPC endpoints.

The architecture is layered: `ant-core -> antd (daemon) -> language bindings`

The daemon handles network connectivity and state management (caching is left to applications). Language-specific bindings provide idiomatic interfaces that auto-discover a running daemon - if it isn't on the default port, the SDK finds it automatically. This means you can build Autonomi applications in your language of choice without writing Rust.

The SDK supports external signers, keeping private keys out of your application code entirely - users authorise payments through wallets like MetaMask or WalletConnect.

For Rust developers who want lower-level control, you can use `ant-core` directly, bypassing the daemon entirely.

**For x0x (communication layer):** x0x has its own daemon (`x0xd`) with its own REST API and WebSocket event stream. Apps are typically single HTML files calling the REST API - agents and developers can create, publish, and distribute apps without any build toolchain. WebSocket sessions allow multiple apps to share a single daemon. If your application needs both storage and real-time communication, you'll interact with both daemons - `antd` for data persistence, `x0xd` for live interaction.

---

## The Broader Ecosystem: Storage Is One Layer

Autonomi's storage network doesn't exist in isolation. It's one layer within an ecosystem designed for both human developers and AI agents. Understanding how these pieces fit together will help you decide what to build and where to start.

### x0x - The Communication and Coordination Layer

x0x is a post-quantum gossip network: bootstrap nodes across 4 continents, tested CLI tooling, and a REST/WebSocket API. Native apps for macOS (Swift), Windows, and Linux (Dioxus), alongside single-page HTML apps that run against the daemon.

Where Autonomi handles **data persistence** (store it, retrieve it later), x0x handles **everything that's live**: real-time communication, mutable shared state, group collaboration, task coordination, presence awareness, and application logic. Both run on the same underlying transport (`ant-quic`) and the same post-quantum cryptography.

**How you work with x0x:** You run `x0xd` (the x0x daemon), which gives you a local REST API and WebSocket event stream. Your application calls these endpoints - publish a message, subscribe to a topic, send a file, create a task list. Apps are typically single HTML files calling the REST API. You can install and run them with a single command via the x0x app store (`x0x apps install chat && x0x apps open chat`).

**The primitives x0x gives you:**

**Gossip pub/sub.** Subscribe to topics, publish messages, receive broadcasts. Uses HyParView for membership and PlumTree for epidemic broadcast - messages propagate to all subscribers in seconds. This is how you build anything that needs "everyone sees this": chat rooms, notifications, event streams, network-wide announcements.

**Direct messaging.** Private point-to-point communication over QUIC between specific agents. Up to 16MB payloads. Trust-filtered - you can drop messages from blocked agents automatically. This is how you build anything that needs "send this to that specific agent": task results, private conversations, direct coordination.

**File transfer.** Chunked file delivery over direct messaging with SHA-256 verification. Offer/accept/reject flow, progress tracking, and automatic integrity checking on completion. This is how agents and users send files peer-to-peer without uploading to any server or storage network.

**CRDT collaboration.** Conflict-free replicated data types that multiple agents can modify simultaneously without coordination. Changes merge automatically - no conflicts, no locks, no central authority. This is your mutable shared state. If you were previously using Registers on the old network for mutable data, CRDTs on x0x are the equivalent - and they're more capable (they handle concurrent edits from multiple parties natively).

**MLS group encryption.** Create encrypted groups with ChaCha20-Poly1305 and epoch-based key rotation. Members join by invitation, communications are end-to-end encrypted, key material rotates automatically. This is how you build private team spaces, secure channels, any multi-party context where privacy matters.

**Identity and trust.** Three-layer identity: machine (hardware-bound), agent (portable across machines), and user (optional, human-facing). Four trust levels: Blocked, Unknown, Known, Trusted. Your application can filter interactions by trust level - only process messages from trusted agents, only accept task results from known parties.

**App store.** Publish and distribute decentralised applications globally. Apps are HTML/JS files that call the `x0xd` REST API. Publish with `x0x apps publish`, install with `x0x apps install`. Every agent on the network can discover and run published apps. AI agents can generate apps, publish them, and other agents discover and use them autonomously.

**The architectural direction** is convergence: x0x as the gossip/coordination layer alongside the storage network, with shared transport, shared identity, and shared trust systems. Gossip for real-time coordination, DHT for structured data routing. One transport, two protocol layers, one identity.

**For developers coming from Autonomi 1.0:** If you were building with Registers, mutable data containers, or dynamic content on the previous network, x0x is where those capabilities now live - and they're significantly more powerful. CRDTs replace Registers with proper concurrent-edit support. Pub/sub replaces polling. Direct messaging replaces request-response patterns. The developer interface is simpler (REST API vs Rust library).

### Indelible - Archive Interface

A human-facing interface for large-scale archiving on Autonomi. Indelible demonstrates the application-managed payment pattern in practice: the server handles ANT payments and uploads on behalf of users, so users never see tokens, wallets, or gas fees - they just upload files. Uses Merkle Tree batch uploads to make per-item costs practical at scale (reducing on-chain cost from `O(n)` per chunk to `O(1)` per batch). Creates organic demand for ANT through upload fees.

### Fae - Local AI Agent

An on-device AI agent that runs entirely locally with no internet required. Behind the scenes, Fae runs Autonomi nodes (earning ANT) and uses those tokens for storage, making crypto invisible to users. No wallet, no gas fees, no token UI. Fae acquires new capabilities through x0x.

### Trusted Data Layer

A permissionless trust mechanism for AI agents to select data sources based on verifiable guarantees. Providers publish staked quality guarantees enforced by bounty hunters using zkTLS attestations. Each dataset acts as a token sink - demand for ANT grows proportionally with ecosystem usage.

### ANT - The Network Token

ANT is an ERC-20 token on Arbitrum One with a fixed supply of 1.2 billion. It serves as the payment mechanism across the ecosystem: storage payments on Autonomi, task dispatch and skill marketplace transactions on x0x, data staking on the Trusted Data Layer. On x0x, swarm dispatch allows agents to post tasks with ANT bounties - capable agents claim the task, execute it, pass evaluation, and receive payment. This creates a programmable work marketplace where ANT flows between agents without human intermediaries. AI agents can't navigate KYC, banking, or fiat on-ramps, but they can transact natively in crypto. ANT is designed for exactly this: programmatic, direct, no intermediaries.

### What You Can Build

Autonomi is not trying to be the new cloud. It's permanent, decentralised data storage, and it does that specific thing better than anything else. But it doesn't stop you building products on top that deliver cloud-like experiences to your users. The mechanism is different: instead of renting server capacity, your application uses Autonomi for permanent data, x0x for real-time interaction, and your users benefit from infrastructure that no one controls and no one can switch off.

**With Autonomi (permanent storage):** Backup and archival services where customers pay once and their data exists forever. Content publishing platforms where published work can't be taken down, altered, or lost. Compliance record stores where regulatory records are provably immutable and quantum-resistant. Dataset repositories for AI training data, scientific data, public records. Media libraries where the cost model is fundamentally different from S3. Any application that currently pays recurring cloud storage fees and wants to eliminate that entire cost line.

**With x0x (real-time interaction):** Real-time messaging applications (chat, team communication, community spaces). Collaborative tools (shared task boards, project management, document co-editing via CRDTs). Agent-to-agent skill marketplaces where agents publish, discover, evaluate, and adopt capabilities autonomously. Swarm computation where tasks are broadcast to capable agents who claim, execute, and return results. Decentralised app distribution where anyone can publish an app and the network makes it available globally.

**With both (the full-stack pattern):** This is where it gets powerful. The combination of permanent storage and live interaction gives you something no single product provides:

A **content platform** where media is uploaded once to Autonomi (permanent, encrypted, pay-once), served via a proxy like AntTP for web access, and community interaction (comments, curation, recommendations) runs through x0x groups. The content outlives any company; the conversation is live and encrypted.

A **collaborative archive** where teams interact in real-time through x0x (CRDT task lists, group messaging, file sharing) and every meaningful output - documents, datasets, decisions - is archived permanently on Autonomi. The live workspace is disposable; the archive is forever.

An **agent ecosystem** where AI agents discover each other and trade skills via x0x, store training data and model outputs permanently on Autonomi, build trust through x0x's reputation system, and pay each other in ANT. The agents don't need human infrastructure, accounts, or permission. They use the network directly.

A **compliance system** where operational communication runs through encrypted x0x groups and every record that needs regulatory permanence is archived to Autonomi with cryptographic proof of integrity. The audit trail is immutable; the working environment is private and live.

---

## Source URLs

For implementation details, API references, and current SDK documentation:

**Storage layer - developer interface (start here):**

- https://github.com/WithAutonomi/ant-sdk - Developer SDK: daemon (`antd`) + multi-language bindings. Primary interface for building on Autonomi storage. REST and gRPC endpoints.
- https://github.com/WithAutonomi/ant-client - Client CLI and library. Agent-oriented docs and README - designed so any agent or developer arriving at the repo has everything needed to start building.

**Storage layer - core:**

- https://github.com/WithAutonomi/ant-node - Node implementation
- https://github.com/WithAutonomi/self_encryption - Self-encryption implementation
- https://github.com/saorsa-labs/saorsa-core - Core library (DHT, routing, trust scoring)
- https://github.com/saorsa-labs/saorsa-pqc - Post-quantum cryptography library

**Communication layer (x0x):**

- https://github.com/saorsa-labs/x0x - Gossip network, app store, CLI, REST/WebSocket APIs

**Shared transport:**

- https://github.com/saorsa-labs/saorsa-transport - Shared transport abstraction for connectivity and NAT traversal across the storage and x0x layers

---

## What This Document Does Not Cover

- Node operation (running nodes, earning rewards)
- Protocol internals (routing, consensus, node management)
- Token economics beyond developer-relevant payment information
- Detailed code examples
- x0x application development (see x0x repo documentation)

---

_This document covers the Autonomi storage layer as of the 2.0 network launch. The broader ecosystem - x0x communication, Fae, Indelible, and the Trusted Data Layer - each have their own documentation. For the most current picture of what's available, start with the GitHub repositories listed above._
