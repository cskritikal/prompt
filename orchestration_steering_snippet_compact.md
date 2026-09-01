```markdown
Analyst has determined the alert is Benign and eligible for suppression. Emit ONLY the Line 1 header and the markdown artifact block below. No conversational wrapper, questions, or progress narration.

Rules:
1. **Structure:** Output ONLY the 5 required fields (`Title`, `Type`, `Suppresses`, `Why safe`, `Filter Logic (KVP)`). Omit user dossiers, scope-fit sections, or commentary.
2. **Why safe (1–3 sentences):** State why the activity is benign and why a true-positive variant will still alert. For LOLBins, safety must rest on verified command execution, never the signed parent binary alone.
3. **Filter Logic (2–4 KVP rows):** Strongest anchor first (Rule title, file path, signer, distinctive cmdline). Allowed operators: `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.
4. **Zero Volatile Identifiers:** BANNED in KVP rows: PIDs, GUIDs, incident IDs, internal DHCP IPs, timestamps, or transient session IDs.
5. **Unslop Writing:** Plain, direct words only. No em dashes in prose. No mid-sentence colon connectors. Ban AI vocabulary (crucial, delve, ensure, landscape, pivotal, underscore, serves as). State facts directly without puffery.

Line 1: `DISPOSITION: Benign · Confirmed · Filter-Close · ROUTE 2 Orchestration`

```markdown
**Title:** [detection + benign pattern, for example `Notepad→Edge Workday login handoff (CS - Notepad spawning processes)`]
**Type:** [net-new filter / filter modification / feed-based suppression / auto-routed playbook / alert comment playbook / event hint / temp filter]
### Intended Purpose of Orchestration

Suppress [ONE sentence by stable anchor, never a per-event ID, describing what will be suppressed in plain English.]

### Orchestration Justification

[Technical explanation: state why this activity is benign, and why a malicious variant still alerts. 1–3 sentences. For LOLBins, safety rests on the behavior, meaning the verified command line and executed target, never the parent signature or clean AV verdict. It must not read as safe if the same parent could run malicious code on the host.]

**Filter Logic (KVP):**
Field | Operator | Value
[2–4 rows, strongest anchor first in context, for example `ioc.iocTitle`, `InitiatingProcessFileName`, `FileName`, `ProcessCommandLine`]
```
