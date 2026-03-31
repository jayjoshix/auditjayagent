# Judging: Severity Criteria

## Severity Definitions

### Critical
- Direct theft of funds from ANY user by an **unprivileged** attacker
- Protocol-wide insolvency or drain (not isolated to a single user)
- Permanent, irreversible lock of user funds with no admin recovery path
- Free infinite minting of tokens backed by real assets
- Requires no special role, no governance vote, executable in a single transaction

### High
- Significant loss of funds (>20% of a pool or material amount) achievable by an attacker
- Permanent lock or irreversible damage affecting a subset of users
- Partial theft enabling repeated extraction over time
- Liquidation DoS causing bad debt to accumulate systemically
- Requires at most one unprivileged on-chain action (e.g., being a depositor)

### Medium
- Loss of funds requiring specific market conditions, multi-step setup, or economic preconditions
- Griefing attacks that cause significant inconvenience or minor economic loss
- Access control misconfiguration giving a user more power than intended (but not leading to direct theft alone)
- Oracle or accounting inconsistencies that create extractable value under specific conditions
- Requires privileged role to exploit, where that role is realistically obtainable

### Low
- Unlikely edge cases with low economic impact
- Code quality issues that could become vulnerabilities with protocol changes
- Missing events, incorrect error messages, documentation mismatches
- Economic inefficiencies that cost users minor amounts non-exploitably

### Informational (Info)
- Best practices not followed (no security impact)
- Rounding that consistently favors the protocol (≤1 wei per op)
- Gas optimizations
- Suggestions to improve code readability

---

## Severity Modifiers

- **Upgrade/Downgrade by 1 level** if:
  - Requires specific market conditions (down)
  - Requires flash loan but is trivially executable (neutral)
  - Admin is the attacker AND admin is NOT explicitly trusted in scope (up)
  - Loss is less than $1000 in realistic scenarios (down)

- **No Griefing-as-High:** A DoS that causes user inconvenience without fund loss is at most Medium unless the DoS permanently destroys value.

- **The 10-Line Rule:** Before scoring, read at least 10 lines before and after the suspicious code. Most mis-scorings come from reading one function in isolation.

---

## Composite Chain Rule

If two Medium findings, when chained, create a Critical-level impact:
- Score the CHAIN separately as High or Critical
- Keep individual findings at their standalone score
- Label: `Chain: [M-1] + [M-2] → [C-1]`
