# TRIAGE REFERENCE — Phases 1–3 judgment logic (Enrich / ROE Scan / Triage Discussion)

Not a skill body — there's no confirmed steerable slot for Phases 1–3 in the CS Agent as of the last check (harness-owned, not overridable by a static paste). This is the analytical judgment that used to live in the front half of the ONE-SHOT prompt. Use it manually while driving Triage Discussion, or test-paste it into that phase if you find an actual slot — see README for how to re-check.

## Tool-reality & enrichment discipline
* **Tool-reality check first.** Before any lookup, know which sources are actually callable tools in this session — not which ones are usually available. Anything not actually callable is `[gap: source unavailable]` immediately; never simulate its output. Anything callable that returns nothing is a failed lookup, not a clean verdict.
* **Self-serve enrichment.** Resolve every open question with whatever is actually callable: CORR history for this detection type, EDR/SIEM consoles (MDE/KQL, Falcon, S1, XDR, Sentinel, Sumo, Panorama, Exchange audit), Entra ID sign-in/audit logs + Intune device state for any identity/sign-in alert, web search, VirusTotal (per Reputation Fidelity below), WHOIS/passive-DNS. Bound the pass: stop when questions are answered, sources exhausted, or marginal queries stop moving the disposition. Never loop.
* **Enrich before you route — escalation is never a shortcut for unfinished analysis.** The alert name is a hypothesis, not the verdict. Pull the context the alert itself can't show (device management/compliance, identity/session state, source-host role, IOC reputation, CORR) before routing on it. An escalation produced before the obvious decisive lookup was attempted is an output failure.
* **Decide, document, proceed.** Where enrichment can't fully close a question, decide on the most defensible reading plus the rule-class floor, and record the residual as a gap. Uncertainty is documented, never escalated to a human. When the decisive unknown is identity/device context you genuinely could not retrieve and there's no active malice indicator, the activity leans benign — route toward suppress/close, not a Medium escalation.
* If no alert content was provided at all, say so and stop — that's the only stop.

## GROUNDING LAWS (accuracy kernel — a violation during enrichment produces a bad handoff downstream)
* **Extraction-only.** Every value you carry forward appears verbatim in the alert or a recorded lookup. Never synthesize/normalize/"complete" hostnames, hashes, paths, IPs, or tool names.
* **Attribution gate.** Never assert ownership/legitimacy/vendor identity of a domain/IP/binary unless evidence confirms it.
* **Rule blurb ≠ evidence.** A detection naming a malware family explains why the rule exists; it is not evidence this event is that malware. Judge telemetry.
* **Reputation fidelity.** Detection stats come only from a lookup you actually ran and can point to. VT's GUI is JS-rendered and usually unfetchable by search/fetch; a hash search returning nothing is expected, not a failure. Report `N/M malicious` only when a retrieved source states it; otherwise hand off a bare link — never a guessed ratio, never treated as clean.
* **Known-infra ban.** Microsoft O365/Exchange relay IPs, `*.protection.outlook.com`, `*.sharepoint.com`, and equivalent trusted cloud infra are never IOCs.

## SENSOR & RULE-CLASS SEMANTICS
* **Sensor honesty.** Never treat absence of telemetry a sensor cannot produce (a firewall/DNS alert can't see execution) as benign evidence — state the visibility limit as a caveat and close it via EDR/DNS-server logs during enrichment.
* **Tunneling/encoded-DNS class.** Encoded/numeric/high-entropy subdomain labels ARE a candidate exfil/C2 channel; never write "no exfil channel." Honest state: "single query observed; volume, recurrence, originating process unverified."
* **Canary/token class.** A tripwire fired by design; benign reading requires identifying the token owner/source, not absence of follow-on.
* **Source-behavior alerts** (enumeration/scanning/lateral probing): establish the source host's ROLE (scanner / RMM / jump box / workstation) via device info, installed software, logged-on users, naming convention, alert history. Role decides disposition more than the behavior.
* **Identity / sign-in class** (impossible travel, atypical/distant consecutive sign-ins, anonymous-IP/risky sign-in, new-country): the alert is a CANDIDATE anomaly, never a confirmed compromise. Pull: device managed + compliance state (Intune/Entra), MFA/strong-auth outcome, Conditional Access result, named/trusted location, Entra sign-in risk level, session/token reuse. A successful sign-in from a managed, compliant device with MFA satisfied and Conditional Access passed, no token/session anomaly, is benign expected behavior → close or suppress, not escalate. Escalate only when enrichment leaves real suspicion: MFA failed/absent, sign-in risk high, unmanaged/non-compliant device, impossible-velocity with successful auth, or AiTM session-token indicators (managed+compliant alone does NOT clear a stolen-cookie/AiTM pattern).

## HISTORY & BENIGN SWEEP
* **CORR**, priority order: Rule+Host/User/Process → Hash+Host/User → Path/Cmdline+Host → Network IOC+Host → Rule Name. Prior FP = tuning precedent (toward benign); prior TP = heightened scrutiny; first-time = caution. No access → `No CORR history available`.
* **Benign sweep:** scanners (Tenable/Qualys/Nessus) · IT/RMM (ConnectWise/SCCM) · Dev/IDE (Cursor/VS Code/pip) · service accounts · scheduled tasks · safe parent chains · benign naming · default-account wordlists (`adm`,`manager`,`USERID`,`ibm`,`DBA`,`help` = vuln-scanner default-credential check, confirmed by source-host role).

## PRIORITY — set AFTER enrichment, never inherited from the rule
* **Filter / Close** — benign; the client doesn't need to see this. No customer escalation. (Whether this becomes a suppression or a manual closure is `agent-orchestration-skill`'s call at Phase 4, not this one.)
* **Low** — real but non-urgent, worth a customer note: expected tooling or a policy violation without a malicious indicator the customer should still know about. (Benign the customer needn't see is Filter/Close, not Low.)
* **Medium** — genuine unresolved suspicion that SURVIVED enrichment: suspicious-unconfirmed, LOLBin abuse without payload, failed attack with live IOCs. Not for an anomalous-sounding alert whose decisive context you simply haven't pulled yet.
* **High / Critical** — confirmed or strongly-indicated ongoing malicious activity: C2, hands-on keyboard, lateral movement, exfil, ransomware, confirmed credential/session compromise.
* **Floor:** a novel (no CORR precedent) detection in a tunneling/C2/exfil rule class is Medium minimum — Low requires documented FP precedent or a verified benign origin.
* **Re-derive, don't inherit.** Ambiguity that leans benign with no malice indicator is Filter/Close or Low — never Medium by default; Medium demands suspicion that survived enrichment.

## ROUTE — the escalate-vs-not decision
* **Escalate** when: confirmed/strongly malicious, suspicious-unconfirmed, OR unverifiable WITH an active malice/suspicion indicator. Also any tunneling/C2/exfil/lateral-movement detection not shown verifiably benign (unsafe to suppress). Priority = severity. No suppression.
* **Don't escalate** when: benign, or benign-leaning with no active malice indicator. Hand off to `agent-orchestration-skill`, which decides suppress vs. manual-close on its own using the anchor/same-entity test.
* **Suppressions are quiet.** A benign or benign-leaning alert with no malice indicator is suppressed or closed WITHOUT escalating to the customer — the vast majority of these need no customer visibility at all. Don't generate a "we proactively suppressed this" customer notice. Decide benign-vs-suspicious yourself from the evidence; never ask the customer.
