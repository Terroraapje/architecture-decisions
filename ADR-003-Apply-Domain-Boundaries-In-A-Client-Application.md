# Apply domain boundaries in the tooling UI to prevent contract leakage

- **Decision**: Model the tooling UI around explicit domain boundaries, not screens or API endpoints
- **Status**: Accepted
- **Date**: 2025-11-06

## 1. Context
The tooling application is a VueJS-based internal client that integrates with multiple backend APIs to support a wide range of operational use cases.
It is used to manage CRUD-style data, configure games and servers, plan campaigns, manage shops, and inspect player-related information.

The application was initially developed with a strong focus on delivery speed.
File structure and organization followed no explicit architectural model beyond Atomic Design for UI components.
Non-UI code (interfaces, enums, helpers, API clients) was grouped primarily by technical type rather than by responsibility or domain concept.

As the codebase grew quickly, this structure led to several problems:
- Related concepts were scattered across the codebase
- There were no clear boundaries defining what files were allowed to do
- No layering was enforced, causing API data and contracts to drift directly into UI components
  This led to hard coupling between API responses, UI components, and feature behavior (e.g. pagination logic embedded in views)
- Feature behavior became harder to reason about
- New contributors struggled to understand ownership and responsibilities

Although the tooling app currently contains little domain logic beyond validation, it still acts as the primary integration point for multiple backend systems.
As a result, it implicitly encodes assumptions about backend contracts and workflows, even when those assumptions are not made explicit.

After approximately six months of growth, multiple senior engineers expressed concern that the existing structure would not scale.
The lack of clear boundaries made continued development increasingly fragile, despite the absence of complex client-side business rules.

The goal was to introduce structure that reflects **what the tooling app is responsible for today**, while leaving room for safe evolution as its scope continues to expand.


## 2. Constraints / Decision Drivers

The decision was shaped not only by technical concerns, but by clear business and organizational risks.

- **Loss of trust and platform fragmentation**\
  The tooling app was intended to be the central operational platform.
  Without structural intervention, other teams were likely to build workarounds or side tools outside the platform, fragmenting ownership and increasing long-term cost.

- **Risk of project failure and restart**\
  After more than six months of development, no single manager or tool had been fully delivered.
  There was a real risk that the project would be deemed unsuccessful and restarted, incurring significant cost and further delaying business value.

- **Internal operations teams as primary users**\
  The tooling app is used by internal operations teams with low tolerance for instability or inconsistent behavior.
  Reliability and predictability were more important than rapid iteration or experimental approaches.

- **Credibility under pressure**\
  Continued feature delivery was required, but the lack of visible outcomes after several months put the team’s credibility at risk.
  Architectural work had to enable delivery quickly — not delay it.

- **Multiple backend APIs with independent evolution**\
  The tooling app integrates with multiple backend systems that evolve independently.
  Without clear boundaries and layers, backend changes risked propagating directly into UI components, increasing regression risk and coordination cost.

- **Absence of existing boundaries and layers**\
  There were no enforced rules defining responsibilities.
  Files were free to depend on anything, which had already led to tight coupling between API contracts, shared behavior (e.g. pagination), and UI components.

- **Atomic Design addressed UI composition, not application structure**\
  Atomic Design helped organize visual components, but provided no guidance for data access, shared behavior, or separation between API contracts and UI concerns.

- **Need for proportional, incremental change**\
  The application had to continue evolving while architectural improvements were introduced.
  A full rewrite or heavy architectural framework would not have been accepted by the business.

The chosen approach needed to stabilize the codebase, restore confidence, and enable visible delivery — without overcorrecting or slowing the team down.

## 3. Alternatives
Several alternatives were considered before introducing a domain-oriented structure in the tooling application.

#### 1. Continue with the existing structure
We could have continued development using the existing file organization and conventions.

This was rejected because the lack of boundaries and layers was already causing tight coupling between API contracts, shared behavior, and UI components.
Continuing without intervention would have increased the risk of fragmentation, regressions, and loss of confidence in the platform.

#### 2. Reorganize files by technical type only
Another option was to improve consistency by more strictly grouping files by technical type (e.g. interfaces, enums, services), without introducing additional structure.

This approach would have improved discoverability, but not responsibility.
It would not have prevented API data from leaking into UI components, nor clarified where shared behavior (such as pagination) should live.

