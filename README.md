# auditjayagent — Smart Contract Security Agent

A parallelized, protocol-type-aware smart contract security audit skill. Built for elite Web3 auditing.

Inspired by [pashov/skills](https://github.com/pashov/skills).

---

## Usage

### From any AI coding assistant (Claude Code, Codex CLI, Cursor, Windsurf)

**Install and run (public GitHub):**
```
Install https://github.com/jayjoshix/auditjayagent and run audit agent on the codebase
```

**Run locally (already cloned):**
```
run audit agent on the codebase
run audit agent on src/LendingPool.sol
run audit agent on src/ --file-output
```

**With file output (saves `audit-report.md` in project root):**
```
run audit agent on the codebase --file-output
```

---

## How It Works

The audit agent follows a strict 4-turn orchestration flow:

| Turn | What Happens |
|---|---|
| **Turn 1 — Discover** | Prints banner, finds .sol files, auto-detects protocol type, creates temp directory |
| **Turn 2 — Prepare** | Builds source bundles (source code + agent instructions) via `cat` |
| **Turn 3 — Spawn** | Launches 4 parallel agents simultaneously (methodology + 2 specialists + generic) |
| **Turn 4 — Report** | Deduplicates, validates via Gates A–F, formats final report |

### Protocol Auto-Detection

The agent greps for protocol-specific keywords and selects the best 2 matching specialist agents:

| Type | Specialist Agent |
|---|---|
| Lending | lending-agent (bad debt, interest ordering, liquidations) |
| DEX / AMM | dex-agent (oracle, slippage, reentrancy, tick math) |
| Perpetuals / CLOB | perps-agent (margin extraction, leverage bypass, liquidation deadlocks) |
| CDP | cdp-agent (stability fees, liquidation insolvency, flash mints) |
| Bridge | bridge-agent (replay attacks, gas limit, calldata injection) |
| Vault / ERC4626 | vault-agent (share inflation, rounding, reward checkpoints) |
| Staking | staking-agent (reward accounting, share inflation, liquid staking) |
| Options | options-agent (tick math, commission bypass, solvency math) |
| NFT | nft-agent (reentrancy via callbacks, signature replay, auction) |
| Oracle | oracle-agent (staleness, TWAP, decimals, read-only reentrancy) |

All audits also run:
- **Methodology agent** (always) — [HackenProof 10-phase](https://hackenproof.com/blog/for-hackers/smart-contract-audit-methodology-guide) + blind-spots mental models
- **Generic agent** (always) — Universal checklist (reentrancy, access control, math, signatures)
- **Validator agent** (after others) — 14 false positive patterns + Gate A–F re-verification

---

## Finding Quality

Every finding is classified as:
- **CONFIRMED** — All 6 gates passed, full exploit path proven
- **PROBABLE** — Most gates pass, one piece of evidence missing
- **HYPOTHESIS** — Pattern spotted but insufficient evidence — includes next validation steps

Gates A–F (from `references/shared-rules.md`):
- **A**: Value at risk (theft/lock/insolvency — not just a design issue)
- **B**: Realistic attacker model
- **C**: Concrete reachable entry point
- **D**: Named broken invariant with proof
- **E**: 3+ step exploit narrative with pre/post state
- **F**: Active disproof attempt (all guards checked)

---

## File Structure

```
auditjayagent/
├── CLAUDE.md                          # AI assistant instructions
├── README.md
└── audit-agent/
    ├── SKILL.md                       # Master orchestrator (4-turn flow)
    ├── VERSION                        # 1.0.0
    └── references/
        ├── shared-rules.md            # Gates A-F + 14 FP patterns + output contract
        ├── report-formatting.md       # Report template
        ├── judging.md                 # Severity criteria (Critical/High/Medium/Low/Info)
        └── hacking-agents/
            ├── methodology-agent.md   # HackenProof phases + blind-spots
            ├── lending-agent.md       # Lending + debt accounting
            ├── dex-agent.md           # DEX + Uniswap V3 ticks
            ├── vault-agent.md         # ERC4626 + yield aggregators
            ├── cdp-agent.md           # CDP + stablecoins
            ├── perps-agent.md         # Perpetuals + CLOB
            ├── bridge-agent.md        # Bridges + cross-chain
            ├── staking-agent.md       # Staking + liquid staking
            ├── oracle-agent.md        # Oracle feeds
            ├── options-agent.md       # Options AMMs
            ├── nft-agent.md           # NFT marketplaces + lending
            ├── generic-agent.md       # Universal checklist
            └── validator-agent.md     # FP filter + gate re-verification
```

---

## Skills Coverage

This agent integrates knowledge from 47 specialized auditing skill modules covering:
- Lending, accounting, Blend V2 specifics
- DEX, Uniswap V3 ticks, shared debt
- ERC4626, yield aggregators
- CDP, CDP-yield strategies
- Perpetuals, derivatives
- Bridges, cross-chain
- Staking pools, liquid staking
- Oracle protocols
- Options AMMs, options vaults
- NFT marketplaces, NFT lending
- Algorithmic stablecoins, reserve currency
- Prediction markets, insurance, gaming
- HackenProof methodology, blind-spots mental models
- False positive filter (14 real-world FP patterns)
- Web3 security auditor (Gates A–F)

---

## Version

`1.0.0` — See [CHANGELOG](CHANGELOG.md) for updates.
