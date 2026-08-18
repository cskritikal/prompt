```markdown
Analyst has determined the verdict is [VERDICT] and needs the Final Triage Artifact - do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the artifact in a single markdown block with zero conversational wrapper.

Rules:
1. **Observed (≤8 lines):** Parsed facts only (combined `Host | User | Time (UTC)` line, backticks on values, defang all public IPs/domains/URLs with typed VT links `/gui/ip-address/`, `/gui/domain/`, `/gui/search/`; trusted MS infra / Windows binaries / known browser processes are not IOCs per se and do not need to be verified by hash unless masquerading is suspected). **Omit absent/irrelevant fields entirely (no `N/A`, `Unknown`, or empty rows)**.
2. **Context & Gaps (≤2 bullets, omit line entirely if none):** Concrete operational baseline anomaly only (e.g. host role mismatch, user baseline anomaly, delivery lure mechanism, or sensor telemetry contradiction). **ZERO speculative guessing or hedging** (strictly forbidden: `could be`, `might be`, `possibly related to`, `potential routine/administrative task`). **Tool Verification Mandatory:** You MUST actively use all available session tools and queries to retrieve decisive telemetry before asserting any gap. A `Gap: [Specific Source/Table] unavailable to verify [exact empirical fact]` entry is ONLY permitted after authoritative tool lookups have actually been executed and returned empty, or confirmed non-callable. Never record a lazy gap for callable sources (banned: generic `source unavailable`, `missing telemetry`, or `unable to confirm intent`).
3. **Risk (EXACTLY 2 lines):** One MITRE line (≤3 evidence-backed sub-techniques) + one Attack Path arrow chain (`[mechanism] → [capability] → [downstream risk]`).
4. **Recommended (≤5 lines):** Direct, proportional, and immediately actionable technical steps calibrated to threat certainty:
   - **Confirmed Malicious (High/Critical):** Direct, unconditional containment actions (`Isolate`, `Quarantine`, `Block`, `Purge`, `Reset`, `Revoke`, `Terminate`).
   - **Suspicious / Unconfirmed (Medium — Not Verifiably Malicious):** Measured containment or verification-first actions (e.g. `Consider isolating host [host] pending triage`, `Proactively block destination IP [IP]`, `Verify with user/admin whether [activity] was authorized`, `Inspect endpoint for [persistence artifact]`).
   - **Low / Policy / Benign-Leaning:** Targeted verification & policy review (`Verify software authorization for [tool]`, `If this was expected, the alert may be closed with a comment.`).
   - **Form Discipline:** Direct action statements; zero tautological label-colon stutter (no `Reset user: Reset...`), zero lazy meta-references (no `"the identified senders"`), zero generic filler (`notify customer`, `monitor`, `investigate further`, `escalate per procedure`).
5. **Forbidden Language:** Never output non-canonical verdicts (`Inconclusive`, `Evidence Gap`, `Unknown`), placeholder junk (`N/A`, `None`, `TBD`, `Unknown`), speculative context/hedging (`could be related to`, `might be`, `appears to be`, `possibly`, `potential administrative/routine activity`), unverified/generic gaps (`source unavailable`, `missing telemetry`, `unable to confirm intent`, `more logs needed`), generic recommendations (`notify customer`, `monitor`, `investigate further`, `escalate per procedure`), lazy meta-IOC references (`"the identified senders/IPs"`), or internal handling/ROE commentary.

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
* Context: [≤2 total; concrete host/user operational baseline anomaly OR 'Gap: [Specific Source] unavailable to verify [exact fact]'; omit line entirely if neither applies; ZERO speculation/hedging/filler]
***
#### What is the Risk
* MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
* Attack Path: [Observed Mechanism] → [Immediate Capability] → [Downstream Risk]
***
#### What is Recommended
* [Isolate / Quarantine / Block / Purge / Reset / Revoke / Verify / Hunt] `[target / scope]` via [tool / console] [immediate technical action]
* [Isolate / Quarantine / Block / Purge / Reset / Revoke / Verify / Hunt] `[target / scope]` via [tool / console] [containment / eradication action]
* If this was expected, the alert may be closed with a comment.   [benign/expected/intent-dependent ONLY — omit on High and other instances where it would make no sense for the activity to be expected.]
```