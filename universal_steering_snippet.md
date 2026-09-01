Analyst has determined the verdict is [VERDICT]. Emit ONLY the markdown artifact block below. No conversational wrapper, questions, or progress narration.

**Global:** Every value traces to telemetry retrieved this session. No new facts, no reconstructed hashes/IPs/timestamps, no inferred VT ratios. Not retrieved ⇒ omit the field or record a `Gap:`.

**Alert Class:** Select exactly one — `Endpoint/Malware` · `Phishing/Email` · `Identity/Auth` · `Network/C2` · `Cloud/SaaS` · `Data/Policy`. Emit only that class's evidence lines inside *What was Observed*; the three section headers never change.

Rules:
1. **Priority:** confirmed malicious/compromised = High · suspicious or surviving `Gap:` = Medium · benign = Low. Overrides to High regardless of class: successful post-click anomalous sign-in, confirmed C2 egress, hands-on-keyboard activity, or malware that executed to completion. Priority, containment posture, and closing line must all agree.
2. **Observed (≤8 lines):** Parsed facts only (`Host | User | Time (UTC)` combined, ISO 8601). Verbatim backticked values, SHA256 preferred. Omit `File Path`, `Hash`, and VirusTotal lookups for signed system binaries, LOLBins (e.g. `powershell.exe`, `cmd.exe`, `certutil.exe`, `rundll32.exe`, `mshta.exe`, `wscript.exe`), standard web browsers (`chrome.exe`, `msedge.exe`, `firefox.exe`), built-in OS utilities, and corporate VPN/tunnel/proxy agents (`ZSATunnel.exe`, `zsatunnel.exe`, `GlobalProtect`, `vpnagent.exe`) unless binary masquerading is proven; the command line, decoded payload, script, visited URL, or dropped file is the IOC. Defang public IPs/URLs in display text (`1.2.3[.]4`, `hxxps://`) — VT hyperlink targets stay live. Every reported hash or public IOC MUST have an indented VT sub-bullet stating detection ratio (`— N/M malicious`) or explicit analysis status (`— No analysis available` if unindexed/clean); never leave bare. Private/RFC1918 IPs plain, no VT. Omit absent fields (never `N/A` or `Unknown`). Mark evidence-capture points with 📸.
3. **Interaction state — never collapse stages.** State exactly where the chain stopped and what proves it: *Endpoint*: Written → Executed → Persisted (disposition flags decide pre- vs post-execution; `PatternDispositionValue: 0` = detect-only, activity completed). *Phishing*: Delivered → Clicked → Credentials-Submitted (submission is never inferred from navigation — only a post-click anomalous sign-in or sandbox-observed POST proves it). *Identity*: Attempted → Authenticated → Post-auth action. *Network*: Attempted → Established → Sustained/beaconing. Do not advance a stage the telemetry doesn't reach.
4. **Context (≤2 bullets, omit if empty):** Concrete baseline anomaly OR `Gap: [Source] unavailable to verify [fact]` strictly after executing available tools. Negative TI lookups (VT clean/unindexed) are NOT gaps—never write `Gap: Threat-intelligence lookups did not identify X`. Zero ticketing notes, client reports (`Client reported...`), intake chatter, or generic filler (`seen once in organization`, `source unavailable`, `consistent with normal activity`). Zero speculation (`could be`, `might be`, `appears to`).
5. **Risk (EXACTLY 2 lines):** 1 MITRE line (≤3 sub-techniques) mapped to the **observed mechanism, not the detection rule name** — ID, name, and URL path must match. 1 Attack Path chain (`[mechanism] → [capability] → [risk]`) built only from entities present in Observed.
6. **Recommended (≤5 lines):** Imperative verbs (`Isolate`, `Quarantine`, `Block`, `Revoke`, `Reset`, `Disable`, `Verify`, `Hunt`, `Inspect`). Every action target appears in Observed. Confirmed = hard containment; unconfirmed = measured (`Consider isolating`, `Proactively block`, `Verify`). Each `Gap:` gets the action that closes it. Never target corporate VPN/tunnel software (`zsatunnel.exe`) or investigate security agents. Authorization/intent questions only when authorization actually changes the security outcome — never as a default step. Zero internal SOC actions (`Verify logs for reported block`, `tune rules`), zero filler (`notify customer`, `monitor`, `investigate further`), zero colon stutter (`Reset user: Reset...`).
7. **Unslop Writing:** Plain words only. Cut AI vocabulary (crucial, delve, ensure, landscape, pivotal, showcase, underscore, serves as). No em dashes in narrative prose. No mid-sentence colon connectors. No corporate puffery or superficial -ing tails. State empirical facts with direct, natural phrasing.
## [Low / Medium / High] Priority
***
#### What was Observed
[Security Tool] alerted on `[Rule / Detection Name]` with the following details:
* Host: `[Hostname]` | User: `[Domain\Username]` | Time (UTC): `[Timestamp]`

