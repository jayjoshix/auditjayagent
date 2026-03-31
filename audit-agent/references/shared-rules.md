# Shared Rules: Gate Evaluation & Output Contract

All agents must apply these rules before producing any finding. These are non-negotiable.

---

## False-Positive Firewall — Gates A–F

Never output a FINDING unless ALL gates pass. If any gate fails, output LEAD or HYPOTHESIS with the gate that failed noted.

### Gate 0 — Permission to Stop
If you cannot prove exploitability with concrete state transitions: output `Not enough evidence to claim exploitability` and stop.

### Gate A — Value at Risk
The finding must cause one of: **theft of funds**, **permanent lock of funds**, **insolvency / bad debt creation**, **free minting**, or **collateral bypass**.
- Design flaws, economic inefficiencies, or UX issues that cannot be escalated to one of these outcomes = **INFO** at most.

### Gate B — Attacker Model (explicit)
State clearly:
- Is the attacker an EOA, a malicious contract, or a privileged role?
- Does the attacker need flash liquidity? MEV? Specific approvals?
- Are allowlists, roles, or whitelists involved? How does the attacker obtain them?

### Gate C — Reachability
Identify ONE of:
- An attacker-callable `external` or `public` entrypoint, OR
- An attacker-triggerable callback path, OR
- A reachable `initialize`/upgrade misconfiguration.
If none exists: **Gate C fails → HYPOTHESIS**.

### Gate D — Broken Invariant
Name the invariant explicitly. Example: "totalShares * pricePerShare == totalAssets". Show the exact state transition that breaks it.

### Gate E — Exploit Narrative
Provide 3–7 concrete steps:
```
Pre-state: [describe initial on-chain state]
1. Attacker calls ContractA.functionB(params)
2. ...
Post-state: [who gained, who lost, by how much]
```

### Gate F — Disproof Attempt (mandatory)
Actively try to kill your own finding. Check:
- `minOut` / slippage protections on all swap paths
- `nonReentrant` modifiers and CEI pattern
- Allowlists / selector-level gating
- Monotonic nonces or epochs
- Balance-delta accounting (post-transfer balance checks)
- Auth modifiers (`onlyOwner`, `onlyRole`)
- Oracle tolerance windows

If a disproof holds conclusively: **downgrade to HYPOTHESIS**.

---

## Known False Positive Patterns

Before finalizing, check these common FPs:

**FP-1: Pre-hook ignored.** Searched for stale state, missed `refresh_*`, `accrue_*`, `sync_*` call earlier in the flow. **Fix:** Trace the full callchain from the entry point.

**FP-2: Intentional design.** Hook/path "missing" is in fact a deliberate different integration point. **Fix:** Read all entry points and code comments.

**FP-3: Lazy accounting.** Global aggregate lags but per-user reconciliation happens on interaction. All participants scale by the same factor. **Fix:** Verify the inverse operation converges.

**FP-4: Cap + proportional scaling.** Reported a cap without seeing the immediate proportional refund/rescale that follows. **Fix:** Always read 10+ lines past any `min()` or clamp.

**FP-5: Same-tx staleness.** Reported divergence between two reads in the same atomic transaction. **Fix:** Confirm both reads are in the same tx — storage is consistent unless modified by intermediate writes.

**FP-6: Protocol-favorable rounding.** Truncation consistently favors the protocol, and max extractable value ≤ 1 unit per operation. **Fix:** Classify as **INFO** only.

**FP-7: Non-EVM donation.** Assuming ERC-20-style `transfer()` donation attacks on SUI/Move/Solana. **Fix:** On SUI check `balance::join()`. On Solana check explicit PDA deposit instructions.

**FP-8: Secondary guard blocks.** Guard A is broken but Guard B independently blocks the exploit. **Fix:** After bypassing Guard A, trace Guards B and C.

**FP-9: Admin trust scope.** Reporting admin-only functions when admin is explicitly trusted in scope. **Fix:** Check README/scope for trust assumptions.

**FP-10: Bounds enforcement exists.** Reporting overflow/div-by-zero without checking `require(x > 0)`, `require(x <= MAX)`, or `saturating_sub`. **Fix:** Grep for the bounds check.

---

## Output Contract

For every finding, output EXACTLY one of:

### CONFIRMED
```
Title:
Severity: Critical | High | Medium | Low
Type: <bug_class>
Contract: <ContractName>
Function: <functionName>
Agents: [agent-N, agent-M]

Impact: <who loses, what is stolen/locked/broken>
Attacker Model: <EOA|contract, flash loan Y/N, role needed>
Reachability: <entry point or callback path>
Broken Invariant: <stated invariant + how it breaks>

Attack Path:
1. Pre-state: ...
2. Attacker calls ...
3. ...
N. Post-state: [profit/loss amounts]

Affected Code: <file.sol:functionName> (line X if known)
Fix Direction: <1 paragraph>
```

### PROBABLE
Same as CONFIRMED, plus:
```
Missing Evidence: <1 line>
Next Validation: <1 concrete action>
```

### HYPOTHESIS
```
Pattern Hit: <what looks suspicious and why>
Gate(s) Failed: <Gate A|B|C|D|E|F — reason>
Next Validation Actions:
1. <specific grep or code read>
2. <specific trace or test>
```

**If uncertain, prefer HYPOTHESIS.**
