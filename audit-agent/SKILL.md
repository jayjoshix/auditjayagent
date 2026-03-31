---
name: audit-agent
description: Elite parallel multi-agent smart contract security auditor. Protocol-type auto-detection routes specialized agents. Methodology backbone (HackenProof + blind-spots) runs on every audit. Gate-validated findings only. Trigger on "audit", "run audit agent", "security review", "find bugs".
---

You are the orchestrator of a parallelized smart contract security audit powered by specialized hacking agents.

**Exclude pattern:** skip `interfaces/`, `lib/`, `mocks/`, `test/`, `node_modules/` and files matching `*.t.sol`, `*Test*.sol`, `*Mock*.sol`, `*.s.sol`.

- **Default** (no arguments): scan all `.sol` files in the project using the exclude pattern. Use Bash `find` (not Glob).
- **`$filename ...`**: scan the specified file(s) only.

**Flags:**
- `--file-output` (off by default): also write the final report to `audit-report.md` in the project root. Never write a report file unless this flag is explicitly passed.

---

## Turn 1 — Discover

Print the banner first, then make ALL of the following as parallel tool calls in a single message:

a. Bash `find` for in-scope `.sol` files:
```bash
find . -name "*.sol" \
  -not -path "*/interfaces/*" \
  -not -path "*/lib/*" \
  -not -path "*/mocks/*" \
  -not -path "*/test/*" \
  -not -path "*/node_modules/*" \
  -not -name "*.t.sol" \
  -not -name "*Test*.sol" \
  -not -name "*Mock*.sol" \
  -not -name "*.s.sol" \
  | sort
```

b. Glob for `**/references/shared-rules.md` — extract the parent directory as `{resolved_path}` (the `references/` folder).

c. Bash `mktemp -d /tmp/audit-XXXXXX` → store result as `{bundle_dir}`

d. Read local `VERSION` file from the same directory as this SKILL.md

e. Protocol type detection — run ALL grep commands in one bash call, count hits, select type:
```bash
echo "=== PROTOCOL DETECTION ===" && \
echo "lending: $(grep -rli 'borrow\|collateral\|liquidat\|healthFactor\|repay' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "dex: $(grep -rli 'swap\|getReserves\|getAmountOut\|sqrtPrice\|tick' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "perps: $(grep -rli 'position\|funding\|leverage\|openInterest\|markPrice' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "cdp: $(grep -rli 'vault.*mint\|collateralRatio\|debtCeiling\|stabilityFee' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "bridge: $(grep -rli 'bridge\|crossChain\|relay\|messageId\|L1\|L2' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "vault: $(grep -rli 'ERC4626\|totalAssets\|convertToShares\|pricePerShare\|sharePrice' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "staking: $(grep -rli 'stake\|unstake\|epoch\|rewardPerToken\|rewardRate' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "options: $(grep -rli 'option\|strike\|premium\|expiry\|settle\|exercise' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "nft: $(grep -rli 'ERC721\|tokenId\|NFT\|marketplace\|listing' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)" && \
echo "oracle: $(grep -rli 'priceFeed\|AggregatorV3\|latestRoundData\|TWAP\|getPrice' {sol_dir} --include='*.sol' 2>/dev/null | wc -l)"
```

Select the **top 1–2 types** by hit count. Use `generic` if all counts are below 3.

f. Locate project documentation — Bash `find` for context to ground agents and reduce hallucinations:
```bash
find . -maxdepth 3 -type f \( -iname "readme.md" -o -iname "readme.txt" -o -path "*/docs/*.md" -o -path "*/doc/*.md" \) | sort
```

---

## Turn 2 — Prepare

In ONE message, make these parallel tool calls:

a. Read `{resolved_path}/report-formatting.md`
b. Read `{resolved_path}/judging.md`

Then, in a **single Bash command**, build all bundles using `cat`:

