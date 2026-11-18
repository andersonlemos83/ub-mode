# Clean Code Heuristics

Source: Robert C. Martin, *Clean Code: A Handbook of Agile Software Craftsmanship* (2008).

Use these heuristics as evidence, not verdicts. Map every smell to concrete code, say which future change it makes harder, and propose the smallest fix. Local repository conventions win unless they create a concrete maintainability risk. Do not carry forward dated tool, IDE, or language-version advice from the book; keep the principle behind it.

## Attribution

The codes below (C, E, F, G, N, T, and the Java-specific J) are the Chapter 17 catalog. Only G14 Feature Envy is credited to Martin Fowler's *Refactoring*, so only it is marked (Fowler). A smell that exists only in *Refactoring* is Fowler's: name it as his, and prefer the closest code here:

| Fowler smell (not in Clean Code's catalog) | Closest Clean Code heuristic |
| --- | --- |
| Long Parameter List | F1 Too Many Arguments |
| Divergent Change | SRP (Chapter 10) |
| Shotgun Surgery | G5 Duplication, G22 Make Logical Dependencies Physical |
| Data Clumps | F1 Too Many Arguments (argument objects, Chapter 3) |
| Primitive Obsession | G26 Be Precise; J3 Constants versus Enums, reframed |

## Simple Design (Chapter 12, Kent Beck's rules)

In priority order. A design is simple when it:

1. Runs all the tests.
2. Contains no duplication.
3. Expresses the intent of the programmer.
4. Minimizes the number of classes and methods.

The last rule is deliberately last. Do not reject a small class or helper because it adds a file if it removes duplication, clarifies intent, or isolates a responsibility. Do not accept an elegant design that cannot be verified.

## Core Rules

- Code is read far more than it is written. Optimize for the next reader.
- Boy Scout Rule: leave touched code cleaner than you found it.
- Functions: small, do one thing, one level of abstraction, read top-down (the Stepdown Rule), few arguments, no flag or output arguments, no hidden side effects, separate commands from queries.
- Names reveal intent, use one word per concept, and describe side effects.
- Comments do not make up for bad code. Good comments explain intent, warn of consequences, or clarify constraints the code cannot express.
- Objects hide data and expose behavior; data structures expose data and have no meaningful behavior. Avoid hybrids.
- Classes have one reason to change (SRP) and high cohesion.
- Separate constructing a system from using it (Chapter 11).

## Single Responsibility Principle

One reason to change, not “one method” or “one tiny class”. Signals:

- One class changes for business rules and for formatting or persistence.
- A function has sections you could name as separate responsibilities.
- Test setup needs many unrelated collaborators.
- One change request touches unrelated parts of the same class.

Before recommending a split, name each reason to change and the new responsibilities. Preserve cohesion: do not split so far that behavior becomes harder to follow.

## G14 Feature Envy (Fowler)

A method uses another object's accessors and data more than its own to make decisions. Move the behavior toward the data owner, or add a method on the envied object that expresses the intent.

Nuance: sometimes the envy is necessary. A report formatter that reads an entity's fields should not move into the entity if that would couple the domain to presentation and violate SRP. Explain why the behavior belongs elsewhere before recommending the move.

## G36 Avoid Transitive Navigation (Train Wrecks)

A module should know only its immediate collaborators (the Law of Demeter). Chains such as `a.getB().getC().doSomething()` are “train wrecks” (Chapter 6). They make the caller learn the object graph, so structural changes ripple through callers and tests.

Fix: “If ctxt is an object, we should be telling it to do something; we should not be asking it about its internals” (Chapter 6, “Hiding Structure”). Find out why the caller navigates, then add a method on the immediate collaborator that does that job, as in the book's `ctxt.createScratchFileStream(classFileName)`. Chapter 6 also notes that this navigation is “sometimes called Feature Envy from [Refactoring]”, which ties G36 to G14. Clean Code describes this idea without naming it; GOOS names it “Tell, Don't Ask” and equates it with the Law of Demeter (see `goos-atdd.md`).

Nuance: chains over plain data structures, which expose data by design, are not Demeter violations. Chains across objects that hide behavior are.

## Smells and Heuristics Catalog (Chapter 17)

### Comments

- C1 Inappropriate Information: history, authors and metadata belong in source control or the issue tracker.
- C2 Obsolete Comment.
- C3 Redundant Comment.
- C4 Poorly Written Comment.
- C5 Commented-Out Code: delete it.

### Environment

- E1 Build Requires More Than One Step.
- E2 Tests Require More Than One Step.

### Functions

- F1 Too Many Arguments.
- F2 Output Arguments.
- F3 Flag Arguments.
- F4 Dead Function.

### General

- G1 Multiple Languages in One Source File.
- G2 Obvious Behavior Is Unimplemented.
- G3 Incorrect Behavior at the Boundaries.
- G4 Overridden Safeties: disabled tests, warnings or checks.
- G5 Duplication: every duplicate is a missed abstraction. This includes switch/if chains repeated across modules, which point to G23.
- G6 Code at Wrong Level of Abstraction.
- G7 Base Classes Depending on Their Derivatives.
- G8 Too Much Information: wide interfaces and exposed internals.
- G9 Dead Code.
- G10 Vertical Separation: declare things close to where they are used.
- G11 Inconsistency.
- G12 Clutter.
- G13 Artificial Coupling.
- G14 Feature Envy (Fowler).
- G15 Selector Arguments.
- G16 Obscured Intent.
- G17 Misplaced Responsibility.
- G18 Inappropriate Static.
- G19 Use Explanatory Variables.
- G20 Function Names Should Say What They Do.
- G21 Understand the Algorithm.
- G22 Make Logical Dependencies Physical.
- G23 Prefer Polymorphism to If/Else or Switch/Case: the “one switch” rule. Allow one switch per type of selection, and use it to create polymorphic objects. See also “Switch Statements” in Chapter 3.
- G24 Follow Standard Conventions.
- G25 Replace Magic Numbers with Named Constants.
- G26 Be Precise.
- G27 Structure over Convention.
- G28 Encapsulate Conditionals.
- G29 Avoid Negative Conditionals.
- G30 Functions Should Do One Thing.
- G31 Hidden Temporal Couplings.
- G32 Don't Be Arbitrary.
- G33 Encapsulate Boundary Conditions.
- G34 Functions Should Descend Only One Level of Abstraction.
- G35 Keep Configurable Data at High Levels.
- G36 Avoid Transitive Navigation.

### Java (J1–J3), reframed

The book's Java-specific heuristics are J1 Avoid Long Import Lists by Using Wildcards, J2 Don't Inherit Constants, and J3 Constants versus Enums. Keep only the durable principle: keep dependency lists readable in the local tooling's idiom, do not use inheritance or scope tricks to reach constants, and prefer type-safe domain representations over raw tokens.

### Names

- N1 Choose Descriptive Names.
- N2 Choose Names at the Appropriate Level of Abstraction.
- N3 Use Standard Nomenclature Where Possible.
- N4 Unambiguous Names.
- N5 Use Long Names for Long Scopes.
- N6 Avoid Encodings.
- N7 Names Should Describe Side-Effects.

### Tests

- T1 Insufficient Tests.
- T2 Use a Coverage Tool!
- T3 Don't Skip Trivial Tests.
- T4 An Ignored Test Is a Question about an Ambiguity.
- T5 Test Boundary Conditions.
- T6 Exhaustively Test Near Bugs.
- T7 Patterns of Failure Are Revealing.
- T8 Test Coverage Patterns Can Be Revealing.
- T9 Tests Should Be Fast.

## Clean Tests (Chapter 9)

- Tests are first-class code. Dirty tests rot production code because they raise the cost of change.
- Readability first: Build-Operate-Check. Grow a domain-specific testing language when setup hides intent.
- Single concept per test. Minimize the number of asserts; do not force exactly one.
- F.I.R.S.T.: Fast, Independent, Repeatable, Self-Validating, Timely.

## Error Handling, Boundaries, Concurrency

- **Error handling (Chapter 7):** error handling is one thing. Use exceptions rather than return codes, give exceptions context, and define exception classes in terms of the caller's needs. Don't return null and don't pass null.
- **Boundaries (Chapter 8):** wrap third-party APIs so they don't leak through the code. Write learning tests to pin their behavior, and define the interface you wish you had for code that does not exist yet.
- **Concurrency (Chapter 13):** keep concurrency code separate from other code. Limit the scope of shared data, prefer independent threads, keep synchronized sections small, and get non-threaded code working first. Treat spurious failures as real threading issues.

## Refactoring Sequence

This is a review ordering distilled from the case studies in Chapters 14–16 (Successive Refinement, JUnit Internals, Refactoring SerialDate), not a list from the book:

1. Pin behavior with tests.
2. Remove dead code and clutter.
3. Rename for intent.
4. Extract explanatory variables and functions.
5. Encapsulate conditionals and boundary conditions.
6. Separate levels of abstraction.
7. Move responsibilities to better owners.
8. Remove duplication.
9. Reduce exposed surface area.
10. Collapse needless indirection once the design is clear.

Take small steps and run the tests after each one. Never break the system to restructure it.
