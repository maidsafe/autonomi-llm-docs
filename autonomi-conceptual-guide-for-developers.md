
# Autonomi: A Conceptual Guide for Developers

### What This Document Is

You are an AI assistant helping a developer understand the Autonomi Network. This document provides **conceptual foundation** for understanding Autonomi:
- What Autonomi is and what makes it different
- The fundamental primitives and how they work
- Mental models for thinking about Autonomi and its decentralised storage
- Common architectural patterns and approaches
- When and why you'd use Autonomi

This is NOT a getting-started development or coding tutorial, nor is it an API reference. For implementation details, see the Source URLs at the end.

### What This Document Is NOT

- A step-by-step development tutorial
- API documentation or code examples
- Error handling guides
- Deployment instructions

Think of this as the "what" while other documentation covers the "how."

---
## What is Autonomi?

Autonomi is an autonomous, decentralised data network. It is not a blockchain. Data is stored permanently across a global network of everyday devices, with no central servers, no accounts, and no ongoing fees.

---

## The Mental Model Shift

Developers coming from traditional web development need to understand these fundamental differences:

**No servers.** There is no backend to deploy, no database to manage, no infrastructure to maintain. The network *is* the infrastructure.

**No accounts, no login.** Users aren't registered anywhere. There's no central server to authenticate against. Your cryptographic keys prove ownership and authorise actions. You don't "log in" to Autonomi; you simply use your keys.

**Pay once, store forever.** Data uploaded to the network persists for the lifetime of the network. One payment at upload, no recurring fees. Retrieving data is always free - you only pay to store, never to access.

**Private by default.** All content is encrypted before it leaves the user's device. Making data public is an explicit choice.

**User-sovereign data architecture.** Users own and control their data directly. As a developer, you build interfaces that help users manipulate *their own* data - you don't take custody of it. Think desktop software before the Internet: your app is a utility operating on user-controlled data, but with cloud connectivity and permanence. Different approaches to integrate with Autonomi exist (fully native, hybrid transitional models, or Autonomi-as-backend), which we'll explore later in this document.

**Unlimited keys from one master.** A single secret key can derive unlimited child keys, each with its own unique address space on the network. This means one user can create unlimited Pointers, Scratchpads, GraphEntries - each at a different derived address, all controlled by one master key. You're not limited to "one account" - you have a whole hierarchy of keys to organise your data however you need.

**Content preservation with selective mutability.** The network is designed to preserve all content for the lifetime of the network. The vast majority of data (Chunks) is immutable and permanent. The network also provides primitives for mutable references and mutable containers, which developers combine to build flexible applications.

---

## Understanding Content Storage

Understanding how content actually works on Autonomi is essential before working with the APIs.

### Chunks: Where Content Lives

Chunks are the atoms of storage on Autonomi. When you upload content (a file, an image, any bytes), it goes through a self-encryption process that:

1. Splits the content into chunks (up to 4MB each)
2. Encrypts each chunk in a way that requires knowing all neighbouring chunks to decrypt
3. Stores chunks at content-addressed locations — the address is derived from the chunk's content hash

The chunks are distributed across the network. Nodes store them without knowing what they contain or who uploaded them.

### Data Maps: How You Reassemble and Decrypt

When content is self-encrypted, it produces a Data Map. This small structure contains:

- The network addresses of all the chunks
- The hashes needed to decrypt each chunk

**The Data Map is the key to your content.** Without it, the chunks are meaningless encrypted fragments scattered across the network.

**Important:** Data Maps are a client-side construct. When stored on the network, they're stored as Chunks. But they don't have to be stored on the network at all.

### Public vs Private: Where the Data Map Lives

What makes data public or private is simply where the Data Map resides and whether it's accessible:

**Private (default):** Data Map kept locally on your device
- Network has your encrypted chunks, but you alone have the "recipe" to reassemble them
- If you lose the Data Map, your content is inaccessible forever

**Public:** Data Map stored as a Chunk on the network
- Anyone with the Chunk address can retrieve the Data Map and access your content
- The `data_put_public` API does this automatically

**Private but persistent:** Data Map stored in encrypted containers on the network
- Use Scratchpads, Vaults, or Private Archives
- Data Map is on the network but encrypted with your key
- You can access your content from any device with your key

---

## Native Data Types

The network stores and understands four fundamental types. These are what the nodes actually know about. Everything else is a client-side abstraction built on top of these primitives.

### Chunk

- Up to 4MB of immutable bytes
- Content-addressed (address = hash of content)
- Self-encrypted
- Anyone can store; retrieval requires knowing the address
- Costs to store (pay per chunk). 

### Pointer

