# Clean Architecture Heuristics

Source: Robert C. Martin, *Clean Architecture: A Craftsman's Guide to Software Structure and Design* (2017).

Use these heuristics to judge structure, not to impose a folder diagram. Inspect actual imports, visibility and call paths instead of trusting diagrams. Local conventions win when they are explicit, tested and actually protect policy from detail. Do not assume every system needs every layer.

## Core Idea: Policy vs Detail

Architecture exists to minimize the lifetime cost of the system by keeping options open.

- **Policy:** the business and application rules that give the system its value.
- **Detail:** the mechanisms that deliver, store, display, transport, schedule, configure or wire that policy, such as databases, the web, frameworks, devices, queues and `main`.

Good architecture lets details be chosen late, replaced or isolated. If an early technology choice has become hard to change, a detail has probably moved inward.

Behavior is urgent and structure is important. Developers must defend structure, because stakeholders mostly feel the urgency.

## The Dependency Rule

Source code dependencies point inward, toward higher-level policy. The circles describe dependencies, not a mandatory folder layout:

```text
Frameworks & Drivers  →  Interface Adapters  →  Use Cases  →  Entities
(web, DB, UI, devices)   (controllers, presenters, gateways)
```

- Inner circles know nothing about outer ones: no names, types, data formats or framework lifecycles leak inward.
- Runtime control may flow outward. When policy calls a detail, invert the source dependency through an interface owned by the policy side.
- Data crossing a boundary is a simple, independent structure. Never pass web requests or persistence rows inward.

## Entities, Use Cases, Boundary Data

- **Entities** hold critical business rules and data that would survive any application, UI or database. Warning signs: persistence annotations, web types or framework base classes in entities; invariants living in controllers or repositories; entities that are only passive mirrors of tables.
- **Use cases** hold application-specific rules and orchestrate entities and gateways toward one goal. Warning signs: orchestration in controllers, jobs or consumers; use cases that return UI-formatted output or persistence records; use cases that know the database, broker or framework.
- **Boundary data** uses simple request and response models and maps at the adapter. Keep the mapping boring and explicit when it protects a real boundary.

## SOLID as Architecture Pressure

| Principle | Architecture-level reading | Warning sign | Move |
| --- | --- | --- | --- |
| SRP | A module answers to one actor | One module changes for unrelated teams; recurring merge conflicts | Split by actor or reason to change |
| OCP | Partition by reason to change and protect higher-level policy from lower-level change | Adding a detail edits core policy; switches in policy pick mechanisms | Let details plug into a boundary; move selection outward |
| LSP | Substitutable contracts, including services and providers | Callers check the implementation type or provider name | Tighten the contract or hide quirks behind an adapter |
| ISP | Don't depend on what you don't use, at source, build or deploy level | A large framework is pulled in for one function; rebuilds triggered by unused changes | Narrow the ports and move broad dependencies outward |
| DIP | Policy owns the abstractions and details implement them | The domain imports infrastructure; use cases instantiate clients | Define ports at the policy side and wire them in `main` |

## Component Principles

**Cohesion (what belongs together):**

- **REP:** the granule of reuse is the granule of release.
- **CCP:** gather classes that change for the same reasons at the same times. This is SRP for components.
- **CRP:** don't force users to depend on things they don't use. This is ISP for components.

REP and CCP make components larger, while CRP makes them smaller. Early in a project CCP usually dominates; reusable libraries later lean toward REP and CRP. The balance shifts over time.

**Coupling (which way dependencies point):**

- **ADP:** no cycles in the component dependency graph. Break a cycle with DIP, or extract a new component that both depend on.
- **SDP:** depend in the direction of stability. Stability means many incoming dependencies, not how rarely the code is edited.
- **SAP:** a component should be as abstract as it is stable. Use SAP to flag concrete, stable components that are painful to change; it is not a mandate to abstract everything.

Treat the metrics (I, A, distance from the main sequence) as thinking tools, not precise laws.

## Review Lenses

Architecture should support four things:

- **Development:** can teams work without coordinating on unrelated changes?
- **Deployment:** do deploy units match real release boundaries?
- **Operation:** are retries, scheduling and logging kept as details outside business rules?
- **Maintenance:** can you find the use case that owns a behavior and change it near the concept?

Screaming architecture: the top-level structure should reveal the use cases, not the framework.

## Details

- **Database:** an I/O mechanism, not the center. Warning signs: schema vocabulary dominates the domain, or testing a rule requires a real database.
- **Web:** a delivery mechanism. Warning signs: use cases take HTTP, session or route types, or a job or CLI cannot reuse the use case without editing it.
- **Frameworks:** tools, never the architecture. Do not marry the framework: keep base classes, annotations and lifecycle at the edge.
- **Main:** the ultimate detail. It wires concrete implementations and configuration, then hands control to policy.

## Services and Packaging

- **Services are not automatically boundaries.** Services that deploy in lockstep, share data shapes or split one business rule across a network are still coupled. Apply the Dependency Rule inside each service.
- Packaging styles (from “The Missing Chapter”, by Simon Brown):
  - **Package by layer:** easy to recognize, but weak encapsulation when everything is public.
  - **Package by feature:** use cases are visible, but technical sublayers can still leak.
  - **Ports and adapters:** the boundary is explicit; useful when integration churn dominates.
  - **Package by component:** groups a use case with its hidden internals, and works when visibility can be enforced.
- The deciding question for any style: does the package structure enforce the architecture, or only illustrate one that callers can bypass?

## Tests and the Humble Object

Tests are part of the system, in its outermost circle. They should depend on stable behavior, not volatile details, and a dedicated testing API can decouple them from structure.

The Humble Object pattern splits behavior that is hard to test (UI, framework callbacks, drivers) into a humble part with almost no logic and a testable part such as a presenter, interactor or mapper.

## Smells and Corrective Moves

| Smell | Signals | Moves |
| --- | --- | --- |
| Framework-centered | Packages named after framework mechanisms; simple rules need framework startup | Push framework mechanics outward; restructure around use cases |
| Database-centered | Entities mirror tables; use cases pass queries through | Put invariants in policy objects and map at the boundary |
| Use case bypass | Controllers or jobs call repositories directly; visibility allows it | Expose the use case as the entry point; hide adapters; test through it |
| Boundary data leakage | Request, response or ORM types move inward | Use independent input and output models; map at the adapter |
| Cyclic components | No independent build, test or release | Invert one dependency or extract a shared component |
| False service boundary | Chatty calls for one decision; lockstep changes | Reshape services around cohesive capabilities |
| Over-ceremonial boundary | Single-implementation interfaces protecting nothing; mappers that only copy fields | Collapse the ceremony; use partial boundaries until volatility justifies a full one |

## Review Checklist

1. What business policy needs protection?
2. Which details currently influence it?
3. Which dependencies point the wrong way?
4. Are there package, module or component cycles?
5. Do entities know about application, persistence, web or framework details?
6. Do use cases know about delivery or storage mechanisms?
7. Does the structure reveal the use cases, or mostly the tools?
8. Can callers bypass the intended use case boundary?
9. Are service boundaries aligned with policy boundaries, or only with deployment?
10. Are tests coupled to volatile details instead of stable behavior?
11. Would a new UI, database, delivery path or provider force changes to core policy?
12. Does a proposed abstraction protect a real option, or only add ceremony?
