# SOC Agent Steering Snippets (Post-Output Corrections)

Modular prompt snippets for correcting LLM-powered SOC agents during alert triage, log enrichment, threat routing, and artifact generation.

These snippets function as targeted follow-up prompts when an agent violates operational rules, hallucinates telemetry, or miscalibrates priority.

The base system prompt they correct is [`soc_one_shot_prompt.md`](soc_one_shot_prompt.md) — the zero-touch triage → route → artifact controller that defines the standard escalation block and line budgets referenced by Section 7.

---

## Overview

LLM-based SOC triage agents exhibit recurring failure modes:
- Priority Inflation: Assigning High or Critical priority based on alert titles rather than post-enrichment evidence or mitigation state.
- Unnecessary Human Pauses: Stopping to ask questions, present options, or narrate progress.
- Absence-Based Hallucinations: Assuming missing indicators or clean VirusTotal lookups constitute proof of benign activity, or inserting N/A placeholders.
- Generic Advice: Emitting non-actionable recommendations such as "notify customer", "monitor", or "investigate further".

Injecting steering snippets in a follow-up turn forces immediate re-evaluation and output correction without requiring upfront prompt redesign.

---

## Snippet Index

All snippets are located in [`steering_snippets.md`](file:///c:/Users/ConnorSmith/Downloads/prompt/steering_snippets.md) and categorized into six operational areas:

### 1. Operating Rules & Workflow Steering
- **1.1 Stop Narrating, Asking Questions, or Seeking Confirmation**: Removes introductory text, progress narration, and confirmation stops; forces autonomous triage.
- **1.2 Force Gap Marker on Missing Telemetry**: Prevents execution halts when SIEM/EDR tools fail; mandates `[gap: source unavailable]` markers.
- **1.3 Self-Serve Enrichment Before Escalating**: Prevents escalation based solely on alert titles; requires log and identity lookups first.
- **1.4 Defer to Analyst Override (No Re-Arguing)**: Forces the agent to accept analyst verdict overrides without defending prior outputs.
- **1.5 Exhaustive Deep Investigation & Zero Non-Data**: Mandates active multi-tool querying; strictly bans hallucinations, simulated query results, and speculative non-data. Also available as standalone [`deep_investigation_steering_snippet.md`](file:///c:/Users/ConnorSmith/Downloads/prompt/deep_investigation_steering_snippet.md).

### 2. Grounding & Evidence Integrity
- **2.1 Verbatim Extraction Only**: Enforces verbatim log extraction; prohibits normalization, synthesized hostnames, and N/A placeholders.
- **2.2 Rule Blurb vs. Actual Telemetry**: Distinguishes detection rule descriptions from observed process/network execution telemetry.
- **2.3 Strict VirusTotal & Reputation Fidelity**: Restricts reputation metrics strictly to lookups executed in session; prohibits invented detection ratios.
- **2.4 Absence of Malice Is Not Proof of Benign**: Corrects reasoning that treats clean VirusTotal results or successful email delivery as proof of benign intent.
- **2.5 Force Concrete Evidence for Malicious Verdicts**: Demands explicit observed telemetry (command line, decoded payload, auth failure) rather than alert names.
- **2.6 Force Positive Proof for Benign Verdicts**: Requires positive verified anchors (authorized IP, signed binary, Intune compliance) for benign closes.
- **2.7 No Fabricated Tool Execution / Honest Tool Auditing**: Prohibits simulating tool execution or claiming unqueried tools returned empty telemetry.

### 3. Class-Specific Rule Corrections
- **3.1 Identity / Sign-in Class Correction**: Corrects false escalations on compliant, managed devices with MFA satisfied.
- **3.2 Tunneling / Encoded-DNS Class Correction**: Prevents "no exfil channel" claims on encoded subdomains; establishes a Medium priority floor.
- **3.3 Source-Behavior & Scanner Role Check**: Mandates host role verification (vulnerability scanner, RMM tool, jump box) before flagging enumeration.
- **3.4 LOLBin, Browser & Interpreter vs. Payload IOCs**: Removes reputation, hash, and file path lookups for signed system binaries (`powershell.exe`, `cmd.exe`), standard browsers (`chrome.exe`, `msedge.exe`), and LOLBins; targets payload command lines and URLs.
- **3.5 Phishing & Sender Auth Decisive Artifact Check**: Enforces sender authentication (SPF/DKIM/DMARC) and originating IP checks over delivery status.

### 4. Priority & Route Selection
- **4.1 Re-derive Priority Post-Enrichment**: Calibrates priority based on required response action (Filter/Close, Low, Medium, High/Critical) post-enrichment.
- **4.2 Suppress Customer Notification on Internal Closure**: Prevents customer notifications for internal suppressions and closures.
- **4.3 Priority Sanity Check & Override (Inflated Severity)**: Overrides High/Critical priority on pre-contained, quarantined, or isolated events.
- **4.4 Durable Filter (Suppress) vs. Manual Closure Selection**: Defines criteria for Route 2 (Suppress with stable KVP anchors) versus Route 3 (Manual Closure for volatile one-offs).

### 5. Artifact Formatting Corrections
- **5.1 Fix Escalation Format**: Enforces context caps (maximum 2 bullets), public IP/domain defanging, and removal of private/Microsoft infrastructure VT links.
- **5.2 Remove Generic / Forbidden / SOC-Side Recommendations**: Replaces generic advice or internal SOC-side actions with customer-side technical containment steps (host isolation, credential reset, firewall block, mailbox purge).
- **5.3 Fix Orchestration Justification Format & Remove Dossier**: Restricts suppression output strictly to Title, Type, Suppresses summary, Why Safe, and KVP table.
- **5.4 Strip Volatile Identifiers from KVP Filter Rows**: Prohibits volatile parameters (PIDs, timestamps, incident IDs) in filter rows; mandates stable anchors.
- **5.5 Fallback to Manual Closing Block**: Replaces unsafe filter generation with a `### Manually Closing` block.

### 6. Output Line 1 Enforcement
- **6.1 Force Strict Line 1 Header & Zero Wrapper**: Mandates line 1 format as `DISPOSITION: [verdict] · [confidence] · [priority] · ROUTE [1/2/3]` followed directly by the artifact block.

### 7. Escalation Artifact Formats (Per-Class)
Compact per-class fills for the single standard escalation block (`## [Priority]` with What was Observed / What is the Risk / What is Recommended), honoring the master line budgets: Observed ≤8, Risk exactly 2, Recommended ≤5.
- **7.1 Universal Escalation Contract**: Locks all classes to the one standard block plus the shared budget, defang, omission of browser/LOLBin/OS binary hashes/paths, and VirusTotal-fidelity rules.
- **7.2 Malware / Endpoint Execution**: Process tree, decoded command/target, hash, C2, persistence; containment via isolate, quarantine, remove persistence.
- **7.3 Phishing / Email**: Sender auth (SPF/DKIM/DMARC), originating IP, URLs/attachment, delivery/ZAP state; containment via purge, block sender, revoke sessions.
- **7.4 Identity / Sign-in (AiTM / Token Theft)**: Sign-in result, device state, MFA/Conditional Access, sign-in risk, token-replay signals; containment via revoke sessions, reset, kill OAuth grants.
- **7.5 Network C2 / Tunneling**: Originating process, destination reputation, protocol, volume/recurrence; containment via block infra, isolate, hunt other hosts.
- **7.6 Recon / Scanner (Unauthorized)**: Confirmed non-scanner role, targets enumerated, default-credential wordlist, breadth/velocity; containment via isolate source, hunt lateral movement.
- **7.7 Generic / Unclassified Escalation (Fallback)**: Anchor entity, activity/event, subject, target, key parameters, network IOC + VT, anomaly signal; generic scope-specific containment.

---

## Usage

### Manual Interaction
Paste the relevant snippet from [`steering_snippets.md`](file:///c:/Users/ConnorSmith/Downloads/prompt/steering_snippets.md) as a follow-up turn when an agent output strays from expected behavior.

### Automated Controller Loop
When integrating into an automated agent framework:
1. Validate agent output against schema/regex checks.
2. Match failure modes to the corresponding snippet ID.
3. Inject snippet into the conversation context to trigger single-pass correction.
