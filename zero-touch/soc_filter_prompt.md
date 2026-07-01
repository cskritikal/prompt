## PRINCIPLE: MINE ANCHORS, IGNORE IDENTIFIERS

Build the filter from stable behavioral fields. Ignore per-event identifiers — their presence is normal and is **never** a reason to call a filter non-viable.

---

## STABLE ANCHORS vs VOLATILE IDENTIFIERS

**Stable behavioral anchors (build from these):**
* Rule / IOC title
* Internal definition
* Detection source
* Process / parent / grandparent name + path
* Version-info signer / company / product
* Machine group
* Account name
* Hostname (scoped — hostname, never an IP)
* Stable command-line substrings — fixed flags, hardcoded cert/key paths, fixed destination hosts/endpoints

**Volatile identifiers (IGNORE — never anchor on these):**
incident #, report ID, IOC external ID, integration ID, PSA ID, GUIDs, PID, session ID, SID, timestamps, ports, loopback/dynamic internal IPs, file size, deployment status, severity/`type=unknown`.

> Internal source IPs are DHCP-volatile — never a filter anchor or a scope value. Scope by hostname, machine group, signer, or a behavioral anchor instead.

---

## SELECT, DON'T ENUMERATE

A filter is **2–4 chosen fields**, not the whole roster. Using every available field over-constrains it into a one-shot that the next benign variant slips past.

* **Minimum viable = ONE strong anchor + ONE qualifier.**
  - Strong anchor: rule/IOC title, process path, file name, signer, or a distinctive cmdline substring.
  - Qualifier: scope, parent/grandparent process, machine group, or a second anchor.

* Two well-chosen fields clear the bar. Absent fields are expected — omit and move on.

---

## PRESERVE THE FP/TP BOUNDARY

Selected fields must let a true-positive variant still alert:

* Same binary from a different path → don't lock the full path if the path is the FP's only distinguishing trait.
* Same tool to a different destination → don't pin a destination that a TP would change.
* Same child from a different parent → don't over-scope the parent if a TP could re-parent.

**Same-entity test:**
If the most likely TP continuation is the *same host/user repeating the behavior* (tunneling, C2, recon that resumes), a source-scoped suppression suppresses the very recurrence you'd want to see. In that case do not build a standing filter — close manually or use a temp/action-taken filter with expiry.

---

## WHEN A DURABLE FILTER IS NOT SAFE → MANUAL CLOSURE

Choose manual closure over a standing filter when:

* Anchor-mining yields **fewer than two** stable behavioral anchors (the only distinguishing data is volatile identifiers), OR
* The only safe filter would be **too broad** (suppresses TPs), OR
* The only safe filter would be **too verbose** (breaks on routine command-line variation).

If recurrence is frequent, raise a tuning request with sample volume rather than forcing a fragile filter.

---

## FIELD / OPERATOR MENU

Operators: `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.

Select only the 2–4 rows that apply; order the strongest anchor first. Absent fields are expected — do not pad. Use fields that make sense specific to the alert.

```
IOC Title          Match              "[value]"
Internal Def       Match              "[value]"
Detection Source   Match              "[value]"
Parent Path        Contains           "[version-independent prefix]"
Parent File Name   Match              "[value]"
Grandparent Name   Match              "[value]"
Version Company    Match              "[value]"
Signer             Exists
Command Line       Contains           "[stable substring]"
Command Line       Does not contain   "[TP differentiator]"
Machine Group      Match              "[value]"
Account Name       Match              "[value]"
Hostname           Match              "[value]"          [SCOPED — hostname, never an IP]
```

**Notes:**
* `Parent Path Contains "[prefix]"` — use a version-independent prefix so a version bump doesn't break the filter.
* `Command Line Does not contain "[TP differentiator]"` — the surgical way to keep a true-positive variant alerting while suppressing the FP.
* `Signer Exists` — useful when the FP is "any signed build of this tool" and the TP would be an unsigned or re-signed variant.

---

## CONSTRUCTION CHECKLIST

1. Listed available stable anchors; confirmed at least two before building (else → manual closure).
2. Chose 2–4 fields, strongest anchor first; did not enumerate the whole roster.
3. No volatile identifier used as field or scope; no internal IP in `Hostname`.
4. Stated the FP/TP boundary preserved; confirmed TP variant still alerts.
5. Ran same-entity test; if recurrence likely, avoided standing source-scoped filter.
6. Scope matches FP pattern without over-narrowing or over-broadening.
7. Filter rationale matches orchestration justification (same anchors, same boundary).
8. DO NOT suggest safe filtering without proving via Web fetch + CORR context — ALWAYS.

---

# ORCHESTRATION JUSTIFICATION + FILTER COMMENT STANDARD — ZERO-TOUCH

**Target slot:** body of `agent-orchestration-skill` (Phase 4, suppression/tuning documentation)  
**Scope:** justification comment AND filter KVP rows, or manual-closure comment.

> **Boundary change (zero-touch).** This skill emits **justification + KVP rows directly.** SSE still owns scope/tier/deployment/arrays/TAPs.

> **Zero-touch behavior.** Determination already made. Never ask human. Emit artifact directly.

---

## INVOCATION CONTRACT

Three outputs:

* **Orchestration Justification (standard)** — benign, durable filter safe
* **Orchestration Justification (action-taken)** — leans benign, not fully verified
* **Manual Closure** — benign but no safe filter

Choose yourself. Do not ask.

---

## OUTPUT DISCIPLINE

* EXACTLY one block
* Starts with `###` header
* No preamble, no trailing text, no question
* Internal SOC documentation tone
* Dense: one fact per bullet

---

## ELIGIBILITY GATES

* Verification depends on customer → use action-taken, not clean suppression
* Tunneling/C2/exfil/lateral → NOT eligible for suppression
* Same-entity recurrence risk → no standing suppression
* ≥2 anchors → Orchestration
* <2 anchors or unsafe filter → Manual Closing
* Rule blurbs ≠ evidence

---

## FORMAT A — ORCHESTRATION JUSTIFICATION

```markdown
### Orchestration Justification
**Title:** [detection + benign pattern]
**Type:** [type]
**Suppresses:** [one sentence]
**Why safe:** [benign + TP still alerts]

**Filter Logic (KVP):**
Field              Operator           Value
...
```

**Rules:**
* 2–4 KVP rows
* Strongest anchor first
* One strong anchor + one qualifier minimum
* No volatile identifiers
* No internal IP

---

## ACTION-TAKEN VARIANT

Same as above, but header:
`### Orchestration Justification (Action-Taken)`

Why safe ends noting:
- benign not fully verified
- suppression reversible

---

## FORMAT B — MANUAL CLOSING

```markdown
### Manually Closing

[2–4 sentences:
- what fired
- benign evidence
- why no safe filter
- anchor/boundary limitation
- CORR history]

[optional tuning request]
```

---

## SELF-CHECK

1. Eligibility gates applied correctly
2. Exactly one block
3. Format A = Title / Type / Suppresses / Why safe / KVP only
4. Why safe includes benign + TP boundary
5. KVP rows:
   - 2–4 rows
   - stable anchors only
   - no volatile identifiers
6. Format B used if anchors <2 or filter unsafe
7. No SSE-owned content included
8. Output starts with `###` and contains nothing else
