# AGENTS.md

Instructions for OpenAI Codex CLI when working in this repository.

## What This Repo Is

A library of smart contract security audit skills built for elite-level Solidity auditing.
Supports Claude Code, OpenAI Codex CLI, Cursor, and Windsurf.

## How to Run the Audit Agent

To install and run from a target codebase:
```
Install https://github.com/jayjoshix/audit-skills and run audit agent on the codebase
```

If already cloned and the `audit-agent/` folder is present:
```
run audit agent on the codebase
run audit agent on src/LendingPool.sol
run audit agent on src/ --file-output
```

## Skill Location

The audit skill entry point is:
```
audit-agent/SKILL.md
```

Read that file to understand the full 4-turn orchestration flow (Discover → Prepare → Spawn → Validate).

## Agent Invocation Keywords

Trigger the skill if the user says any of:
- "audit" / "run audit" / "run audit agent"
- "security review" / "find bugs" / "check for vulnerabilities"
- "audit on [filename or directory]"

## Rules (Apply to ALL Findings)

- Read all bundle files fully before producing findings
- Never fabricate findings — every claim must trace to actual source code lines
- Apply Gates A–F before reporting any finding (see `audit-agent/references/shared-rules.md`)
- Output only CONFIRMED, PROBABLE, or HYPOTHESIS — never bare assertions
- Discard findings that require a trusted role (owner, admin, deployer) unless escalation path is proven
- Do not report the same root cause twice with a different attack path — merge instead