The core problems were about ownership and boundaries, not naming or folder hygiene.

#### 3. Feature- or screen-based modularization
We considered organizing the codebase strictly around features or screens, with each feature owning its components and API interactions.

This approach works well for smaller or short-lived applications, but in this case it risked duplicating shared behavior and reinforcing coupling between features and backend contracts.
Given the breadth of functionality in the tooling app, this would likely have increased long-term maintenance cost.

#### 4. Full Domain-Driven Design in the frontend
A full DDD-style implementation in the frontend was explicitly considered and rejected.

The tooling app currently contains little client-side business logic beyond validation.
Introducing aggregates, entities, or complex domain layers would have been disproportionate to its responsibilities and would have slowed delivery without clear business benefit.

---

The selected approach needed to sit between these extremes:
more structure than ad-hoc or feature-based organization, but significantly lighter than a full DDD implementation.

## 4. Decision

The tooling application is structured around explicit **domain suites**.
A suite represents a cohesive subject area in the tooling app (e.g. Player, Platform, Monetization) and groups related features, managers, and tools.

Suites are **frontend-owned** and do not mirror backend services.
They exist to manage client-side responsibility and complexity across multiple APIs, without coupling the UI to backend boundaries.

### Enforced structure and responsibilities

Each suite follows strict, enforced rules:

- **UI components**
    - May only interact with **composables**
    - Must not access services, domain models, or API data directly

- **Composables**
    - Orchestrate data and state for the UI
    - Consume domain services
    - Expose **UI-specific DTOs** to components
    - Prevent domain or API concerns from bleeding into the UI layer

- **Domain services**
    - Own domain models and value objects
    - Are the only layer allowed to call API services

- **API services**
    - Are the sole integration point with backend APIs
    - Must map all external data into internal DTOs
    - Prevent API contracts from leaking beyond the transport layer

- **Shared domain**
    - Contains cross-suite concerns and tooling-wide concepts
    - Encapsulate shared behavior (e.g. pagination, filtering)
    - Explicitly separates application-level behavior from suite-specific logic

### Enforcement

This structure is not aspirational.

- Naming conventions, file locations, and allowed dependencies are enforced through linting
- Imports across suites are restricted
- Violations fail builds and code reviews

As a result, components consume **domain state**, not raw API data, and responsibilities remain explicit and auditable.

### Testing strategy (preparatory)

While test coverage is still evolving, the structure explicitly prepares for:
- Domain-level tests validating behavior and orchestration
- Component-level tests focused on rendering and interaction, not data shaping

This separation ensures that future tests can align with architectural boundaries rather than component trees.

### 5. Trade-offs
This decision introduced intentional friction and constraints.

- **Higher upfront cost for new features**\
  Adding functionality now requires following the suite structure and layering rules.
  Simple changes may take longer initially due to required mappings and indirection.

- **Reduced freedom in implementation**\
  Developers can no longer “just wire things together”.
  Hard rules around file placement, naming, and allowed dependencies limit flexibility in exchange for consistency.

- **Increased learning curve**\
  New contributors must understand the suite structure, layers, and responsibilities before being productive.
  This is mitigated by explicit conventions and enforced rules, but remains a real cost.

- **More code and indirection**\
  Introducing DTOs, composables, services, and API services increases the amount of code compared to a flat structure.
  This was accepted to prevent implicit coupling and hidden dependencies.

- **Stricter enforcement over developer autonomy**\
  Trust is placed in the structure and tooling rather than individual judgment.
  Linting and import restrictions may feel heavy-handed, but were necessary given the previous lack of boundaries.

These trade-offs were accepted deliberately to stabilize the codebase, restore confidence, and reduce long-term maintenance and coordination cost.

### 7. Signs for re-evaluation
This decision should be revisited if one or more of the following signals emerge:

- **The tooling app’s responsibilities shrink**\
  If the application becomes a thin CRUD or read-only interface with minimal orchestration, the current level of structure may no longer be justified.

- **Suite boundaries become unclear or contested**\
  If domain suites start to overlap heavily or require frequent cross-suite coordination, the boundaries should be re-evaluated.

- **The cost of enforcement outweighs its benefits**\
  If linting rules, import restrictions, or layering constraints significantly slow delivery without providing stability or clarity, the approach should be adjusted.

These signals are intended to ensure the architecture remains a tool for delivery, not a constraint maintained beyond its usefulness.