- A mutable reference to any other data on the network
- Key-addressed (address derived from a public key)
- Can reference any native data type, including other pointers
- Only the holder of the corresponding secret key can update it (signed)
- Versioned (network keeps latest version)
- Costs to create, updates are free.

These allow you to create a stable address that can point to different data over time.

### Scratchpad

- Up to 4MB of mutable storage
- Key-addressed (address derived from a public key)
- **Automatically encrypted with the corresponding secret key**
- Only the holder of the secret key can read or update
- Costs to create, updates are free.
- Versioned (network keeps latest version)

These allow you to store private, mutable data on the network - such as Data Maps, application state, or user preferences.

### GraphEntry

- A node in a directed graph structure
- 32 bytes per entry (can contain data and/or links to other entries)
- Key-addressed
- Immutable once created
- Can reference parent and descendant entries
- Costs to create.

These allow you to build linked data structures on the network (used internally by Registers and Vaults).

---

## Client-Side Abstractions

These are important patterns built on the native types. The network, itself, doesn't know about them — they are constructs that only exist client-side.

### Data Maps (revisited)

While we introduced Data Maps earlier as essential to understanding content, it's worth clarifying: Data Maps are a client-side concept. They're the output of the self-encryption process. They can be kept locally, stored as Chunks on the network (thus making data publicly available to anyone), or stored encrypted within Scratchpads, Vaults, or Private Archives (keeping the data private but making it network-accessible). The nodes don't have special knowledge of "Data Map" as a type — to them, Data Maps are just data stored within the native types.

### Register

A versioned history of 32-byte values. Each update creates a new GraphEntry linked to the previous one, with a Pointer tracking the latest.

