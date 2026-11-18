# Reaction Protocols

What to do when a harness tool reports a bad result. Each protocol points to the heuristics that explain the fix:

- `heuristics/clean-code.md`
- `heuristics/clean-architecture.md`
- `heuristics/goos-atdd.md`

Smell codes such as G5 or F3 refer to the Clean Code catalog in `heuristics/clean-code.md`.

Heuristics are context, not dogma. Explain the risk in the code at hand before applying any of them.

## crapper: a function scores CRAP ≥ 10

```text
CRAP = CC² × (1 − coverage)³ + CC
```

At 100% coverage, CRAP equals CC (cyclomatic complexity). So the first question is always: **what is CC?**

### CC ≥ 10: split the function

No amount of testing can bring this function under 10. It must be split.

- Look for G30 (does more than one thing) and G34 (descends more than one level of abstraction). Extract functions whose names reveal intent (G20).
- Encapsulate conditionals and boundary conditions (G28, G33). Replace repeated type switches with polymorphism (G23, the “one switch” rule).
- If the branching is a business decision that lives in a controller, handler or adapter, move it into a use case or entity. See Entities, Use Cases, Boundary Data in `clean-architecture.md`.
- **Case statements:** a flat dispatch `case` / `switch` adds one to CC per branch but is often the clearest form. Before splitting it, ask whether it is the single allowed switch (G23). If you keep it, report the crapper gate as failed with a justified exception, naming the function and the reason. Do not report the gate as passed.

### CC < 10: test it, split it, or both

Coverage can bring the score under 10. For example, CC = 9 needs coverage above roughly 77%.

- Write behavior tests for the uncovered branches (T1, T5) using the outside-in loop in `goos-atdd.md`.
- If a branch is hard to reach from a test, listen to the test: the function probably hides a collaborator or does two things. Splitting may be the better fix.

## dryer: structural similarity ≥ 0.8

ub-mode's threshold is 0.8, while dryer's default is 0.82. Run it with `--threshold 0.8`.

dryer exits 0 whenever it produces a report, so read the report itself, not the exit code.

### Duplication within one module

- G5: extract the shared function or constant, named for intent.
- If the copies differ by a type-based switch, consider G23.

### Duplication across modules

- First ask whether the logic belongs with the data it uses (G14 Feature Envy, G17 Misplaced Responsibility).
- If the duplicates always change together, CCP says they belong in one component. Consolidate them, or extract a shared component that respects ADP and SDP (`clean-architecture.md`).

### Duplication across a boundary

Request and response models, and mappers on either side of an architectural boundary, can look alike on purpose: their independence is the boundary. Before merging them, confirm that the coupling you would add costs less than the duplication. If you keep the duplicates, record the reason.

## mutator: survivors and uncovered sites

Exit codes: 0 means every executed mutant was killed, 2 means the baseline tests fail, and 3 means a mutant survived.

### Surviving mutants

A survivor means a test runs the code but does not notice when its behavior changes.

- Assert the observable outcome the mutant changes: the value returned, the state exposed, or the message sent to a collaborator when that collaboration is the behavior (`goos-atdd.md`).
- Test boundary conditions explicitly (T5). Many survivors are `<` versus `<=`.
- If no observable behavior can tell the mutant apart from the original, the code may be dead or redundant. Delete it (G9, F4).

### Uncovered sites

- Cover them with behavior tests (T1), including the error paths. Error handling is one thing and deserves its own tests.
- Inject failures through owned roles or adapters, never through doubles of third-party types (`goos-atdd.md`).
- If you cannot write a meaningful test for a site, question whether the code should exist.

## uml-viewer: dependency-rule violation (optional)

uml-viewer is an interactive viewer, not a gate. It marks an arrow red only when the project's policy declares `:levels` and a dependency runs from a higher-level (inner) component to a lower-level (outer) one. It does not detect cycles automatically; look for them in the diagram yourself.

When you see a violation or a cycle:

- Invert the dependency with an interface owned by the policy side (DIP).
- Break the cycle by inverting one edge or extracting a shared component (ADP).
- If a stable component depends on a volatile one, extract an abstraction (SDP, SAP).
- If a framework or database dominates names and packages, isolate it behind an adapter.

All of these moves are described in `clean-architecture.md`.
