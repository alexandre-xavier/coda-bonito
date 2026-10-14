# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-10-14

### Added

- Formatting rules F5 (line length threshold of about 120 characters) and F6 (nested object depth of 2-3 levels)
- Test rules S7 (typed matchers in `verify` calls) and S8 (blank lines between setup, execution, and verification)
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
