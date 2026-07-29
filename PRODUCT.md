# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Users

Primary users are Security Operations Center (SOC) Analysts, Threat Responders, and Security Engineers. Their situation involves triaging automated alerts, managing LLM-driven security agents, preventing hallucinated or miscalibrated threat outputs, and maintaining operational control over triage workflows under high alert volume.

## Product Purpose

SOC Agent Steering Snippets is an open-source reference library and interactive documentation web app designed to correct LLM-powered security agents during alert triage, log enrichment, threat routing, and artifact generation. Success means giving security teams deterministic, single-turn post-output correction prompts that immediately enforce operational rules without full prompt re-engineering.

## Positioning

Unlike generic prompt collections, this product provides class-specific, evidence-grounded steering snippets specifically tuned for real-world SOC workflows (e.g., identity sign-ins, network tunneling, LOLBins, phishing sender auth), backed by a strict one-shot base triage prompt and exact line-budget contracts.

## Operating Context

Used by SOC analysts during active incident triage and by AI/security engineers building automated agent controller loops. Operates alongside SIEMs, EDRs, VirusTotal lookups, and orchestration playbooks in high-stakes, fast-paced security environments.

## Capabilities and Constraints

- **Capabilities**: Categorized snippet index across 7 operational areas (Workflow Steering, Grounding Integrity, Class-Specific Rules, Priority Routing, Artifact Formatting, Line 1 Enforcement, and Per-Class Escalation Templates).
- **Constraints**: Pure web frontend reference and interactive documentation site. Zero-dependency markdown/web format, preserving verbatim log extraction rules and defanged indicator security standards.

## Brand Commitments

- **Tone & Voice**: Authoritative, precise, high-density, mission-critical security ops tone.
- **Key Assets**: `steering_snippets.md`, `soc_one_shot_prompt.md`, and `README.md`.

## Evidence on Hand

- Standard zero-touch triage system prompt in [soc_one_shot_prompt.md](file:///c:/Users/ConnorSmith/Downloads/prompt/soc_one_shot_prompt.md).
- Complete modular steering snippet library in [steering_snippets.md](file:///c:/Users/ConnorSmith/Downloads/prompt/steering_snippets.md).

## Product Principles

1. **Evidence First**: Zero tolerance for hallucinated telemetry, N/A placeholders, or ungrounded conclusions.
2. **Deterministic Control**: Immediate correction via single-pass steering injection without re-arguing or narrating progress.
3. **High Information Density**: Compact, scannable, operational outputs respecting strict line budgets.
4. **Autonomous Fidelity**: Self-serve enrichment over title-based escalation panic.

## Accessibility & Inclusion

- High contrast, dark-mode optimized, keyboard-navigable documentation interface designed for multi-monitor SOC control rooms.
