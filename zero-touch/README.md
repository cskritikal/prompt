# Zero-Touch Prompt Set

Full-auto rebuild of the SOC investigation suite, optimized against the agent's actual skill-routing behavior (`agent-escalation-skill`, `agent-orchestration-skill`). Goal: **near-zero interaction, maximum output density.** Every analyst/customer confirmation seam in the original suite has been removed; the agent decides on the evidence, documents residual uncertainty inline, and emits the finished artifact.

## What changed vs. the original prompts
Three confirmation seams removed:
1. **Triage** no longer ends by inviting the analyst to "confirm, redirect, or add context." It routes AND emits the artifact in one pass.
2. **Escalation** no longer "surfaces recommended SOC-side actions to the analyst before drafting." Those actions are written straight into *What is Recommended*.
3. **Orchestration** no longer "confirms the suppress determination with the analyst." The determination is made autonomously during triage.

Everywhere a gate previously implied asking a human (verification-pending bar, eligibility gates, same-entity test), the gate is now a self-check the agent decides — never a question. Residual uncertainty is carried as a `Gaps`/Context caveat inside the artifact, never as a stop.

Orchestration output is also now **actionable, not advisory**: the agent emits the actual 2–4 KVP filter rows (field/operator/value) under a `### Orchestration Justification` header, or a single `### Manually Closing` block when no durable filter is safe. The old "construction is SSE-owned, no KVP" boundary is dropped for the basic rows; only scope-tier/array/template construction stays with the SSE.

All grounding/density/IOC/MITRE laws from the originals are preserved.

**Priority = response urgency, set after enrichment.** Priority is defined by the action it should trigger, not the rule's built-in severity: *Filter/Close* (client doesn't need to see it) · *Low* ("when you get a chance") · *Medium* ("look at this today") · *High/Critical* ("right now" / confirmed ongoing malice). The prompts re-derive priority from post-enrichment evidence and treat escalation as a destination earned after investigation, never a shortcut — ambiguity leaning benign with no malice indicator is Filter/Close or Low, never Medium by default. A dedicated **identity / sign-in class** (impossible travel, distant consecutive sign-ins, risky sign-in) requires pulling device managed/compliant state, MFA, and Conditional Access from Entra/Intune before disposition: a compliant managed device with MFA satisfied is benign (close/suppress), not a Medium escalation — while AiTM/token-theft indicators still keep it suspicious.

## Files

| File | Use |
|---|---|
| `0_oneshot_megaprompt.md` | **Paste-once, fully self-contained.** Silent triage → route → emits only the disposition line + the correct artifact(s). Closest to zero interaction. Start here. |
| `1_triage_router.md` | Split-architecture router. Routes and then auto-invokes the format spec(s) below in the same pass. Use if you want to keep triage and formatting as separate skill bodies. |
| `2_escalation_format_spec.md` | Body of `agent-escalation-skill`. Customer-facing write-up, standard + action-taken. |
| `3_orchestration_justification_spec.md` | Body of `agent-orchestration-skill`. Emits a `### Orchestration Justification` block ending in **2–4 KVP filter rows** (field/operator/value) the analyst applies directly, or a single `### Manually Closing` block when no durable filter is safe. Standard + action-taken. |
| `4_sse_filter_construction_job_aid.md` | **Human/SSE reference — not fed to the agent.** Deeper construction (scope-tier, array logic, TAPs) and the cross-check that the emitted KVP rows hold the FP/TP boundary. |
| `5_phish_oneshot.md` | Standalone phishing investigation. Single pass → Part 1 escalation + Part 2 orchestration verdict (2A/2B/2C). |

## How to run
* **Simplest (recommended for zero interaction):** paste `0_oneshot_megaprompt.md`, then paste the raw alert. One artifact comes back, no follow-up.
* **Split skills:** wire `2` and `3` as the respective skill bodies; paste `1_triage_router.md` upstream. Triage routes and the skills format in the same pass.
* **Phishing:** paste `5_phish_oneshot.md`, then the raw Defender/message-trace data.
* **Filter build:** after orchestration recommends suppression, the SSE opens `4` to construct the filter.

## Output shape
Every prompt's first output character is the artifact (or the one-line `DISPOSITION:` header in the one-shot/triage). No preamble, no progress narration, no closing question. If no alert content is supplied, the prompt says so in one line and stops — the only permitted stop.
