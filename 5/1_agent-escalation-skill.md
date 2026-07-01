# AGENT-ESCALATION-SKILL — Customer-Facing Escalation Format (Phase 4 output only)

By the time this skill runs, Phases 1–3 (Parse → Enrich → ROE Scan → Initial Briefing → Triage Discussion → Pre-Resolution Safeguard) have already executed and the analyst has already confirmed Escalation as the resolution. This skill does not re-decide severity, does not re-run enrichment, and does not decide suppression eligibility. Its only job is to render the customer-facing artifact from what's already been established, and to render it accurately.

## Inputs this skill assumes are already available
* Alert facts: host/user/process/hash/path/cmdline/network artifacts, as already parsed.
* Enrichment findings already gathered upstream: any IOC reputation results, source-host role, identity/device context, CORR history result.
* Assigned severity/priority (Low/Medium/High) and verdict confidence (confirmed/indicated/unconfirmed) from Triage Discussion.

If any of the above is genuinely missing when this skill runs, treat it as an absent field per Grounding Laws — never re-derive it from memory, never guess to fill a gap.

## Ground before you draft
Match every field you're about to write against the handed-off data first, silently. A value written before you've confirmed it's actually there is how placeholders get into a customer-facing document.

## GROUNDING LAWS (accuracy kernel — supersedes every other section in this file on conflict; a violation is an output failure)
* **Extraction-only.** Every backticked value appears verbatim in the data you were handed. Never synthesize/normalize/"complete" hostnames, hashes, paths, IPs, or tool names. Absent value → absent line; no N/A or placeholders.
* **Attribution gate.** Never assert ownership/legitimacy/vendor identity of a domain/IP/binary unless the handed-off data confirms it. Unconfirmed = describe observed behavior only.
* **Rule blurb ≠ evidence.** A detection naming a malware family explains why the rule exists; it is not evidence this event is that malware. Judge telemetry.
* **Reputation fidelity.** Render `— N/M malicious` ONLY if that exact ratio was actually produced upstream and handed to you. No ratio handed to you → bare VT link, no invented figure, never presented as clean.
* **Known-infra ban.** Microsoft O365/Exchange relay IPs, `*.protection.outlook.com`, `*.sharepoint.com`, and equivalent trusted cloud infra never occupy IOC lines or receive VT links — one Context bullet if material as lure/relay.

## FORMAT
Output exactly the fenced block below and nothing else around it.
* **Density:** parsed facts only; one fact/value per bullet; delete any bullet that is label-only, self-evident, restates the rule name, or wouldn't change the reader's next step. Omit absent fields. Same-type sets >5 → one bullet: count + ≤5 representative values revealing the pattern. Backticks on discrete values only; truncate >100 chars with `...`. Decoded line only when real Base64/hex is present.
* **Context bullets:** hard cap TWO, norm zero/one — only a telemetry contradiction/containment gap, a material interpretation caveat, or the single most decision-relevant unshown fact.
* **IOC/VT:** every public IP/domain/URL host/hash gets a VT sub-bullet, governed entirely by Reputation Fidelity above. Defang public IPs/domains in EVERY line (`192[.]168[.]1[.]1`, `hxxps://bad[.]com`). IP→`/gui/ip-address/{ip}`, domain→`/gui/domain/{domain}`, hash→`/gui/search/{hash}` (SHA256>SHA1>MD5). Full URL: defang whole URL, VT-link the host. Tokenized/encoded subdomain: show full FQDN defanged, VT-link the registrable domain. RFC1918/loopback/link-local: plain text, no defang, no VT. Public resolvers as destinations aren't IOCs. Trusted signed MS/OS binary or LOLBin with no masquerading: behavior is the IOC, not the binary — omit its hash/canonical path/bare cmdline.
* **Risk:** alert-specific, not boilerplate. MITRE = observed mechanisms only, most specific sub-technique, cap 2–3 evidence-backed, no intent technique on benign-leaning. Attack Path = `[observed mechanism] → [immediate capability] → [concrete downstream consequence]`; unobserved next leg → state the gap, never hand-wave.
* **Recommendations:** technical, customer-actionable, grounded in observed artifacts; containment specific (host + scope + follow-on). FORBIDDEN: `notify customer`, `escalate per procedure`, `monitor`, `investigate further`. High → contain/isolate/eradicate/hunt; Medium → verify + proactive containment; Low → verify.
* **Priority:** the triage-assigned Low / Medium / High.

```markdown
## [Low / Medium / High] Priority
***
#### What was Observed
[Security Tool] alerted on `[Rule / Detection Name]` with the following details:
* Host: `[Hostname]` | User: `[Domain\Username]` | Time (UTC): `[Timestamp]`
* Process: `[name]`
* File Path: `[path]`
* Hash ([Type]): `[hash]`
  - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, only if enrichment produced it]
* Command Line: `[command]`
  * Decoded: `[Base64/encoded originals — only if encoding present]`
* Parent Process: `[name]` | `[cmdline]`
* Network / IOC: [defanged public IP/domain/URL OR plain private]
  - [VirusTotal — typed per IOC rules; stats only if produced]
* Context: [≤2 total, often zero.]
***
#### What is the Risk
* MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
* Attack Path: [Observed mechanism] → [Immediate capability] → [Downstream risk]
***
#### What is Recommended
* [Action Label]: [Step]
* [Action Label]: [Step]
* If this was expected, the alert may be closed with a comment.   [benign/expected/intent-dependent ONLY — omit on High and on confirmed/strongly-suspicious]
```

## OUTPUT
One line, then the fenced block, nothing else:
`Verdict: [confirmed/indicated/unconfirmed] · [Low/Medium/High]`

## SELF-CHECK (silent, before emit)
1. Provenance diff: every backticked token matches the handed-off data verbatim — fix or cut before anything else.
2. Every `N/M malicious` figure was actually handed to you; anything not handed to you is a bare link, never a guessed ratio, never shown as clean.
3. ≤2 Context bullets; absent fields omitted; sets >5 consolidated; correct typed VT per public indicator; RFC1918 untouched; trusted-binary suppression applied.
4. MITRE ≤3, evidence-backed; no intent technique on benign-leaning.
5. Output = verdict line + fenced block only. No briefing, no options, no confirmation, no trailing question.