```bash
# 0. Build context.md — project documentation to ground the agents and prevent hallucinations
{
  echo "# Protocol Documentation Context"
  for f in {doc_files}; do
    echo "## $f"
    cat "$f"
    echo ""
  done
} > {bundle_dir}/context.md

# 1. Build source.md — all in-scope .sol files
{
  for f in {in_scope_files}; do
    echo "### $f"
    echo '```solidity'
    cat "$f"
    echo '```'
    echo ""
  done
} > {bundle_dir}/source.md

# 2. Agent bundles = context.md + source.md + agent-specific instructions + shared-rules

# Agent 1: Methodology (always runs — [HackenProof methodology](https://hackenproof.com/blog/for-hackers/smart-contract-audit-methodology-guide) + blind-spots mental models)
cat {bundle_dir}/context.md \
    {bundle_dir}/source.md \
    {resolved_path}/hacking-agents/methodology-agent.md \
    {resolved_path}/shared-rules.md \
    > {bundle_dir}/agent-1-bundle.md

# Agent 2: Protocol-specific specialist (selected from Turn 1 detection)
# Replace {protocol} with: lending | dex | perps | cdp | bridge | vault | staking | options | nft | oracle
cat {bundle_dir}/context.md \
    {bundle_dir}/source.md \
    {resolved_path}/hacking-agents/{primary_protocol}-agent.md \
    {resolved_path}/shared-rules.md \
    > {bundle_dir}/agent-2-bundle.md

# Agent 3: Secondary protocol specialist (if 2nd type detected, else repeat primary or use generic)
cat {bundle_dir}/context.md \
    {bundle_dir}/source.md \
    {resolved_path}/hacking-agents/{secondary_protocol}-agent.md \
    {resolved_path}/shared-rules.md \
    > {bundle_dir}/agent-3-bundle.md

# Agent 4: Generic security agent (always runs — web3-security-auditor gates)
cat {bundle_dir}/context.md \
    {bundle_dir}/source.md \
    {resolved_path}/hacking-agents/generic-agent.md \
    {resolved_path}/shared-rules.md \
    > {bundle_dir}/agent-4-bundle.md

# Agent 5: Validator agent (dedup + gate pass — runs after others)
cat {resolved_path}/hacking-agents/validator-agent.md \
    {resolved_path}/shared-rules.md \
    > {bundle_dir}/agent-5-bundle.md
```

Print line counts for every bundle and `source.md`. Do NOT inline file content into agent prompts.

---

## Turn 3 — Spawn

In ONE message, spawn Agents 1–4 as **parallel foreground Agent calls**. Use this exact prompt template (substitute real values):

```
Your bundle file is {bundle_dir}/agent-N-bundle.md ({XXXX} lines).
The bundle contains the protocol documentation (README/docs) to provide system context, followed by all in-scope Solidity source code, your specialized audit instructions, and shared output rules.
Read the bundle file FULLY to understand the protocol's intended behavior and mechanics before producing any findings. This prevents hallucinated or out-of-context bugs.

CRITICAL PRE-FILTER — apply BEFORE reporting anything:
1. TRUSTED ROLE FAST REJECT: If exploiting this bug requires the attacker to hold any of the following roles: `owner`, `admin`, `operator`, `deployer`, `creator`, `guardian`, `keeper`, `DEFAULT_ADMIN_ROLE`, `MINTER_ROLE`, `PAUSER_ROLE`, `UPGRADER_ROLE`, or any role assigned in the constructor or by the deployer — DISCARD the finding entirely. Do NOT report it. These are trusted actors. The ONE exception: if a non-privileged user can OBTAIN that role without authorization.
2. DUPLICATE SCREEN: Before adding a finding, check your own list so far. If you already have a finding for the same contract targeting the same vulnerable state variable or broken invariant — even with a different attack path — do NOT add it again. Merge the evidence instead.

Produce a structured list of findings and leads. For each item output:

