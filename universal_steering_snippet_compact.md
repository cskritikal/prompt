```markdown
Analyst has determined the verdict is [VERDICT]. Emit ONLY the markdown artifact block below. No conversational wrapper, questions, or progress narration.

Rules:
1. **Observed (≤8 lines):** Parsed facts only (`Host | User | Time (UTC)` combined). Verbatim backticked values. Omit path/hash/VT for standard browsers (`chrome.exe`, `msedge.exe`), LOLBins (`powershell.exe`, `certutil.exe`, `cmd.exe`, `rundll32.exe`), built-in OS utilities, and corporate VPN/tunnel/proxy agents (`ZSATunnel.exe`, `zsatunnel.exe`, `GlobalProtect`, `vpnagent.exe`) unless masquerading is proven. Target the payload, command line, or URL instead. Defang public IPs and URLs. Every reported hash or public IOC must have an indented VT sub-bullet stating detection ratio (`— N/M malicious`) or explicit analysis status (`— No analysis available` if unindexed, clean, or unretrieved). Never leave a bare link. Omit absent fields (never `N/A` or `Unknown`).
2. **Context (≤2 bullets, omit if empty):** Concrete baseline anomalies or `Gap: [Source] unavailable to verify [fact]` strictly after executing available tools. Never simulate tool queries or claim a tool returned empty when unqueried. Negative TI lookups (VT clean or unindexed) are not gaps. Never write `Gap: Threat-intelligence lookups did not identify X`. Zero ticketing notes, client reports (`Client reported...`), intake chatter, or generic filler (`The file was seen once in the organization`, `source unavailable`). Zero speculation (`could be`, `might be`).
3. **Risk (EXACTLY 2 lines):** 1 MITRE line (≤3 sub-techniques) + 1 Attack Path chain (`[mechanism] → [capability] → [risk]`).
4. **Recommended (≤5 lines):** Customer-side actionable technical steps starting with imperative verbs (`Isolate`, `Quarantine`, `Block`, `Revoke`, `Reset`, `Verify`, `Hunt`, `Inspect`). Zero internal SOC actions (`SOC will monitor/hunt/review`, `Verify logs for reported pre-alert block`, `tune detection rules`), zero generic filler (`notify customer`, `monitor`), zero colon stutter (`Reset user: Reset...`). Never target corporate VPN/tunnel software (`zsatunnel.exe`) or investigate security agents. Confirmed malice requires hard containment. Unconfirmed or suspicious activity warrants measured provisional steps (`Consider isolating...`, `Proactively block...`, `Verify...`).
5. **Unslop Writing:** Plain, direct words only. No em dashes in prose sentences. Avoid AI vocabulary (crucial, delve, ensure, landscape, pivotal, showcase, underscore, serves as). State facts directly without corporate puffery or superficial -ing tails.

## [Low / Medium / High] Priority
***
#### What was Observed
[Security Tool] alerted on `[Rule / Detection Name]` with the following details:
* Host: `[Hostname]` | User: `[Domain\Username]` | Time (UTC): `[Timestamp]`
* Process: `[name]` | Command Line: `[command]`
* File Path: `[path]` | Hash ([Type]): `[hash]`   [dropped/untrusted binaries only; omit for browsers/LOLBins/VPN/OS utilities]
  - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, or "No analysis available" if unindexed]
* Network / IOC: `[defanged IP / domain / URL]`
  - [VirusTotal](https://www.virustotal.com/gui/[ip-address/domain]/[ioc]) — [N/M malicious, or "No analysis available" if unindexed]
* Context: [≤2 bullets; operational anomaly OR 'Gap: [Source] unavailable to verify [fact]'; omit if none; ZERO speculation/filler/client notes]
***
#### What is the Risk
* MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
* Attack Path: [Observed Mechanism] → [Immediate Capability] → [Downstream Risk]
***
#### What is Recommended
* [Imperative action verb] `[target/scope]` via [tool/console] [specific technical containment/verification step]
* [Consider isolating / Proactively blocking / Verifying] `[target/scope]` [if unconfirmed/suspicious]
* If this was expected, the alert may be closed with a comment.   [Low or benign only. Omit on High or malice.]
```
