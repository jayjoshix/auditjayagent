# CLAUDE.md

Instructions for Claude Code when working in this repository.

## What This Repo Is

A library of smart contract security audit skills. Built for elite-level Solidity auditing on Claude Code, OpenAI Codex CLI, Cursor, and Windsurf.

## Usage

To install and run:
```
Install https://github.com/[your-username]/auditjaygent/ and run audit agent on the codebase
```

Or if already cloned locally:
```
run audit agent on the codebase
run audit agent on src/LendingPool.sol
run audit agent on src/ --file-output
```

## Available Skills

- **audit-agent** — Full parallel multi-agent security audit with protocol-type routing, methodology backbone, and gate-validated findings

## Rules

- Read all bundle files fully before producing findings
- Never fabricate findings — every claim must trace to actual source code
- Apply Gates A–F before reporting any finding (see `references/shared-rules.md`)
- Output only CONFIRMED, PROBABLE, or HYPOTHESIS — never bare assertions
