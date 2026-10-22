# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.3.0] - 2026-10-22

### Changed

- **BREAKING:** renamed the skill `bonito` to `coda-bonito`. The manual command changes from `/coda:bonito` to `/coda:coda-bonito`; there is no alias. The plugin (`coda`) and marketplace (`coda-bonito`) names are unchanged
- Plugin and marketplace descriptions now refer to the `coda-bonito` skill

## [0.2.0] - 2026-10-15

### Added

- Formatting rule F5: line length as a soft target (~120 characters including indentation). A call followed by a chain may stay on one line if it is under the limit; otherwise it is extracted to a variable with one argument per line
- Formatting rule F6: one-dot-per-line for chains (2+ dots); lambda bodies are exempt. Nested object construction is limited to 3 levels (prefer 2 in new code), with a marked 3-level example
- Test rule S7: typed matchers in `verify` calls, arguments one per line (F2)
- Test rule S8: three logical test blocks (setup, execution, verification); within-block grouping per F4
- Verification checklist entries for the four new rules
- Skill description now covers line length, method chains, nested objects, typed matchers, and test structure

### Changed

- F2 behaviour: now applies everywhere, tests included. F5 is the exception for a call followed by a chain

## [0.1.0] - 2026-10-08

### Added

- `bonito` skill with Java and Groovy style rules:
  - Parameter ordering (P1, P2): domain data first, request context last
  - Naming (NM1–NM5): unit suffixes, role names, test names, no boolean prefixes, collection names
  - Formatting (F1–F4): one parameter per line, multi-line calls, blank-line rules
  - Types (T1–T3): primitives vs wrappers, `Optional` only as a return type, prefer `var`
  - Cleanup (C1–C4): unused code, forwarding factories, wrappers that do nothing
  - Logging (L1–L3): secrets, PII, request values, MDC context
  - Constants (N1–N3): named constants, `Duration`, holder classes, shared limits
  - Lambdas (LM1–LM3): exception assertions, ternaries, long lambdas
  - Comments (CM1): explain why, not what
  - Tests (S1–S6): frameworks, Spock labels, verify assignment, `doReturn`/`doThrow`, `StepVerifier`
- Decision trees and a verification checklist
- Claude Code plugin (`coda`) and marketplace (`coda-bonito`) manifests

[Unreleased]: https://github.com/alexandre-xavier/coda-bonito/compare/v0.3.0...HEAD
[0.3.0]: https://github.com/alexandre-xavier/coda-bonito/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/alexandre-xavier/coda-bonito/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/alexandre-xavier/coda-bonito/releases/tag/v0.1.0
