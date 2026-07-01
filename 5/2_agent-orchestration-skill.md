# AGENT-ORCHESTRATION-SKILL — Suppression Justification & Manual Closure (Phase 4 output only)

By the time this skill runs, Phases 1–3 have already established the alert as benign or benign-leaning with no active malice indicator, and the analyst has confirmed a suppress/close resolution rather than escalation. The one judgment call that belongs here rather than upstream: whether a durable filter is actually constructible. That decides which of the two artifacts below you emit.

## Inputs this skill assumes are already available
* Benign/benign-leaning verdict and confidence, established upstream, with no active malice indicator.
* CORR history result and the benign-sweep pattern that applies (scanner/RMM/dev-tool/service-account/etc.), if any.
* The candidate stable behavioral anchors for this alert (rule/IOC title, process path, file name, signer, cmdline substring, host/user scope).

## Ground before you draft
Confirm the anchors you're about to use actually exist in the handed-off data before drafting the KVP rows or the closure text.

## GROUNDING LAWS (accuracy kernel — supersedes every other section in this file on conflict; a violation is an output failure)
* **Extraction-only.** Every backticked value/field/anchor appears verbatim in the data you were handed. Never synthesize a path, hash, or field name that "should" be there.
* **Attribution gate.** Never assert ownership/legitimacy of a domain/IP/binary unless the handed-off data confirms it.
* **Rule blurb ≠ evidence.** A detection naming a malware family is not evidence this event is that malware.
* **Reputation fidelity.** Any detection stat referenced in "Why safe" was actually handed to you — never invented.
* **Known-infra ban.** Trusted cloud infra (`*.protection.outlook.com`, `*.sharepoint.com`, MS/O365 relay IPs) never becomes a filter anchor.

## ELIGIBILITY GATE — decide before drafting
* **Orchestration Justification** when: rule class is not tunneling/C2/exfil/lateral-movement, ≥2 stable behavioral anchors exist, and the FP/TP boundary survives the same-entity test (same host/user repeating the behavior would still alert).
* **Manual Closure** when: <2 stable anchors, the only safe filter is too broad/verbose, or the same-entity test fails.
* Tunneling/C2/exfil/lateral-movement on a single event is never eligible for suppression. If this skill is invoked on one anyway, that's an upstream routing error — say so in the output rather than silently suppressing it.
* **Suppressions are quiet.** Neither artifact below generates a customer notice. If an analyst later decides to inform the customer, that's their manual call, not something this skill produces.

## ORCHESTRATION JUSTIFICATION FORMAT (+ KVP rows)
The KVP table is the deliverable the analyst applies directly — not a suggestion for someone else to translate. Do NOT emit scope/tier/deployment settings, array-field/Django templates, or TAPs.

```markdown
### Orchestration Justification
**Title:** [concise filter title naming the detection + benign pattern — e.g. `Notepad→Edge Workday login handoff (CS - Notepad spawning processes)`]
**Type:** [net-new filter / filter modification / feed-based suppression / auto-routed playbook / alert comment playbook / event hint / temp filter]
**Suppresses:** [ONE sentence — what this filter suppresses, by stable anchor, never a per-event ID]
**Why safe:** [why benign AND why a TP variant still alerts (different child/path/destination, or same host spawning shells/script hosts/LOLBins) — because the filter is anchored to this exact pattern. 1–3 sentences, no bullet sprawl.]

**Filter Logic (KVP):**
Field              Operator           Value
[2–4 rows, strongest anchor first; use actual CORR/sensor field identifiers where known, e.g. `ioc.iocTitle`, `InitiatingProcessFileName`, `FileName`, `ProcessCommandLine`]
```
* **Keep it to Title / Type / Suppresses / Why safe / KVP only** — no user-device dossier, no scope-fit, CORR, or residual/expiry as standalone lines.
* **Operators (CORR):** `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.
* **KVP rules:** 2–4 rows, strongest anchor first; minimum = ONE strong anchor + ONE qualifier. Never enumerate the whole roster. Use a version-independent path prefix with `Contains`; `Command Line Does not contain "[TP differentiator]"` keeps a TP variant alerting. NO volatile identifier as a field/value (incident#, GUID, PID, SID, timestamp, port, internal/DHCP IP, file size, `type=unknown`); no internal IP in a hostname field.

## MANUAL CLOSING FORMAT
```markdown
### Manually Closing
[2–4 sentences: what fired (`rule` + pattern), the specific benign evidence, and why no durable filter is safe — name the anchor/boundary problem (only volatile identifiers distinguish it / a viable filter would suppress TPs / would break on cmdline variation / same-host recurrence is the likely TP). CORR stated.]
[If recurrence frequent: one line — tuning request with sample volume.]
```

## OUTPUT
One line, then the matching fenced block, nothing else:
`Verdict: [confirmed/indicated/unconfirmed] · [Suppress/Manual Closure]`

## SELF-CHECK (silent, before emit)
1. Provenance diff on every anchor and field used.
2. Eligibility gate applied and decided autonomously — no query back to the analyst.
3. Orchestration block is Title/Type/Suppresses/Why-safe/KVP only; 2–4 KVP rows, strongest anchor first, no volatile identifier; Manual Closing used instead when anchors <2 or filter unsafe.
4. No proactive customer-notification language anywhere in the output.
5. Output = verdict line + fenced block only.
