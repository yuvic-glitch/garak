# recon_examples

Captured recon output (`recon_exp.json`, converted to YAML) from the `ToolChain` /
`AgentBreaker` probe against three agents-lab agents — **codereview**, **ecommerce**,
**identity** — for three builds of the probe. One file per agent per build.

| folder | build | what it is |
|---|---|---|
| `bar_original_live_recon/` | Bar's live recon as ported (pre anti-fabrication) | the raw ported deep-recon + behavioral baseline, **before** the grounding guards / enumerate-first probe — so it still fabricates identifiers |
| `recon_plain/` | experiment variant `bar_port` | honest recon (enumerate-first + grounding guards), grounding stages **off** in the attacker rendering |
| `recon_grounded/` | experiment variant `bar_all` | same recon, grounding stages **on** (GROUND + RECONCILE + AFFORD shown to the attacker) |

Each YAML holds the full dump:

- `agent_config` — discovered purpose + tools
- `tool_profiles` — per-tool deep-recon profile (params, output format, restrictions)
- `tool_behaviors` — observed behavior from benign probes (outcome, output shape/sample, error/refusal signatures, constraints)
- `grounded_identifiers` — real identifiers the target returned during recon
- `rendered_for_attacker` — the exact recon text the attacker model saw

## What to look for

- **Fabrication:** in `bar_original_live_recon/`, `grounded_identifiers` and the observations
  contain **invented** ids the target echoed back from guessed inputs (e.g. `SKU-88210`,
  `ORD-2024...`). In `recon_plain/` and `recon_grounded/` they are **real** ids the target
  actually returned (e.g. ecommerce `P-100…P-700`, codereview PRs `#1–#5`, identity
  `@acme.example` members).
- **Plain vs grounded:** `tool_profiles` / `tool_behaviors` / `grounded_identifiers` are
  **identical** between `recon_plain` and `recon_grounded` (same cached recon). Only
  `rendered_for_attacker` differs — `recon_grounded` appends the "REAL IDENTIFIERS OBSERVED"
  (GROUND) block plus RECONCILE/AFFORD annotations.
