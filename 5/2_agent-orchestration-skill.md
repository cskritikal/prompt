```markdown
**Orchestration Justification**
**Title:** [concise filter title naming the detection + benign pattern — e.g. `Notepad→Edge Workday login handoff (CS - Notepad spawning processes)`]
**Type:** [net-new filter / filter modification / feed-based suppression / auto-routed playbook / alert comment playbook / event hint / temp filter]

FORMAT

### Intended Purpose of Orchestration
**Suppresses:** [ONE sentence — what this filter suppresses, by stable anchor, never a per-event ID]

### Justification/Supporting Documentation
**Why safe:** Why is it ok to suppress this? Why benign AND why a TP variant still alerts (different child/path/destination, or same host spawning shells/script hosts/LOLBins) — because the filter is anchored to this exact pattern. 1–3 sentences, no bullet sprawl.

**Filter Logic (KVP):**
Field              Operator           Value
[2–4 rows, strongest anchor first; use actual CORR/sensor field identifiers where known, e.g. `ioc.iocTitle`, `InitiatingProcessFileName`, `FileName`, `ProcessCommandLine`]
```
* **Keep it to Title / Type / Suppresses / Why safe / KVP only** — no user-device dossier, no scope-fit, CORR, or residual/expiry as standalone lines.
* **Operators (CORR):** `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.
* **KVP rules:** 2–4 rows, strongest anchor first; minimum = ONE strong anchor + ONE qualifier. Never enumerate the whole roster. Use a version-independent path prefix with `Contains`; `Command Line Does not contain "[TP differentiator]"` keeps a TP variant alerting. NO volatile identifier as a field/value (incident#, GUID, PID, SID, timestamp, port, internal/DHCP IP, file size, `type=unknown`); no internal IP in a hostname field.

## MANUAL CLOSING FORMAT
```markdown
### Manually Closing
[2–4 sentences: what fired (`rule` + pattern), the specific benign evidence, and why no durable filter is safe — name the anchor/boundary problem (only volatile identifiers distinguish it / a viable filter would suppress TPs / would break on cmdline variation / same-host recurrence is the likely TP). CORR stated.]
[If recurrence frequent: one line — tuning request with sample volume.]
```

## OUTPUT
One line, then the matching fenced block, nothing else:
`Verdict: [confirmed/indicated/unconfirmed] · [Suppress/Manual Closure]`

## SELF-CHECK (silent, before emit)
1. Provenance diff on every anchor and field used.
2. Eligibility gate applied and decided autonomously — no query back to the analyst.
3. Orchestration block is Title/Type/Suppresses/Why-safe/KVP only; 2–4 KVP rows, strongest anchor first, no volatile identifier; Manual Closing used instead when anchors <2 or filter unsafe.
4. No proactive customer-notification language anywhere in the output.
5. Output = verdict line + fenced block only.
