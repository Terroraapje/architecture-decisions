# Reject semantic versioning for clients that don’t evolve together

### 1. Context
We were migrating a long-running legacy monolith to a new API platform.\
The system served multiple consumers, each with its own lifecycle, capabilities, and constraints. Some clients evolved quickly, others slowly — and a few could not be updated at all.

The goal of the new platform was clear: enable long-term evolution without repeatedly breaking consumers.
The challenge was less obvious: not all clients consumed the API in the same way, nor supported the same feature set.

This asymmetry turned API versioning into a design problem rather than a mechanical one.

### 2. Constraints
- One API, multiple long-lived consumers
- Consumers with different feature capabilities
- Some clients could not be updated
- The API needed to evolve safely over time
- Runtime capability detection was unreliable
- Backward compatibility could not be assumed globally

These constraints were structural — not temporary.

### 3. Why semantic versioning didn’t fit
Semantic versioning works well when all consumers share a common understanding of the product and its feature set. A “minor” version implies that existing clients can safely upgrade without breaking.

In our case, that assumption simply **wasn’t true.**

A change that was non-breaking for one client could be breaking for another. Using semantic versioning would have communicated safety guarantees that did not actually exist.

Semantic versioning would not reduce risk — it would hide it.

### 4. The naive alternatives
Before settling on a solution, we considered several common approaches:
- Defaulting requests without a version to the “latest” API\
→ leads to hidden breaking changes
- Conditional payloads based on inferred client behavior\
→ increases runtime complexity and failure modes
- Maintaining multiple undocumented behaviors\
→ creates unclear contracts and operational risk

All of these approaches pushed complexity into runtime, where it is hardest to reason about and debug.

### 5. The decisions
**Decision 1 — Explicit default version mapping**\
Some legacy clients could not be updated to send an explicit API version.\
Instead of maintaining implicit “latest version” behavior, we made a deliberate choice:

Requests without a version are explicitly mapped to **v1**.

This preserved existing behavior for unupdatable clients while forcing all migrating clients to explicitly declare which API contract they depend on.
The absence of a version stopped being ambiguous.
---
**Decision 2 — Non-linear, single-digit API versions**\
We deliberately chose **non-linear, single-digit API versions**, treating each version as a strict, isolated contract:
- Each version represents a complete and explicit API contract
- No implicit backward-compatibility promises between versions
- Versions coexist instead of evolving linearly
- Compatibility is validated at design time, not runtime

This approach avoids pretending that all clients evolve together — because they don’t.

### 6. Trade-offs
These decisions introduced real, intentional costs:
- Client teams must actively validate API compatibility
- Documentation becomes a critical part of the contract
- Migrations require explicit coordination
- Upfront effort increases for consumers that can evolve

We accepted this friction intentionally to protect long-term API evolution and avoid hidden failures in production.

### 7. Outcome & learnings
This approach led to:
- Clearer and more honest API contracts
- Fewer implicit assumptions about client capabilities
- Safer parallel evolution of multiple consumers
- Easier introduction of future API versions

Most importantly, it shifted complexity away from runtime behavior and into explicit design decisions.

### 8. Signals for Re-evaluation
This decision would be re-evaluated if:
- All active clients can reliably declare and upgrade API versions
- Client lifecycles converge instead of remaining asymmetric
- Contract proliferation begins to slow overall platform evolution
- Operational overhead of parallel versions outweighs runtime safety

### 9. Closing thought
**Compatibility is not a promise — it’s a decision.**

APIs should communicate truth, even when that truth introduces friction.

---
*This post describes an architectural decision pattern rather than a specific implementation. Details have been generalized to focus on reasoning and trade-offs.*
