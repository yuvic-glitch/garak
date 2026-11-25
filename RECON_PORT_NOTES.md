# Port: Bar's recon onto main (branch tool-chain-recon-from-bar)

Adds the deep-recon + behavioral-baseline recon from barlanyado/garak@7e90e413 ("improved tool
chain attack", remote `bar`, branch tool_chain_attack) to the main-branch AgentBreaker/ToolChain
probe, which previously had only discovery + analysis.

## What was ported (into garak/probes/agent_breaker.py, base AgentBreaker class)
- DEFAULT_PARAMS: behavioral_probe_enabled (True), behavioral_probe_calls_per_tool (5).
- __init__: self.tool_profiles, self.tool_behaviors.
- Methods (verbatim from bar, with one mechanical adaptation):
  - _perform_deep_recon(generator)  -> {tool: profile}  (asks the agent to describe each tool,
    parses via PARSE_TOOL_PROFILE)
  - _perform_behavioral_probe(generator) -> {tool: [observations]}  (red-team proposes benign
    probes via BEHAVIORAL_PROBE, sends them, parses outcomes via PARSE_TOOL_BEHAVIOR)
  - _format_tool_profile / _format_tool_behavior (static renderers)
  - _perform_recon_probes() orchestration helper (deep recon always; behavioral if enabled)
- _format_tools_for_analysis (base + ToolChain override) now renders the profile + observed
  behaviour under each tool, so ANALYSIS and chain planning are grounded in recon, not just the
  static description. All callers pass self.tool_profiles / self.tool_behaviors.
- _perform_recon_probes() is called before _analyze_attackable_tools() in both the base
  _create_init_attempts and the ToolChain setup.
- Prompts added to garak/data/agent_breaker/prompts.yaml: DEEP_RECON, PARSE_TOOL_PROFILE,
  BEHAVIORAL_PROBE, PARSE_TOOL_BEHAVIOR.

## Adaptation to main's API
- bar's `self._detector._extract_json(x)` -> main's module-level `extract_json_object(x)`
  (main's convention; _get_model_response / _setup_parse_model names already match).

## Deliberately NOT ported
- Fault-injection probe (`_perform_fault_injection_probe`, fault_probe_* params): it is opt-in
  (default off) AND its `FAULT_PROBE` prompt is MISSING from bar's prompts.yaml, so it is
  incomplete upstream. Left out to keep the port clean; can be finished later if wanted.

## Verification done
- py_compile OK; prompts.yaml parses with all 11 keys; the two static formatters render correctly
  on sample profile/behaviour dicts.
## Still needed (not runnable here - no garak install / live agents)
- Live run against an agent to exercise _perform_deep_recon / _perform_behavioral_probe end-to-end
  and the existing tests/probes/test_agent_breaker*.py suite under a garak env.
