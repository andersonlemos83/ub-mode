# ub-mode

`ub-mode` is an agent skill that enforces disciplined software engineering principles when writing or changing code. It is loosely based on portions of [@poteto](https://github.com/poteto)'s rules (`poteto-mode` / `pstack`), reduced and paired with deterministic quality-gate tools:

1. **First Make It Work** — Laziness Protocol, Foundational Thinking, TDD, Redesign From First Principles, Attack the Premise, Subtract Before You Add, Outcome-Oriented Execution, Build the Lever, and Architecture (Model the Domain, Boundary Discipline, Type System Discipline, Make Operations Idempotent, Separate Before Serializing Shared State).
2. **Then Make It Right** — Fix Root Causes, Sequence Work into Verifiable Units, Test Behavior (Not Implementation), the Harness (Crapper (`CRAP < 10`), Dryer (duplication `< 0.8`), Mutator (cover the uncovered, kill the survivors, delete dead code)), the Heuristics, and Refactoring.
3. **Practice** — Keep a Log, Guard the Context Window, Worktree Cleanup when needed, and small ordered commits when authorized.

## Layout

```text
SKILL.md                         # the skill: three acts, harness, heuristics index
heuristics/
  clean-code.md                  # Clean Code (2008): smell catalog C/E/F/G/N/T, Simple Design, SRP
  clean-architecture.md          # Clean Architecture (2017): Dependency Rule, components, boundaries
  goos-atdd.md                   # GOOS (2009): test doubles, listening to tests, acceptance tests
references/
  harness-install.md            # requirements and commands for the tools, OS-agnostic
  reaction-protocols.md         # what to do when a gate fails, linked to the heuristics
```

The skill is self-contained: the heuristics are embedded, not loaded from other skills.

## Companion Tool Repositories

`ub-mode` relies on deterministic quality-gate tools to enforce complexity, duplication, and test thoroughness:

- [unclebob/crapper](https://github.com/unclebob/crapper) — Computes cyclomatic complexity, test coverage, and CRAP score per function so every function stays under the CRAP threshold (`< 10`).
- [unclebob/dryer](https://github.com/unclebob/dryer) — Detects structural code duplication so duplicate blocks (`>= 0.8` similarity) are eliminated.
- [unclebob/mutator](https://github.com/unclebob/mutator) — Runs mutation testing to expose uncovered code and surviving mutants that tests fail to catch.
- [unclebob/uml-viewer](https://github.com/unclebob/uml-viewer) — Optional. An interactive viewer for components, coverage, CRAP, mutation scores, and dependency directions. A review aid, not a gate.

## Getting Started

### 1. Install the skill

Place the skill files (`SKILL.md`, `heuristics/`, `references/`) in your agent's skills directory under `ub-mode/`, for example `~/.grok/skills/ub-mode/`, `~/.claude/skills/ub-mode/`, or `~/.cursor/skills/ub-mode/`. Check your agent's documentation for the exact location.

For a committed version, export tracked files from `HEAD` so `.git` and editor folders stay behind. This excludes uncommitted and untracked changes; copy the skill files directly if you need those changes:

```sh
mkdir -p ~/.claude/skills/ub-mode
git -C /path/to/ub-mode archive HEAD | tar -x -C ~/.claude/skills/ub-mode
```

Then invoke `/ub-mode`.

### 2. Install the tools

Follow [references/harness-install.md](references/harness-install.md). In short:

- Clone `crapper`, `dryer`, and `mutator` side by side under `~/.ub-mode/tools/`; mutator expects crapper at `../crapper`.
- They need Python 3.11+ and bootstrap their own `.venv` on first run. Verify with `crapper/crapper --help` and so on.
- `uml-viewer` is optional and needs the Clojure CLI and Java 21+.

If a tool is missing, the agent installs it under `~/.ub-mode/tools/`, outside your project. It asks once before installing a system toolchain such as Python. If installation fails, it reports that gate as not run rather than claim it passed.

## Usage Example

> /ub-mode Add a tiered discount to the checkout total.

1. **First Make It Work.** The agent writes a failing test in domain terms, for example “a gold customer gets 10% off orders above 100”. It watches the test fail for the expected reason, then makes the smallest change that passes.
2. **Then Make It Right.** From the project root it runs the gates:

   ```sh
   crapper --threshold 9.99
   dryer --threshold 0.8
   mutator
   ```

   Suppose crapper flags `applyDiscount` with CC 12. The reaction protocol says that at full coverage CRAP equals CC, so tests alone cannot fix it. The agent splits the function, perhaps replacing the tier `if` chain with one dispatch per G23 in `heuristics/clean-code.md`. Suppose mutator then reports a survivor on `>` versus `>=`. The agent adds the boundary test (T5) and reruns until the gates pass.
3. **Practice.** The agent reports its log and, when authorized, commits in small steps. Each message says why, what changed, and how it was proved:

   ```text
   Add tiered checkout discount

   Gold and silver customers get tiered discounts above a minimum order.
   Tier selection lives in one dispatch; boundary at the minimum is tested.
   Proof: crapper max CRAP 4, dryer no candidates >= 0.8, mutator 0 survivors.
   ```

## Sources

- Principles: [@poteto](https://github.com/poteto)'s `poteto-mode` / `pstack` (adapted and reduced).
- Heuristics: Robert C. Martin, *Clean Code* (2008) and *Clean Architecture* (2017); Steve Freeman and Nat Pryce, *Growing Object-Oriented Software, Guided by Tests* (2009).
- Tools: [crapper](https://github.com/unclebob/crapper), [dryer](https://github.com/unclebob/dryer), [mutator](https://github.com/unclebob/mutator), [uml-viewer](https://github.com/unclebob/uml-viewer).
