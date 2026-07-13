---
name: code-review
description: >-
  Reviews code changes for correctness, security, and maintainability.
  Use when reviewing pull requests, examining a diff, or when the user asks
  for a code review or feedback on recent changes.
---

# Code Review

## Quick Start

When reviewing code:

1. Read the diff (or changed files) in full before commenting.
2. Check for correctness and edge cases first — logic bugs matter more than style.
3. Check for security issues (injection, unsafe deserialization, secrets in code, missing auth checks).
4. Assess readability and maintainability (naming, function size, duplication).
5. Confirm tests exist and actually cover the new behavior.

## Review Checklist

- [ ] Logic is correct and handles edge cases (empty input, nulls, concurrency)
- [ ] No security vulnerabilities (injection, XSS, secrets, unsafe deserialization)
- [ ] Error handling is explicit, not swallowed silently
- [ ] Naming is clear; functions are focused and appropriately sized
- [ ] No dead code, commented-out blocks, or debug prints left behind
- [ ] Tests cover the new/changed behavior, including failure paths
- [ ] Public APIs/interfaces are documented if behavior isn't obvious

## Providing Feedback

Group feedback by severity:

- 🔴 **Critical**: Bugs, security issues, or breaking changes — must fix before merge.
- 🟡 **Suggestion**: Real improvement, but not blocking.
- 🟢 **Nice to have**: Optional polish (naming, minor refactor).

Be specific: cite the file and line, explain *why* it's an issue, and propose a concrete fix rather than just flagging the problem.
