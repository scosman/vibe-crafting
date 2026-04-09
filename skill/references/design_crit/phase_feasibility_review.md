# Technical Feasibility Review

## What to Look For

### Technology Fit
- Each component can be built with the stated technology choices — the framework, language, or platform actually supports the described behavior
- Technology selections are appropriate for the problem domain (not using a real-time streaming framework for a batch job, not using a relational database for a graph traversal problem)
- Stated versions of frameworks or libraries actually exist and support the features the spec relies on
- Platform constraints are acknowledged (mobile vs web capabilities, browser API availability, OS-level access)

### Hard Problem Acknowledgment
- Distributed consensus, real-time sync, conflict resolution, offline-first, CRDT — if the spec describes any of these, is the complexity acknowledged and designed, or hand-waved?
- Search and ranking: if the spec says "users can search for X," is there a search infrastructure design, or is it assumed to be a simple database query?
- Real-time features: if the spec describes live updates, collaborative editing, or push notifications, is the infrastructure designed (WebSockets, SSE, polling)?
- Data migration: if the system replaces an existing system, is data migration designed or assumed to be trivial?
- Cross-platform consistency: if the spec targets multiple platforms, are platform-specific limitations acknowledged?

### Performance Assumptions
- Stated scale targets (users, requests/sec, data volume) are realistic with the proposed architecture
- Performance-critical paths are identified and the architecture accounts for them (caching, indexing, denormalization)
- Growth projections are addressed — does the architecture handle 10x the initial scale without fundamental redesign?
- Latency requirements are stated and the architecture can meet them (a design with 5 sequential service calls can't promise sub-100ms response)

### Third-Party Dependency Availability
- External services the spec relies on actually exist and provide the assumed capabilities
- Library assumptions are valid — the spec says "use library X to do Y" and library X actually supports Y
- API rate limits, pricing tiers, and usage restrictions of third-party services are acknowledged
- Vendor lock-in risks are identified if heavy reliance on a specific provider's proprietary features

### Implementation Ordering
- The implementation plan doesn't have circular dependencies (phase 3 requires phase 5 which requires phase 3)
- Foundation components are built before features that depend on them
- Integration points between phases are clear — what exactly does phase N produce that phase N+1 consumes?
- Parallel workstreams don't have hidden dependencies

## Common Issues

- **Assuming libraries handle complexity they don't**: "We'll use Library X for real-time sync" when Library X only handles transport, not conflict resolution. The hard part is still undesigned.
- **Underestimating integration difficulty**: The spec treats integration with an external service as a simple API call when it actually involves auth flows, webhook handling, retry logic, data mapping, and error recovery.
- **Ignoring operational complexity**: The architecture describes what gets built but not how it's deployed, monitored, scaled, backed up, or recovered after failure. Operational concerns are part of feasibility.
- **Underestimating search complexity**: The spec says "users can search across all content" without designing an indexing strategy, relevance ranking, faceted filtering, or autocomplete. Full-text search over a relational database doesn't scale, and each of these features introduces its own infrastructure and tuning requirements.
- **Assuming network reliability**: The design treats all service-to-service calls as synchronous and reliable. No design for timeouts, retries, circuit breakers, or partial failures.
- **Underestimating state management complexity**: The spec describes a stateful UI or workflow without designing how state is persisted, synchronized, recovered after crashes, or handled across multiple instances.

## Severity Guidance

**Critical:**
- A core feature relies on technology that doesn't support the required capability — the architecture fundamentally can't deliver the stated design
- A hard problem (distributed consensus, real-time sync, conflict resolution) is central to the product but treated as a simple implementation detail
- Scale targets are stated but the architecture has obvious bottlenecks that will fail well before those targets (single database for millions of concurrent writes, synchronous processing for real-time requirements)
- Circular dependencies in the implementation plan that make the build order impossible

**Moderate:**
- Third-party dependency assumptions are unverified — the spec relies on a library or service capability that may not exist or may have significant limitations
- Operational concerns (deployment, monitoring, migration) are entirely absent — implementable but will cause pain
- Performance assumptions lack justification — the spec claims sub-second response times without designing for it
- Integration complexity is underestimated — the spec treats a multi-step integration as a single API call

**Mild:**
- Technology choices are workable but not ideal for the problem domain (using a general-purpose tool when a specialized one would be better)
- Growth beyond stated targets isn't considered (the design works at 1x scale but there's no thinking about 10x)
- Minor implementation ordering issues that can be resolved with small adjustments to the plan
