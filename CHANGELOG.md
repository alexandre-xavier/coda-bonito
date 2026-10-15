# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-10-14

### Added

- Formatting rule F5, an exception to F2 that applies only to a call followed by a chained call: the call stays on one line when it fits in 120 characters (indentation included), with the chain on the next line. When it exceeds 120 characters, it is extracted to a variable with one argument per line (F2). A standalone call follows F2
- Formatting rule F6: one dot per line in every chain (constructors included; plain getters in argument lists stay inline only when a chain follows and the line fits in 120 characters), alongside extracting deep nesting to intermediate variables, with a 3-level chain example
- Test rules S7 (typed matchers in `verify` calls, one argument per line) and S8 (blank lines between setup, execution, and verification)
- Verification checklist entries for the four new rules

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

[Unreleased]: https://github.com/alexandre-xavier/coda-bonito/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/alexandre-xavier/coda-bonito/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/alexandre-xavier/coda-bonito/releases/tag/v0.1.0
