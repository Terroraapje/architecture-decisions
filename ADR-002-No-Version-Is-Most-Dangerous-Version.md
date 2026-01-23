# When “No Version” Is the Most Dangerous Version
## Making Default API Behavior Explicit During a Legacy Migration

### 1. Context
During the migration of a long-running legacy monolith to a new API platform, we had to support a wide range of existing consumers.
Some clients evolved actively and could be migrated over time.
Others were effectively frozen — still operational, business-critical, but impossible to update.

To bridge the old and new worlds, incoming requests from legacy systems were proxied to the new platform, carrying additional metadata via headers.
At first glance, API versioning seemed straightforward. 

In practice, it wasn’t.

### 2. Constraints
- Legacy APIs had no formal versioning system
- Implicit rule in the old system:
*“If no version is provided, use the latest behavior”*
- Some clients could never be updated (e.g. legacy runtime constraints)
- New platform introduced explicit API versioning
- Both systems needed to coexist for an extended period

Backward compatibility was required — but ambiguity was not.

### 3. The hidden risk of implicit defaults
The most tempting option was to preserve existing behavior:
Requests without an explicit version would resolve to the latest API version.

This appeared backward-compatible, but it introduced a subtle and dangerous risk.

Every future API change would silently affect clients that had no way to signal intent.
Breaking changes would not be breaking at deployment time, but weeks or months later — when assumptions diverged.

Implicit defaults don’t remove risk.
They defer it, hide it, and make it harder to reason about.

### 4. Alternatives considered
We considered several approaches:
- Treating “no version” as “latest”\
→ hides breaking changes
- Attempting to infer client capability dynamically\
→ unreliable and brittle
- Maintaining special-case legacy behavior indefinitely\
→ leaks legacy assumptions into the new platform

Each option optimized for short-term convenience at the cost of long-term safety.

### 5. The decision — explicit default version mapping
We made a deliberate and explicit choice:
**Requests without an API version are mapped to v1.**

Not implicitly.\
Not dynamically.\
But explicitly, as a documented rule.\
This meant:
- Legacy, unupdatable clients remained stable
- v1 became a clearly bounded compatibility surface
- All migrating clients were forced to declare intent
- “No version” stopped being ambiguous

The absence of a version no longer meant “whatever is current”.\
**It meant one specific contract.**

### 6. Trade-offs
This decision came with real consequences:
- Migrating clients had to explicitly specify API versions
- Client teams had to become aware of contract boundaries
- Versioning became a conscious design decision rather than a default

We accepted this friction intentionally.\
The goal was not to optimize for ease, but for **predictability and evolvability.**

### 7. Outcome & learnings
Making default behavior explicit had several effects:
- Breaking changes became visible and intentional
- Legacy constraints were contained instead of spreading
- The new platform remained free to evolve
- API behavior became easier to reason about over time

Most importantly, the system stopped relying on assumptions that no one remembered making.

### 8. Closing thought
**The most dangerous API behavior is the one nobody realizes exists.**\
If a system depends on implicit defaults, it will eventually fail in ways that are difficult to explain — and even harder to fix.
Making behavior explicit is not about rigidity.\
It’s about honesty.

>Legacy systems don’t break because they’re old.
They break because nobody knows which assumptions are still true.

---
*This post describes an architectural decision pattern rather than a specific implementation. Details have been generalized to focus on reasoning and trade-offs.*