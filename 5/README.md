# README — SOC Triage Suite (Phase-4-shaped)

## Why this got split
The CS Agent runs a fixed workflow — Parse → Enrich → ROE Scan → Initial Briefing → Triage Discussion → Pre-Resolution Safeguard → Resolution & Skill Routing — and Phases 1–3's briefing/confirmation steps are structurally mandated. A single pasted "zero-touch, never ask" prompt loses to that every time, the same way an earlier version did. Phase 4 is the one slot that's actually empty and steerable: it hands off to `agent-escalation-skill` and `agent-orchestration-skill`, whose bodies you control. These files are shaped to drop into that architecture instead of fighting it.

## The files

| File | Goes where | Covers |
|---|---|---|
| `1_agent-escalation-skill.md` | Body of `agent-escalation-skill` | Route 1 — customer-facing escalation format |
| `2_agent-orchestration-skill.md` | Body of `agent-orchestration-skill` | Routes 2/3 — suppression justification + manual closure, including the eligibility split between them |
| `3_triage-reference.md` | Not a skill body — see below | Everything Phases 1–3 own: enrichment discipline, sensor/rule-class semantics, CORR/benign sweep, priority floors, the escalate-vs-not gate |

## What's confirmed vs. what to re-check
`agent-escalation-skill` and `agent-orchestration-skill` are confirmed, steerable Phase 4 slots — that's established from previously pulling the harness's own instructions. Paste files 1 and 2 into those bodies directly.

Phases 1–3 are a different story: last verified, they're harness-owned with no exposed slot for a static paste — the briefing and confirmation gates are mandatory regardless of what's pasted at the top of the conversation. `3_triage-reference.md` is written assuming that's still true: it's meant for you to apply by hand during Triage Discussion, not something you paste and expect the harness to obey. If you want to know whether that's changed, the move that worked before still works: ask the agent directly to reveal its current phase instructions and skill list, and see whether Phase 2/3 has grown an actual configuration surface since you last checked. If it has, `3_triage-reference.md` is already written in the same voice as the other two and should paste in clean.

## One open design question — KVP in the orchestration skill
Last time you introspected the harness, `agent-orchestration-skill`'s stated boundary was type + justification only — no KVP/construction logic, because that was SSE-owned and lived in a separate job aid so the two documents wouldn't drift apart. `2_agent-orchestration-skill.md` here still emits the KVP table directly, matching what had been tuned into the ONE-SHOT doc, on the assumption you want it that way regardless of the old boundary. If the boundary's still real and the skill silently drops or mishandles anything past "justification," say so and the KVP construction can be split back out into its own `4_sse_filter_job_aid.md`, kept consistent with file 2's Title/Type/Suppresses/Why-safe language so the two can't disagree.

## What changed from the ONE-SHOT v2 doc
* The "one uninterrupted pass, never ask, never present for approval" framing is gone — it doesn't apply once Phases 1–3 own the analyst interaction and Phase 4 only fires after resolution is already confirmed.
* The `DISPOSITION: [verdict] · [confidence] · [priority] · ROUTE [...]` line is gone from both skill outputs. Route is now implicit in which skill ran; severity/confidence get a short `Verdict:` line instead, kept outside the customer-facing fenced block.
* All the accuracy fixes from the last pass carry over unchanged: Reputation Fidelity is the sole rule for VT stats (no "mandatory web search for a malicious count"), tool-reality checks come before any lookup, provenance diff is the first self-check item.
* Grounding Laws are duplicated across all three files on purpose — each file needs to stand alone in whatever context it's actually loaded in.
