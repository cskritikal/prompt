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
> Do not ask questions, present options, or seek confirmation. Do not include progress narration or conversational intro/outro text. Ingest the alert, resolve open questions autonomously using available tools, and emit ONLY the disposition header and finished artifact block.
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
> Format the Escalation artifact strictly per Megaprompt standards:
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
>   - [VirusTotal](https://www.virustotal.com/gui/search/[hash]) — [N/M malicious, only if a lookup produced it]
> * Command Line: `[command]`
>   * Decoded: `[only if real Base64/hex present]`
> * Parent Process: `[name]` | `[cmdline]`
> * Network / IOC: [defanged public IP/domain/URL OR plain private]
>   - [VirusTotal — typed per IOC; stats only if produced]
> * Context: [≤2 total; if unexplained activity exists, include a 1-sentence description of what it COULD be related to.]
> ***
> #### What is the Risk
> * MITRE ATT&CK: [Tactic] — [[T####.###](https://attack.mitre.org/techniques/T####/###/)] [Name]
> * Attack Path: [Observed mechanism] → [Immediate capability] → [Downstream risk]
> ***
> #### What is Recommended
> * [Imperative verb] [scope]: [step]
> * [Imperative verb] [scope]: [step]
> ```
> Line Budgets: Observed ≤8 fact lines; Risk EXACTLY 2 lines; Recommended ≤5 imperative lines. Defang all public IPs/domains (`192[.]168[.]1[.]1`, `hxxps://bad[.]com`). RFC1918 private IPs left plain.
> ```

> [!tip] Full per-class field set
> This snippet *cleans* an escalation. For the compact per-class field set — what to put in **What was Observed / What is the Risk / What is Recommended** by alert class — see [[#7. Escalation Artifact Formats (Per-Class)|Section 7]].

---

### 5.2 Remove Generic / Forbidden Recommendations
#formatting/recommendations #technical-actions

> [!warning] Trigger Condition
> The agent writes generic actions like "notify customer", "monitor", "investigate further", or "escalate per procedure".

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format Recommendations strictly per Megaprompt standards:
> ```markdown
> #### What is Recommended
> * Isolate host `[Hostname]` via EDR console
> * Revoke all active session tokens for user `[User]`
> * Block IP `[defanged IP]` on edge firewall
> ```
> 1. Hard budget: ≤5 lines total.
> 2. Each line MUST start with an imperative verb (`Isolate`, `Quarantine`, `Verify`, `Hunt`, `Block`, `Reset`).
> 3. FORBIDDEN generic phrases: "notify customer", "escalate per procedure", "monitor", "investigate further".
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
> The agent omits the required `DISPOSITION:` line or adds intro text.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Megaprompt Output Discipline failed.
> Line 1 MUST be strictly in this format:
> `DISPOSITION: [verdict] · [confirmed/indicated/unconfirmed] · [Filter-Close/Low/Med/High] · ROUTE [1 Escalation / 2 Orchestration / 3 Manual Closure]`
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
> **Format & Grounding Preamble:** Output strictly the 3-section escalation block below under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** is ≤8 fact lines of parsed values only (combined `Host | User | Time (UTC)` line, omit absent fields without `N/A`, backticks on discrete values, defang all public IPs/domains/URLs with typed VirusTotal sub-bullets where stats reflect only verified lookups; trusted MS infra never an IOC line) with **Context** capped at ≤2 bullets (1-sentence plausible explanation for unexplained activity or telemetry gap; never internal ROE/handling mechanics). **What is the Risk** must be EXACTLY two lines: one MITRE ATT&CK line (cap 2–3 evidence-backed sub-techniques) and one Attack Path arrow chain (`[mechanism] → [capability] → [downstream risk]`). **What is Recommended** must be ≤5 lines total, each starting with an imperative action verb (`Isolate`, `Quarantine`, `Block`, `Reset`, `Verify`, `Hunt`) with specific containment scope—never generic filler like "notify customer", "escalate per procedure", "monitor", or "investigate further".
> ```

---

### 7.2 Malware / Endpoint Execution Escalation
#rule-class/malware #escalation-format

> [!warning] Trigger Condition
> The agent escalates a dropper / LOLBin payload / commodity-malware execution with generic or over-long observed fields.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Malware / Endpoint):** Output strictly the 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** ≤8 fact lines of parsed values (Host | User | Time (UTC); Process `name`; Parent `name` | `cmdline`; Command Line + `Decoded:` if present; File Path; Hash + typed VT; defanged C2 / network IOC + typed VT; Persistence if present; Context ≤2 bullets; omit hash/path of signed OS interpreters; omit absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (≤3 observed sub-techniques) and Attack Path (`[execution mechanism] → [capability gained] → [C2 / ransomware / lateral movement]`). **What is Recommended** ≤5 imperative lines (`Isolate`, `Quarantine`, `Remove`, `Reset`, `Block`)—never generic filler ("notify customer", "monitor").
> ```

---

### 7.3 Phishing / Email Escalation
#rule-class/phishing #escalation-format

> [!warning] Trigger Condition
> The agent escalates a phishing / impersonation / BEC alert on delivery status or message appearance, or with generic fields.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Phishing / Email):** Output strictly the 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** ≤8 fact lines of parsed email telemetry (Recipient(s) | Time (UTC); Sender; raw `SPF/DKIM/DMARC` auth result; defanged Originating IP + authorized-for-domain check + typed VT; Subject; defanged URL(s) + domain VT; Attachment name + Hash + typed VT; Delivery/ZAP status + user interaction; Context ≤2 bullets; trusted MS relay infra never an IOC line; omit absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (Phishing `T1566.x` + credential access) and Attack Path (`[delivery + auth result] → [user action] → [credential / session compromise]`). **What is Recommended** ≤5 imperative lines (`Purge`, `Block`, `Reset`, `Revoke`, `Hunt`)—never generic filler ("notify customer", "monitor").
> ```

---

### 7.4 Identity / Sign-in Escalation (AiTM / Token Theft)
#rule-class/identity #escalation-format

> [!warning] Trigger Condition
> The agent escalates a sign-in / identity alert — valid only on a named trigger actually met (see 3.1), never on "couldn't confirm benign".

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Identity / Sign-in):** Output strictly the 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper (escalate only on verified triggers: MFA failed/absent, Entra risk `High`, unmanaged/non-compliant device, impossible travel, or AiTM/token replay). Maintain a hard budget: **What was Observed** ≤8 fact lines of parsed identity telemetry (User (UPN) | Time (UTC); sign-in result; defanged Source IP + ASN + location + typed VT; device managed + compliant state; MFA and Conditional Access status; Entra sign-in risk level; token anomaly signal; post-auth actions; Context ≤2 bullets; omit absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (Valid Accounts / Session Cookie Theft) and Attack Path (`[auth outcome + risk] → [session / token control] → [inbox rule / OAuth persistence / lateral]`). **What is Recommended** ≤5 imperative lines (`Revoke`, `Reset`, `Require`, `Inspect`, `Remove`)—never generic filler ("notify customer", "monitor").
> ```

---

### 7.5 Network C2 / Tunneling Escalation
#rule-class/network #escalation-format

> [!warning] Trigger Condition
> The agent escalates a C2 / tunneling / exfil detection (unsafe-to-suppress class) with generic fields or a "no exfil channel" claim.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Network C2 / Tunneling):** Output strictly the 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper (unsafe-to-suppress class; floor is Medium minimum absent documented FP precedent). Maintain a hard budget: **What was Observed** ≤8 fact lines of parsed network telemetry (Host | Time (UTC); originating process `name` | `cmdline`; defanged destination FQDN/IP + typed VT on registrable domain; protocol; encoded subdomain sample; volume + beacon metrics; Context ≤2 bullets; omit absent fields without `N/A`; never write "no exfil channel"). **What is the Risk** EXACTLY 2 lines: MITRE (DNS / C2 protocols) and Attack Path (`[process + channel] → [C2 / tunnel beacon] → [exfil / remote control]`). **What is Recommended** ≤5 imperative lines (`Block`, `Isolate`, `Terminate`, `Hunt`)—never generic filler ("notify customer", "monitor").
> ```

---

### 7.6 Recon / Scanner Escalation (Unauthorized Only)
#rule-class/recon #escalation-format

> [!warning] Trigger Condition
> The agent escalates scan / enumeration / lateral behavior — valid only AFTER host role (see 3.3) confirms the source is NOT a sanctioned scanner / RMM / jump box.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Recon / Scanner):** Confirm host role first per 3.3 (if sanctioned scanner/RMM, suppress/close). If unauthorized workstation, output strictly the 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper. Maintain a hard budget: **What was Observed** ≤8 fact lines (Source host | User | Time (UTC) + confirmed workstation role; Targets enumerated; Protocol; Default-credential wordlist if present; Fan-out velocity; Auth success; Context ≤2 bullets; omit absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (Discovery / Brute Force) and Attack Path (`[unauthorized enumeration] → [target mapped] → [lateral movement / initial access]`). **What is Recommended** ≤5 imperative lines (`Isolate`, `Reset`, `Block`, `Hunt`)—never generic filler ("notify customer", "monitor").
> ```

---

### 7.7 Generic / Unclassified Escalation Format (Fallback)
#rule-class/generic #escalation-format #fallback-template

> [!warning] Trigger Condition
> The agent escalates an alert from an uncovered or custom alert class (e.g. cloud IAM, DLP, web application, custom SIEM) without using a structured field layout or by inventing an invalid per-class structure.

> [!quote] Copyable Prompt Snippet
> ```markdown
> **Format & Grounding Preamble (Generic / Unclassified):** Output strictly the 3-section escalation block under `## [Low / Medium / High] Priority` with no conversational wrapper for uncovered alert classes. Maintain a hard budget: **What was Observed** ≤8 fact lines of parsed telemetry (Anchor Entity | Time (UTC); Activity / Event; Primary Subject; Target / Resource; Key Parameters / payloads; defanged Network/Hash IOC + typed VT; Anomaly signal; Context ≤2 bullets; omit absent fields without `N/A`). **What is the Risk** EXACTLY 2 lines: MITRE (≤3 observed sub-techniques) and Attack Path (`[observed action / anomaly] → [immediate capability] → [downstream impact]`). **What is Recommended** ≤5 imperative lines (`Isolate`, `Revoke`, `Block`, `Quarantine`, `Reset`, `Hunt`)—never generic filler ("notify customer", "monitor").
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
> The agent emits an artifact without the mandatory Line 1 four-field disposition header, or includes intro narration / confirmation questions before Line 1.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Line 1, all four fields REQUIRED — never just the route:
> `DISPOSITION: [verdict] · [confirmed/indicated/unconfirmed] · [Filter-Close/Low/Med/High] · ROUTE [1 Escalation / 2 Orchestration / 3 Manual Closure]`
> 
> A disposition line missing verdict, confidence, priority, or route is an output failure.
> Immediately follow Line 1 with the single artifact for the route, exactly per the formats — ONLY the sections that appear in that format. Reasoning stays silent. Nothing before line 1; nothing after the artifact.
> ```

