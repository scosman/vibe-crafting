# Architecture Review

## What to Look For

### Component Boundaries
- Each component has a clear, single responsibility — not a "god component" that handles authentication, data processing, and UI rendering
- Component names accurately describe what they do (a component named "Utils" or "Helpers" is a design smell)
- The boundary between components is at a natural seam — components can be understood, tested, and modified independently
- Internal implementation details of one component don't leak into another's interface

### Coupling
- Components communicate through well-defined interfaces, not by reaching into each other's internals
- Changes to one component don't cascade into changes across many others
- Shared state between components is explicit and minimal — not implicit global state
- Temporal coupling is identified (component A must run before component B, but this isn't enforced or documented)

### Data Flow
- The path data takes through the system is traceable from input to output
- Data transformations between components are explicit — what format does data arrive in, what format does it leave in
- Data ownership is clear — one component is the source of truth for each piece of data
- Data doesn't flow in circles (A sends to B, B transforms and sends to C, C sends back to A without clear reason)

### Scalability
- The architecture supports the stated scale targets without fundamental redesign
- Stateless components that can be horizontally scaled are identified
- Stateful components have a scaling strategy (sharding, partitioning, replication)
- Bottlenecks are acknowledged — if everything flows through a single component, that component's scaling is critical

### Failure Modes
- What happens when each component fails is defined — does the system degrade gracefully or fail entirely?
- Cascading failure scenarios are identified — if the database slows down, does every upstream service also slow down and eventually crash?
- Recovery strategies are designed — after a failure, how does the system return to a healthy state?
- Timeout and retry policies are specified for inter-component communication
- Circuit breaker patterns are considered for dependencies that can fail

### Observability
- Logging strategy is defined — what is logged, at what level, and in what format (structured vs. unstructured)
- Key metrics are identified — latency, error rates, throughput, queue depth, or other indicators that reflect system health
- Health check endpoints or heartbeat mechanisms are designed for each component
- Distributed tracing is considered for request flows that cross component boundaries

### Extension Points
- Where the spec anticipates variability (new features, new integrations, new data sources), the architecture provides extension points
- Extension points are at the right granularity — not so fine-grained that they add complexity everywhere, not so coarse that adding a feature requires touching everything
- Plugin or provider patterns are used where the spec describes multiple implementations of the same concept (e.g., multiple auth providers, multiple storage backends)

## Common Issues

- **God components**: One component that does everything — it handles requests, processes data, manages state, talks to external services, and renders output. This makes the system impossible to test, modify, or scale independently.
- **Leaky abstractions**: The architecture defines clean component boundaries on paper, but the actual interfaces expose implementation details. A "storage" component whose interface includes database-specific query syntax isn't abstracting anything.
- **Distributed monolith**: Multiple services that must be deployed together, share a database, and can't function independently. This has all the complexity of microservices with none of the benefits.
- **Missing observability design**: The architecture describes the happy-path data flow but includes nothing about how operators will monitor the system, diagnose problems, or understand what's happening in production. Logging, metrics, tracing, and health checks should be designed, not bolted on.
- **Implicit orchestration**: Multiple components must coordinate to complete a workflow, but there's no explicit orchestrator or coordination mechanism. Each component assumes the others will do their part, with no design for what happens when they don't.
- **Over-engineering for hypothetical scale**: The architecture uses distributed systems patterns (event sourcing, CQRS, microservices) for a system that will serve 100 users. Complexity should be proportional to actual requirements.
- **Single point of failure unmarked**: A critical component has no redundancy, failover, or recovery design, but the spec doesn't acknowledge this as a limitation.

## Severity Guidance

**Critical:**
- Component boundaries are unclear or overlapping — implementers won't know where to put code, leading to duplication or conflicting implementations
- A single component failure cascades to total system failure with no designed recovery path
- Data ownership is ambiguous — multiple components can modify the same data with no conflict resolution
- The architecture has fundamental scaling bottlenecks that contradict the stated scale requirements

**Moderate:**
- Coupling between components is higher than necessary — changes to one component will require coordinated changes to others
- Failure modes are unaddressed for non-critical components (the system works but degrades poorly)
- Observability is not designed — the system will be difficult to operate and debug in production
- Extension points are missing where the spec explicitly mentions future variability

**Mild:**
- Component naming is vague or inconsistent (functional but confusing)
- Data flow is traceable but not documented — readers can figure it out by reading multiple spec sections
- Minor over-engineering that adds complexity without clear benefit but doesn't block implementation