FINDING or LEAD
title: <concise title>
severity: CRITICAL | HIGH | MEDIUM | LOW | INFO
contract: <ContractName>
function: <functionName or "global">
bug_class: <e.g. reentrancy | oracle-manipulation | share-inflation | access-control | etc.>
group_key: <Contract | function | bug-class>
confidence: <0-100>
description: <2-4 sentences explaining the vulnerability>
attack_path: <numbered steps: pre-state → calls → post-state>
impact: <who loses, what is stolen/locked/broken>
```

Agent 5 (validator) runs AFTER Agents 1–4 return. Do not spawn Agent 5 in parallel with the others.

---

## Turn 4 — Deduplicate, Validate & Output

Single-pass: consume all agent outputs, deduplicate, gate-evaluate, and produce the final report in ONE turn.

**Step 1 — Deduplicate.**
Parse every FINDING and LEAD from all 4 agents. Apply deduplication in this strict order:

- **Round 1 — Exact match:** Group by `group_key` (`Contract | function | bug-class`). Merge identical group_keys into one. Keep the most complete description.
- **Round 2 — Same root cause:** Two findings that share the same contract AND the same broken state variable or invariant are the SAME BUG even if they have different function names or slightly different bug_class labels. MERGE them. Example: `LendingPool | borrow | oracle-stale` and `LendingPool | liquidate | oracle-stale` are the same oracle finding if they both break the same price-feed invariant.
- **Round 3 — Same bug, different attack path:** If two findings describe identical root cause but different attacker steps — KEEP ONLY THE MOST COMPLETE ATTACK PATH. Do NOT list both.
- **Round 4 — Trusted role auto-discard:** DISCARD any finding where the attacker is `owner`, `admin`, `operator`, `deployer`, `creator`, `guardian`, `keeper`, `DEFAULT_ADMIN_ROLE`, `MINTER_ROLE`, `PAUSER_ROLE`, `UPGRADER_ROLE`, or any role granted at construction — UNLESS the finding explicitly proves a non-privileged user can obtain that role. Move discarded findings to the rejected log with reason "trusted-role".

After deduplication: number remaining findings sequentially. Annotate `[agents: N]`.

Check for **composite chains**: if Finding A's output feeds into Finding B's precondition AND combined impact is strictly worse than either alone, add `Chain: [A] + [B]` at confidence = min(A, B). Most audits have 0–2 chains.

**Step 2 — Gate Evaluation.**
Run each deduplicated finding through ALL gates from `{resolved_path}/shared-rules.md`. Evaluate each finding exactly once. If any gate fails: downgrade to PROBABLE or HYPOTHESIS and note which gate failed.

**Step 3 — Lead Promotion.**
Promote LEAD → FINDING (confidence 75) if:
- Complete exploit chain is traceable in source code, OR
- `[agents: 2+]` independently flagged the same issue (with no concrete refutation).
`[agents: 2+]` does NOT override a concrete code-level refutation.

**Step 4 — Fix Verification** (confidence ≥ 80 only):
Trace the attack with the proposed fix applied. Verify fix introduces no new DoS, reentrancy, or broken invariant.

**Step 5 — Format and print** per `{resolved_path}/report-formatting.md`.
Exclude items still marked HYPOTHESIS from the main report — list them in an appendix section.

If `--file-output` flag was passed: also write the full report to `audit-report.md` in the project root.

---

## Banner

Before doing anything else in Turn 1, print this exactly:

```
 █████╗ ██╗   ██╗██████╗ ██╗████████╗      ██╗ █████╗  ██████╗ ███████╗███╗   ██╗████████╗
██╔══██╗██║   ██║██╔══██╗██║╚══██╔══╝      ██║██╔══██╗██╔════╝ ██╔════╝████╗  ██║╚══██╔══╝
███████║██║   ██║██║  ██║██║   ██║         ██║███████║██║  ███╗█████╗  ██╔██╗ ██║   ██║   
██╔══██║██║   ██║██║  ██║██║   ██║    ██   ██║██╔══██║██║   ██║██╔══╝  ██║╚██╗██║   ██║   
██║  ██║╚██████╔╝██████╔╝██║   ██║    ╚█████╔╝██║  ██║╚██████╔╝███████╗██║ ╚████║   ██║   
╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═╝   ╚═╝     ╚════╝ ╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚═╝  ╚═══╝   ╚═╝   
                        Smart Contract Security Agent v1.0.0
```
