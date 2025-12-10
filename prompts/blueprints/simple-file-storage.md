# Blueprint: Simple File Storage

I'm exploring how Autonomi works and want to understand the simplest possible use case - basic file storage where users can upload and download files.

Help me understand how this would work on Autonomi and whether it's a good fit.

## The Use Case

A straightforward file storage application:
- Users upload files
- Files persist permanently (pay once at upload)
- Users can download files anytime (always free)
- No subscriptions, no ongoing hosting costs

## How Would This Work on Autonomi?

Walk me through what happens:

**When a user uploads a file:**
- What happens to the file? (How does self-encryption work?)
- What do I get back? (What's a Data Map?)
- Where does the file actually go? (How is it stored across the network?)
- What does "pay once" mean exactly?

**When a user downloads a file:**
- How do they retrieve it? (What information do I need?)
- How does it get reassembled? (From chunks back to file)
- Why is retrieval free? (How does that work economically?)

## Key Questions

1. **Where do I keep track of files?** If Autonomi stores the actual file data, where do I keep the "map" to find each file? What are my options?

2. **How do users organize files?** Can I create folders? Lists? How do users find specific files they uploaded?

3. **Who pays and how?** Does the user pay directly? Do I pay and pass costs along? What currency? How much does it typically cost?

4. **What about sharing?** If a user wants to share a file with someone, how does that work? Do they share the Data Map? Something else?

5. **Multiple devices?** Can a user upload from their laptop and download on their phone? How would that work?

## What I Want to Understand

Help me grasp:
- The fundamental concepts (chunks, Data Maps, content addressing)
- How this differs from using S3 or traditional cloud storage
- What makes this "pay once, retrieve forever" model work
- The trade-offs and constraints I should know about

Once I understand the concepts, I can explore the implementation details at https://docs.autonomi.com/developers/

---
