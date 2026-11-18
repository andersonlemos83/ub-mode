# Harness Install

ub-mode uses four tools by Robert C. Martin. This guide states what each one needs and where its authoritative instructions live. It does not give OS-specific commands. Install the prerequisites with whatever your platform provides (package manager, official installer, version manager), and follow each tool's README when it differs from this summary.

| Tool | Repository | Role in ub-mode |
| --- | --- | --- |
| crapper | https://github.com/unclebob/crapper | Gate: CRAP per function |
| dryer | https://github.com/unclebob/dryer | Gate: structural duplication |
| mutator | https://github.com/unclebob/mutator | Gate: mutation testing |
| uml-viewer | https://github.com/unclebob/uml-viewer | Optional: interactive architecture review |

All four analyze Clojure, Java, Go, TypeScript, Rust and Python sources. uml-viewer's project launcher targets Clojure projects.

## Requirements

- **git**, to clone the tools.
- **Python ≥ 3.11** (the `requires-python` in each `pyproject.toml`), available as `python3`. You can run crapper, dryer and mutator directly through the launcher script in each repo: the first run creates a local `.venv` inside the tool's own directory and installs the tool there.
- **mutator needs crapper checked out as a sibling directory** (`../crapper`), because it takes namespace and function names from crapper. Clone both under the same parent directory.
- **The target project's own test and coverage tooling.** crapper and mutator run the project's tests to measure coverage. Each language has its own default coverage command: see the “Coverage commands” table in crapper's README. Override it with `--coverage-command`.
- **uml-viewer only:** the Clojure CLI and Java 21 or newer. Use the `clojure` command, or `clj` if rlwrap is installed. The project launcher also uses tmux and starts an interactive Grok companion session. See the uml-viewer README.

## Install

1. Use `~/.ub-mode/tools/` as the parent directory, outside any project. Clone `crapper`, `dryer` and `mutator` into it, side by side. If they are already there, run `git pull` in each instead of cloning again.
2. Run each launcher once with `--help` (`crapper/crapper --help`, and so on). The first run bootstraps the `.venv`; afterwards `--help` should print usage.
3. If a launcher script does not run on your platform (for example on Windows without a POSIX shell), create one venv at `~/.ub-mode/tools/venv` and run `pip install -e <tool>` for each tool. The commands then live in that venv's scripts folder.
4. Put the launchers on your `PATH`, or call them by full path, for example `~/.ub-mode/tools/crapper/crapper`. The launchers resolve their own location, so you can call them from any directory.

## Running Against a Project

Run each tool from the target project's root. With no path argument, each tool walks the whole tree and skips `test`, `spec`, `vendor`, `node_modules` and `target`. Do not assume a `src/` layout. Pass paths only to narrow the scope, or use `--changed` to limit the run to git additions and edits.

| Tool | Gate command | Pass condition | Output |
| --- | --- | --- | --- |
| crapper | `crapper --threshold 9.99` | exit 0 and every reported score < 10; exit 2 means the threshold was exceeded | table on stdout; `.metrics/crap.edn` |
| dryer | `dryer --threshold 0.8` | no candidates reported (it exits 0 whenever it produces a report) | report on stdout; `.metrics/dry.edn` |
| mutator | `mutator` | exit 0; exit 3 means a survivor; exit 2 means the baseline tests failed | `.metrics/mutate/<namespace>.edn` |

Notes:

- crapper exits 2 only when the worst score is higher than `--threshold`, which is why ub-mode's “CRAP < 10” rule uses `9.99`. Without `--threshold`, crapper never fails; read the table. Verify scores against the strict `< 10` rule and the precision reported by the installed version: a `9.99` threshold is not mathematically identical to `< 10` for scores between 9.99 and 10.
- If a coverage tool is missing, or the project's tests fail, crapper scores that language at 0% coverage and still writes the snapshot. Check the run output before you trust the numbers. A wall of 0% usually means the coverage step broke, not that nothing is tested.
- mutator runs are differential once a snapshot exists. `--mutate-all` reruns everything.
- crapper installs missing coverage tooling into the project's environment, for example `coverage` and `pytest` for Python or `cargo-llvm-cov` for Rust. That is expected; record it in the log.
- Add `.metrics/` to the project's ignore file unless the team wants the snapshots versioned.

## uml-viewer (Optional Review Tool)

uml-viewer is an interactive GUI. It lays out a UML diagram and overlays crapper and mutator results from `.metrics/`. It is not a pass/fail gate.

- It marks an arrow red (a Dependency Rule violation) only when the project's policy declares `:levels`, which group namespace segments from inner to outer. Without `:levels`, nothing is marked.
- To view a real project, use the repo's `get-uml-viewer` script from the project root. It clones the viewer into a gitignored `.uml-viewer/` directory and writes a `./uml` launcher. That launcher starts a tmux session running a Grok companion.
- Inside the uml-viewer repo, `clojure -M:run <file.edn>` only opens example diagrams.

Use it when a human wants to review dependency direction. Do not report “zero red arrows” as a gate result unless `:levels` is configured and you actually looked at the diagram.

## Install Policy

The gates are part of ub-mode, so installing what is missing is part of the work:

1. **crapper, dryer and mutator:** install them without asking, as described above, under `~/.ub-mode/tools/`. Never install them inside the target repo.
2. **Coverage tooling that crapper adds to the project:** allowed. Record it in the log.
3. **System toolchains** (Python 3.11 or newer): ask once before installing. Use the platform's package manager or official installer. Never pipe remote scripts into a shell.
4. **If installation fails or the user declines:** say which tool is missing and why, continue without that gate, and apply the matching heuristics by hand: CC by inspection, duplication by reading, and boundary and mutation thinking from `goos-atdd.md`. Report the gate as not run. Never claim it passed.