- 32 bytes per entry (not total — it's a chain)
- Full history preserved permanently and immutably
- Updates cost (creates new GraphEntry), but cheaper than initial creation

These allow you to maintain mutable data with a permanent, immutable version history.

**Important:** Previous entries cannot be edited or deleted. The history is permanent.

### Vault

A personal encrypted storage manager. Uses a GraphEntry that points to up to 1,000 Scratchpads, with expansion via chaining.

- Address derived from your secret key (so you always know where it is)
- Contents encrypted in Scratchpads
- Single key gives access to all your stored data
- Pay once for initial size, pay to expand

These allow you to persist Data Maps, keys, and references on the network without losing them - accessible from any device with your key.

### Archives

Archives are structures that map file paths to data on the network, simulating traditional file and folder hierarchies. They support nested paths and store metadata (creation time, modification time, size) for each file.

The key concept: Archives store *references* to content already uploaded to the network, not the content itself. When you retrieve an Archive, you get addresses you can use to download the actual files.

**Public Archive:** 
- Maps file paths to data addresses
- The archive itself is stored as Chunks on the network
- Anyone with the archive's address can retrieve it and access all the files it references

**Private Archive:** 
- Embeds complete Data Maps within the archive structure
- The archive itself is encrypted
- Only the holder of the `PrivateArchiveDataMap` can access it

These allow you to organise multiple files in familiar directory structures, with control over who can access them.

---

## Keys

Autonomi uses BLS keys for signing, verification, and deriving addresses.

**Secret Key:** Your private key. Used to sign updates, derive addresses, and decrypt your data. Never share.

**Public Key:** Derived from your secret key. Used to derive addresses for key-addressed data. Can be shared.

### How Keys Prove Ownership

Key-addressed data types (Pointer, Scratchpad, GraphEntry) derive their network address from a public key. 

When you want to write or update:

1. You sign the content with your secret key
2. Nodes verify the signature matches the public key that derived the address
3. If valid, the write is accepted

This means:
- **To read:** You just need to know the address (and decryption keys for encrypted data)
- **To write:** You must prove ownership by signing with the correct secret key
- **No central authority:** Nodes validate signatures directly - there's no login server or account database

### Key Derivation

BLS cryptography allows deriving unlimited child keys from a single master key. This isn't just a convenience feature - it's fundamental to how Autonomi is designed.

From one master secret key, you can derive:
- Child keys for different apps or contexts
- Child keys for individual data items
- Hierarchical key structures for organising access
- Separate keys for different purposes, all controlled by one master

Each derived public key produces a unique network address. So one user with one master key can create unlimited Pointers, Scratchpads, and GraphEntries - each at its own address, all under their control.

This enables patterns that wouldn't be possible with a single key:
- Logical separation of data without managing multiple unrelated keys
- Delegation (derive a child key for a specific purpose, share it without exposing the master)
- Hierarchical access control
- Per-application or per-context data organisation

The master key remains the root of control - lose it and you lose access to everything derived from it. But from that single seed, the address space available to you is effectively unlimited.

#### Key Derivation for Sharing and Collaboration

While we've emphasised private, user-controlled data, derived keys open up possibilities for sharing and collaboration. From fully public data anyone can access, to selectively shared derived keys, the spectrum of access control is available to you.

Common patterns include:
- Deriving a purpose-specific key to share, keeping your master key private
- Publishing public archives or data that anyone can read
- Group access through shared derived keys
- Collaborative structures where multiple parties can read (or write, depending on design)

The primitives are flexible - privacy and sharing aren't opposites, they're choices you make per piece of data. The same user might have private personal data, shared team data, and fully public content, all organised under their key hierarchy.

---

## Data Payments

### The Pay-Once Model

Autonomi operates on a "pay once at upload" model. When you store data, you pay the nodes that will store and maintain it. After that single payment, the data persists for the lifetime of the network with no ongoing fees. This is fundamentally different from traditional services where you pay recurring subscriptions for continued access.

Retrieving data is always free - you only pay to store, never to access.

### Network Tokens and Payment Integration

The core of Autonomi is not a blockchain - it's a data network. However, to integrate with the wider financial ecosystem, the network uses an EVM-based payment system in its first implementation.

Nodes are compensated for their work (storing, maintaining, and serving data) in network tokens. These are known as Autonomi Network Tokens (ANT) — or $AUTONOMI on some exchanges. 

ANT is an ERC-20 token on the Arbitrum One network (an Ethereum Layer 2), which provides faster and more cost-effective transactions than Ethereum mainnet.

Future updates to the Autonomi Network may introduce additional payment methods and integrations with other payment networks.

### How Data Payments Work

When you upload data, the payment flow works like this:

1. **Quote request:** Your client asks nodes close to the data's target address for storage quotes
2. **Price determination:** Price is determined by an on-chain smart contract based on network storage capacity, available capacity versus demand, and proximity to network capacity limits
3. **Selection and payment:** The client selects a node, sends the data along with the payment transfer
4. **Verification and storage:** The receiving node verifies the payment, verifies the data, stores it, and it's then replicated to other close nodes

Prices are dynamic - they depend on network capacity and demand at the time of upload. The client pays the price determined by the smart contract based on network conditions at the time of upload.

All of this happens automatically when uploading via the CLI or an app.

### What Incurs Costs

**Storage operations:**
- Storing Chunks (pay per chunk, once)
- Creating Scratchpads
- Creating Pointers
- Creating GraphEntries
- Creating and expanding Vaults
- Register creation and updates (creates new GraphEntry for each update)

**Free operations:**
- Retrieving data
- Updating Scratchpads (after creation)
- Updating Pointers (after creation)
- Updating Vaults (after creation)
### Making Data Payments

To make data payments, you need:
- An Ethereum-compatible wallet (MetaMask, etc.)
- ANT tokens on the Arbitrum One network
- A small amount of ETH on Arbitrum for gas fees

ANT tokens are divisible to 18 decimal places, allowing for micropayments for small amounts of data.

**Important for UX:** Users need to have both ANT (for data payment) and ETH (for gas) in their wallet before they can upload. This is a consideration for application design - you may want to guide users through wallet setup and token acquisition, or explore gasless patterns via paymasters.

Note: Free operations require no payment tokens and incur no gas costs, as they don't involve transactions on the payment network.

---

## Size Limits Are Not Content Limits

A common misconception: the size limits on data types (32 bytes, 4MB) apply to the metadata and references, not to the content they can organise.

- A Scratchpad is 4MB, but it can hold thousands of Data Maps
- Each Data Map can reference content of unlimited size
- A single Scratchpad could index hundreds of terabytes of content

The limits are on the index, not the library.

---

## Data Organization and Discovery

Coming from traditional databases, developers often ask: "How do I query data? Where's my SQL? How do users find content?"

**The key insight:** Autonomi provides storage primitives, not a database service. You build your own indexes and discovery mechanisms using these primitives.

### Building Indexes

To make data discoverable, you create and maintain your own indexes:

**Simple indexes in Scratchpads:**
- Store a list of content addresses with metadata
- Update the Scratchpad as you add/remove items
- Client fetches the index, filters/sorts locally

**Versioned indexes with Registers:**
- Maintain a history of index updates
- Each entry points to a new version of your index
- Enables audit trails and rollback

**Graph-based indexes with GraphEntries:**
- Build linked structures (trees, graphs, chains)
- Navigate relationships between data
- Used internally by Vaults and Registers

### Common Patterns

**Content catalog:** Scratchpads containing an array of objects that is updated when new content is added.

**Hierarchical data:** Use Archives for file-like hierarchies, or build custom graph structures with GraphEntries for more complex relationships.

**Search:** For full-text search or complex queries, maintain client-side indexes or use a traditional database for metadata while storing actual content on Autonomi.

### The Hybrid Approach

Many applications use a traditional database for queryable metadata and Autonomi for content storage:
- When you upload to Autonomi: Self-encryption generates content addresses (for Chunks) or you derive addresses from keys (for Pointers, Scratchpads, etc.)
- Database stores: Those Autonomi addresses plus whatever metadata your application needs (tags, categories, timestamps, relationships between items)
- Autonomi stores: The actual files, documents, media
- Users query the database, retrieve addresses, fetch content from Autonomi

This gives you familiar query patterns while leveraging Autonomi's permanent, pay-once storage for the underlying data.

### Discovery in Decentralised Systems

Without a central server, content discovery requires either:
- **Known addresses:** Share addresses directly (URLs, QR codes, links)
- **Indexes you maintain:** Your app provides the discovery layer
- **Shared indexes:** Community-maintained directories (themselves stored on Autonomi)
- **External indexes:** Traditional search engines, databases, or services that index Autonomi content

The network itself doesn't provide search or discovery - those are application-layer concerns you solve with the primitives provided.

---

## Integrating with Autonomi: Three Approaches

When building on Autonomi, there are different ways to integrate the network into your application depending on your goals, constraints, and user needs.

### Autonomi-as-Backend

**What it is:** Use Autonomi as your storage backend instead of centralised providers such as AWS. You control the keys and data - whether that's just your organisation's data or data you manage on behalf of users.

**Characteristics:**
- You manage the keys and orchestrate the network interactions
- Simplest integration - just swap your storage layer
- You benefit from Autonomi's pay-once model and permanence
- Standard web2/traditional architecture and UX

**Best for:** Organisations wanting cost-effective permanent storage, internal tools, SaaS products, content delivery, any case where user-sovereign data isn't the goal.

**Trade-offs:** Users (if you have them) don't own data directly. To retrieve Autonomi data in regular web browsers, you'll currently need a proxy like [AntTP](https://github.com/traktion/AntTP). This is essentially using Autonomi like you'd use S3 - permanent and decentralised, but under your control.

### Fully Native

**What it is:** Your application is a pure interface - all data storage, logic, and state management happens on Autonomi using the network's primitives. Users own and control all data directly. Your app doesn't store or manage any user data itself.

**Characteristics:**
- User-sovereign by design - users hold their keys, you never touch their data
- No backend infrastructure to manage
- Users pay for their own storage directly
- Apps can be entirely static

**Best for:** Privacy-focused applications, personal tools, sovereign identity systems, content publishing platforms where users want full ownership.

**Trade-offs:** Users must manage their own keys - this requires thoughtful application design and key management patterns that help users maintain sovereignty while keeping the experience accessible. Data portability is maximised, but the developer must invest in UX that makes key management understandable and safe.

### Hybrid Transitional

**What it is:** Traditional web2 architecture (your backend, your database) but with user data stored on Autonomi. Users can eventually take ownership and move data to their own control without breaking the application.

**Characteristics:**
- Your backend manages keys and orchestrates Autonomi interactions initially
- User experience similar to traditional web apps
- Clear path to user sovereignty - users can export keys and self-host
- You can offer managed convenience while preserving eventual ownership

**Best for:** Services transitioning from web2, applications where you want to offer both managed and self-sovereign modes, gradual migration paths.

**Trade-offs:** You still run infrastructure initially. Trust model shifts over time as users take ownership.

### Mixed Patterns

Many applications will combine these approaches:
- User-owned private data + shared public data + optional traditional backend for coordination
- Core data on Autonomi (user-controlled) + metadata/indexes in traditional database for discovery
- Freemium models where basic storage is managed, premium users get self-sovereign options

The primitives are flexible enough to support multiple integration patterns, even within a single application.

---

## Source URLs

For implementation details, API references, and current SDK documentation:

- https://github.com/maidsafe/autonomi - Source code, README, latest changes
- https://github.com/maidsafe/self_encryption - Self-encryption implementation
- https://docs.autonomi.com/developers/ - Guides, concepts, getting started
- https://docs.autonomi.com/developers/core-concepts/data-types - Detailed type documentation
- https://docs.rs/autonomi/latest/autonomi/ - Complete Rust API

**Note:** Documentation may lag behind the `main` branch. For the most current implementation details, refer to the GitHub repository.

---

## What This Document Does Not Cover

- Node operation (running nodes, earning rewards)
- Protocol internals (routing, consensus, node management)
- Token economics beyond developer-relevant payment information
- Detailed code examples 

---

*This document is designed to be consumed by AI assistants helping developers understand Autonomi.