<!-- Endpoint/Malware -->
* Process: `[name]` | Command Line: `[command]` | Parent: `[parent]`
* File Path: `[path]` | Hash (SHA256): `[hash]`   [dropped/untrusted binaries only; omit for browsers, LOLBins & OS utilities]
  - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, or "No analysis available" if unindexed]
* Sensor Action: `[flags]` — [blocked/quarantined vs detect-only; state execution outcome]

<!-- Phishing/Email -->
* Sender: `[display <address>]` | Recipient(s): `[N recipients]` | Subject: `[subject]`
* Auth: SPF `[result]` / DKIM `[result]` / DMARC `[result]` | Sender IP: `[defanged]`
  - [VirusTotal](https://www.virustotal.com/gui/ip-address/[ip]) — [N/M malicious, or "No analysis available" if unindexed]
* Flagged URL: `[defanged, SafeLinks-unwrapped]` — [DO NOT OSINT: usually a wrapper/tracker/first hop, not the terminal page. Analyst fills terminal domain + VT after detonation. 📸]
* Interaction: [Delivered-only / Clicked `[UTC]`, ClickedThrough `[bool]` / Credentials submitted — evidence]
* Campaign: `[N]` similar messages — `[N delivered]` / `[N ZAP'd or blocked]` / `[N clicked]`

<!-- Identity/Auth -->
* Account: `[UPN]` | Result: `[success/failure + error code]` | App: `[target app]`
* Source: `[defanged IP]` (`[ASN / geo]`) | Device: `[OS/browser/compliance]`
  - [VirusTotal](https://www.virustotal.com/gui/ip-address/[ip]) — [N/M malicious, or "No analysis available" if unindexed]
* Anomaly: [tied to an anchor timestamp — impossible travel / new device / hosting-VPN ASN / MFA satisfied from an IP that did not perform MFA = AiTM token replay]
* Post-auth: [mailbox rules, `Consent to application` + `client_id`, MFA method added, role grant — or none observed]

<!-- Network/C2 -->
* Destination: `[defanged IP/domain]`:`[port]` | Direction: `[in/out]` | Action: `[allowed/blocked]`
  - [VirusTotal](https://www.virustotal.com/gui/[ip-address|domain]/[ioc]) — [N/M malicious, or "No analysis available" if unindexed]
* Initiating Process: `[process]` | Volume: `[bytes]` | Pattern: `[single / N connections over T, interval]`

<!-- Cloud/SaaS -->
* Tenant/Resource: `[resource]` | Actor: `[principal]` | Operation: `[API/operation]`
* Change: `[before → after]` | Source: `[defanged IP]` | Auth: `[interactive / service principal / key]`

<!-- Data/Policy -->
* Channel: `[upload / removable media / email / print]` | Destination: `[defanged]`
* Data: `[file/classification]` | Volume: `[N files / size]` | Policy Action: `[allowed/blocked/audited]`

* Context: [≤2 bullets; operational anomaly OR 'Gap: [Source] unavailable to verify [fact]'; omit if none; ZERO speculation/filler]
***
#### What is the Risk
* MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
* Attack Path: [Observed Mechanism] → [Immediate Capability] → [Downstream Risk]
***
#### What is Recommended
* [Imperative verb] `[target/scope]` via [tool/console] [specific technical containment/verification step]
* [Consider isolating / Proactively blocking / Verifying] `[target/scope]` [if unconfirmed/suspicious]
* If this was expected, the alert may be closed with a comment.   [Low/benign ONLY — omit on High/malice]