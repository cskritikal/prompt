```markdown
Analyst has determined the alert is Benign and eligible for suppression. Emit ONLY the Line 1 header and the markdown artifact block below. No conversational wrapper, questions, or progress narration.

Rules:
1. **Structure:** Output ONLY the 5 required fields (`Title`, `Type`, `Suppresses`, `Why safe`, `Filter Logic (KVP)`). Omit user dossiers, scope-fit sections, or commentary.
2. **Why safe (1–3 sentences):** State why the activity is benign AND why a true-positive (TP) variant will still alert. For LOLBins, safety must rest on verified command/target execution, never the signed parent binary alone.
3. **Filter Logic (2–4 KVP rows):** Strongest anchor first (Rule title, file path, signer, distinctive cmdline). Allowed operators: `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.
4. **Zero Volatile Identifiers:** BANNED in KVP rows: PIDs, GUIDs, incident IDs, internal DHCP IPs, timestamps, or transient session IDs.

Line 1: `DISPOSITION: Benign · Confirmed · Filter-Close · ROUTE 2 Orchestration`

```markdown
**Title:** [detection + benign pattern — e.g. `Notepad→Edge Workday login handoff (CS - Notepad spawning processes)`]
**Type:** [net-new filter / filter modification / feed-based suppression / auto-routed playbook / alert comment playbook / event hint / temp filter]
### Intended Purpose of Orchestration

Suppress [ONE sentence — by stable anchor, never a per-event ID, describing what will be suppressed in plain English.]

### Orchestration Justification

[Technical explanation - why this is benign AND why a TP variant still alerts. 1–3 sentences. LOLBin/behavioral rule: benign rests on the BEHAVIOR (what the command decoded/executed, shown expected), never the parent's signature or a clean AV verdict on the parent. Must not read as safe if the same signed parent on the same host could perform the malicious version.](In other words, why does this never need to be triaged again?)

**Filter Logic (KVP):**
Field | Operator | Value
[2–4 rows, strongest anchor first in context; e.g. `ioc.iocTitle`, `InitiatingProcessFileName`, `FileName`, `ProcessCommandLine`]
```
