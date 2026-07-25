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
> Clean up the Escalation artifact:
> 1. Cap Context bullets to maximum 2 (only material gaps or contradictions).
> 2. Defang all public IPs/domains (`192[.]168[.]1[.]1`, `hxxps://bad[.]com`).
> 3. Remove VT links for RFC1918 private IPs, loopbacks, and trusted Microsoft cloud infrastructure (`*.outlook.com`, `*.sharepoint.com`).
> 4. Format VT links as: `[VirusTotal](https://www.virustotal.com/gui/search/[hash]) — N/M malicious`.
> ```

---

### 5.2 Remove Generic / Forbidden Recommendations
#formatting/recommendations #technical-actions

> [!warning] Trigger Condition
> The agent writes generic actions like "notify customer", "monitor", "investigate further", or "escalate per procedure".

> [!quote] Copyable Prompt Snippet
> ```markdown
> Rewrite the Recommendations section. FORBIDDEN phrases: "notify customer", "escalate per procedure", "monitor", "investigate further". Provide only concrete, technical containment/isolation steps grounded in the observed indicators (e.g., host isolation, credential reset, block IP on firewall).
> ```

---

### 5.3 Fix Orchestration Justification Format & Remove Dossier
#formatting/orchestration #kvp-table

> [!warning] Trigger Condition
> The agent includes user dossiers, scope-fit analysis, or extra commentary in Orchestration Justification.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format the Orchestration Justification strictly. Include ONLY:
> - **Title:** [concise filter title]
> - **Type:** [filter type]
> - **Suppresses:** [ONE sentence summary]
> - **Why safe:** [1-3 sentences covering benign rationale + why TP still alerts]
> - **Filter Logic (KVP):** Table with fields: Field, Operator, Value.
> Delete all user dossiers, scope fit, CORR history blocks, or residual risk lines.
> ```

---

### 5.4 Strip Volatile Identifiers from KVP Filter Rows
#formatting/kvp-logic #stable-anchors

> [!warning] Trigger Condition
> The agent uses PIDs, timestamps, incident IDs, or internal IPs in KVP filter logic.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Invalid KVP filter logic detected. NEVER use volatile identifiers (Incident ID, PID, SID, GUID, timestamp, port, internal/DHCP IP, file size). Use 2–4 stable anchors (e.g., `InitiatingProcessFileName`, `FileName`, `ProcessCommandLine`, `ioc.iocTitle`) with operators like `Match`, `Contains`, `In`, `Does not contain`.
> ```

---

### 5.5 Fallback to Manual Closing Block
#formatting/manual-close #fallback

> [!warning] Trigger Condition
> The agent attempts to create a filter when an alert should be closed.

> [!quote] Copyable Prompt Snippet
> ```markdown
> A durable filter is unsafe here. Switch from Orchestration Justification to a single `### Manually Closing` block (2-4 sentences detailing what fired, benign evidence, and why no durable filter is safe).
> ```

---

## 6. Output Line 1 Enforcement

### 6.1 Force Strict Line 1 Header & Zero Wrapper
#formatting/disposition #line1-enforcement

> [!warning] Trigger Condition
> The agent omits the required `DISPOSITION:` line or adds intro text.

> [!quote] Copyable Prompt Snippet
> ```markdown
> Format requirement failed. Line 1 of your response MUST be strictly in this format:
> `DISPOSITION: [verdict] · [confirmed/indicated/unconfirmed] · [Filter-Close/Low/Med/High] · ROUTE [1 Escalation / 2 Orchestration / 3 Manual Closure]`
> Immediately follow Line 1 with the single fenced artifact block. Nothing before line 1; nothing after the artifact.
> ```
