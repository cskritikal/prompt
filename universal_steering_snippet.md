```markdown
Analyst has determined the verdict and needs the Final Triage Artifact - do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY Line 1 followed immediately by the 3-section artifact with zero conversational wrapper.

Line 1 (all 4 fields strictly canonical):
`DISPOSITION: [Malicious / Suspicious / Benign] · [Confirmed / Indicated / Unconfirmed] · [Filter-Close / Low / Medium / High] · ROUTE [1 Escalation / 2 Orchestration / 3 Manual Closure]`
*(Forbidden verdicts: Inconclusive, Evidence Gap, Unknown; telemetry gaps belong strictly inside Context as `[gap: ...]`).*

Rules:
1. **Observed (≤8 lines):** Parsed facts only (combined `Host | User | Time (UTC)` line, backticks on values, defang all public IPs/domains/URLs with typed VT links `/gui/ip-address/`, `/gui/domain/`, `/gui/search/`; trusted MS infra never an IOC). **Omit absent/irrelevant fields entirely (no `N/A`, `Unknown`, or empty rows)**.
2. **Context (≤2 bullets):** 1-sentence plausible explanation for unexplained activity or single-line `[gap: ...]` (never internal ROE/handling mechanics).
3. **Risk (EXACTLY 2 lines):** One MITRE line (≤3 evidence-backed sub-techniques) + one Attack Path arrow chain (`[mechanism] → [capability] → [downstream risk]`).
4. **Recommended (≤5 lines):** Imperative verbs only (`Isolate`, `Quarantine`, `Block`, `Reset`, `Revoke`, `Verify`, `Hunt`). Proportional, sensible, and immediately actionable by client—never generic filler (`notify customer`, `monitor`, `investigate further`, `escalate per procedure`).

## [Low / Medium / High] Priority
***
#### What was Observed
[Security Tool] alerted on `[Rule / Detection Name]` with the following details:
* Host: `[Hostname]` | User: `[Domain\Username]` | Time (UTC): `[Timestamp]`
* Process: `[name]` | Command Line: `[command]`
* File Path: `[path]` | Hash ([Type]): `[hash]`
  - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, only if lookup produced it; bare link if not indexed]
* Network / IOC: `[defanged IP / domain / URL]`
  - [VirusTotal](https://www.virustotal.com/gui/[ip-address/domain]/[ioc]) — [N/M malicious, only if lookup produced it]
* Context: [≤2 total; 1-sentence plausible explanation or single-line [gap: ...]]
***
#### What is the Risk
* MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
* Attack Path: [Observed Mechanism] → [Immediate Capability] → [Downstream Risk]
***
#### What is Recommended
* [Isolate / Quarantine / Block / Reset / Revoke / Verify / Hunt] `[target / scope]`: [immediate technical action]
* [Isolate / Quarantine / Block / Reset / Revoke / Verify / Hunt] `[target / scope]`: [containment / eradication action]
* If this was expected, the alert may be closed with a comment.   [benign/expected/intent-dependent ONLY — omit on High]
```
