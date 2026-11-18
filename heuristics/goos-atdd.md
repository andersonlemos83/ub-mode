# GOOS: Test Design and Acceptance Tests

Source: Steve Freeman and Nat Pryce, *Growing Object-Oriented Software, Guided by Tests* (2009).

Use this as thinking pressure for test-driven design, not as a framework guide. Adapt the ideas to the codebase in front of you and keep its testing conventions and libraries. Do not import vocabulary from other testing schools as if it came from GOOS.

## Where GOOS Fits in ub-mode

The TDD rules in `SKILL.md` are the loop for every change. Use this file for what they do not cover, and do not let it grow a small change:

- **Always:** test doubles (Unit Tests and Collaboration), Listening to the Tests, and Readable, Diagnostic Tests.
- **When a change adds behavior a user or an external system can observe:** start with a failing acceptance test (The Loop, Acceptance Tests). A bug fix or an internal change starts with the unit test from `SKILL.md`.
- **Only when you start a new system or cross a new architectural boundary:** build a walking skeleton.
- **Interaction checks only when the collaboration is the behavior.** Otherwise assert the outcome. A test that only checks that something was called lets mutants survive in the values that matter.

## The Loop

GOOS nests two feedback loops:

```text
outer: write a failing acceptance test ──────────────────→ acceptance passes
        └─ inner: failing unit test → make it pass → refactor ──┘ (repeat)
```

1. Name the behavior in domain terms.
2. Pick the smallest meaningful example that proves progress.
3. Write an acceptance test that fails for the expected reason.
4. Let that failure point to the next missing capability.
5. Drive the internals with unit tests about responsibilities and collaborations.
6. Refactor production code and tests once green.
7. Keep the acceptance test as regression coverage.

Distinguish work-in-progress acceptance tests from regression tests, which must always pass.

## Walking Skeleton

Before the first feature, build a walking skeleton: the thinnest end-to-end slice that can be built, deployed, started and tested automatically. It does almost nothing, but it crosses every real architectural boundary that matters. It flushes out build, deployment and environment risks first, and it gives the first acceptance test a place to run.

## Outside-In

Start at the edge where behavior is observable (a user, operator, external system or business process) and work inward. Each object you discover is described by what its neighbors need from it. Interfaces emerge from those needs, so name them after the role they play in the domain, not after their implementation.

## Acceptance Tests

Good acceptance tests:

- Say what the system does, not how its objects do it.
- Use domain vocabulary and externally meaningful inputs and outputs.
- Exercise real entry points, end to end where feasible, including the deployment and environment risks.
- Check every visible effect that defines the behavior.
- Fail with diagnostics that explain the missing behavior.

Weak acceptance tests bypass the real entry point without reason, only assert “something was called”, duplicate unit-level detail, or hide behavior behind broad fixtures, sleeps or vague assertions.

## Unit Tests and Collaboration

GOOS uses mock objects as a design tool. A mock helps you discover the roles an object needs from its neighbors and the messages they exchange.

- **Mock roles, not objects.** This is the title of the authors' earlier paper, which the book cites. Use interfaces to name the roles objects play, and double those roles.
- **Only Mock Types That You Own (Chapter 8).** Never double third-party types. Wrap them in an adapter you own, then double the adapter. Test the adapter itself with focused integration tests against the real dependency.
- **Don't Mock Values (Chapter 20).** Plain values and pure data are created and asserted directly.
- **Unit-Test Behavior, Not Methods (Chapter 5).** Test features of the object, not one test per method.
- **Use interaction checks when the collaboration is the behavior**, such as an object that notifies a listener or sends a command to a neighbor. Do not use them to freeze incidental implementation details.
- **Allow Queries; Expect Commands.** Stub the queries an object needs, and set expectations only on the commands that are part of its contract.
- **Tell, Don't Ask (Chapter 2).** The caller describes what it wants in terms of the role its neighbor plays and lets the neighbor decide how. The book equates this with the Law of Demeter: objects decide based on what they hold or receive, and avoid navigating to other objects. This keeps interactions observable and makes it easy to swap objects that play the same role. GOOS also notes that we still “ask” to get values and collections, or when using factories.

## Listening to the Tests

When a test is hard to write, the production design is usually telling you something. Do not force the test through; fix the design.

| Test friction | What it suggests |
| --- | --- |
| Bloated setup, many constructor arguments | Too many responsibilities or dependencies; missing defaults or builders |
| Many expectations in one test | The object coordinates too much |
| Need to mock a concrete or third-party class | A missing role interface or adapter |
| Need to reach into private state | A hidden collaborator that should be explicit |
| Fragile assertions | Coupling to representation instead of intent |
| Sleeps or polling hacks | The system lacks observable synchronization points |
| Poor failure messages | Diagnostics are not treated as part of the design |
| Slow tests | An unclear line between fast feedback and integrated confidence |

## Readable, Diagnostic Tests

- Name tests for the behavior and the expected outcome.
- One coherent feature per test, which may take a handful of assertions. Split the test if it asserts different features.
- Keep arrange, act and assert visibly separate.
- Use Test Data Builders when raw construction hides intent; spell out only the values that matter.
- Make failure messages explain the broken behavior. Check the diagnostic while the test is still red, before you make it pass.
- If a reader cannot tell why a test exists, it probably asserts the wrong thing or too many things.

## Logging

GOOS separates support logging, which is a feature someone relies on (operations, audit), from diagnostic logging, which is developer debug output.

- Support logging is a requirement. Hide it behind a domain-named role (for example, a notifier or auditor) and test that interaction like any other.
- Diagnostic logging should not drive design or tests.

So whether verifying a log call is right depends on which kind of log it is.

## Asynchrony, Persistence and Other Boundaries

- Acceptance tests prove the integrated behavior that users or external systems see.
- Unit tests keep domain decisions independent of infrastructure mechanics.
- Boundary and integration tests check adapters, serialization, persistence mapping and delivery semantics. Test only where those are the behavior under test.
- **Asynchronous code (Chapter 27):** sample or listen for the expected state, and fail if it does not appear within a timeout. Never use fixed sleeps. Define the timeout in one place, and keep synchronization distinct from assertions.
- **Testing That an Action Has No Effect:** “nothing happened” needs an explicit observation point, for example a follow-up event that proves the system processed the action. Otherwise the test passes vacuously.
- **Clean Up Persistent Data at the Start of a Test, Not at the End** (Chapter 25, under “Isolate Tests That Affect Persistent State”).

## Review Checklist

1. What behavior is this test proving?
2. Who observes that behavior?
3. Is this the smallest useful example?
4. Did the test fail for the right reason before the implementation?
5. Is the acceptance test written in domain language?
6. Do the unit tests clarify responsibilities rather than mirror the implementation?
7. Are test doubles used only for owned roles or boundaries?
8. Are setup, action and assertion readable enough to explain intent?
9. Will the failure diagnostics help the next developer fix the problem?
10. Does the suite balance fast design feedback with integrated confidence?
