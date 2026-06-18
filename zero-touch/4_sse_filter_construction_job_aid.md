# CORR FILTER CONSTRUCTION — SSE JOB AID
**Target slot:** analyst/SSE reference (NOT a skill, NOT fed to the agent).

> **Note (zero-touch):** the orchestration skill now emits the **2–4 KVP filter rows** directly (see `3_orchestration_justification_spec`). This job aid is no longer the source of those basic rows — it is the deeper reference for the construction the agent still does NOT do: scope/tier/deployment settings, array-field logic, TAPs/Django templates, and the human cross-check that the emitted KVP rows preserve the FP/TP boundary before deployment. The anchor/operator method below is kept consistent with what the skill emits so the two never disagree.

Use this to validate or extend the agent's emitted KVP rows, or when building a filter that needs scope-tier/array logic the skill does not produce.

---

## PRINCIPLE: MINE ANCHORS, IGNORE IDENTIFIERS
Build the filter from stable behavioral fields. Ignore per-event identifiers — their presence is normal and is **never** a reason to call a filter non-viable.

**Stable behavioral anchors (build from these):**
* Rule / IOC title
* Internal definition
* Detection source
* Process / parent / grandparent name + path
* Version-info signer / company / product
* Machine group
* Stable command-line substrings — fixed flags, hardcoded cert/key paths, fixed destination hosts/endpoints

**Volatile identifiers (IGNORE — never anchor on these):**
incident #, report ID, IOC external ID, integration ID, PSA ID, GUIDs, PID, session ID, SID, timestamps, ports, loopback/dynamic internal IPs, file size, deployment status, severity/`type=unknown`.

> Internal source IPs are DHCP-volatile — never a filter anchor or a scope value. Scope by hostname, machine group, signer, or a behavioral anchor instead.

## SELECT, DON'T ENUMERATE
A filter is **2–4 chosen fields**, not the whole roster. Using every available field over-constrains it into a one-shot that the next benign variant slips past.

* **Minimum viable = ONE strong anchor + ONE qualifier.**
  - Strong anchor: rule/IOC title, process path, file name, signer, or a distinctive cmdline substring.
  - Qualifier: scope, parent/grandparent process, machine group, or a second anchor.
* Two well-chosen fields clear the bar. Absent fields are expected — omit and move on.

## PRESERVE THE FP/TP BOUNDARY
Selected fields must let a true-positive variant still alert:
* Same binary from a different path → don't lock the full path if the path is the FP's only distinguishing trait.
* Same tool to a different destination → don't pin a destination that a TP would change.
* Same child from a different parent → don't over-scope the parent if a TP could re-parent.

**Same-entity test:** if the most likely TP continuation is the *same host/user repeating the behavior* (tunneling, C2, recon that resumes), a source-scoped suppression suppresses the very recurrence you'd want to see. In that case do not build a standing filter — close manually or use a temp/action-taken filter with expiry.

## WHEN A DURABLE FILTER IS NOT SAFE → MANUAL CLOSURE
Choose manual closure over a standing filter when:
* Anchor-mining yields **fewer than two** stable behavioral anchors (the only distinguishing data is volatile identifiers), OR
* The only safe filter would be **too broad** (suppresses TPs), OR
* The only safe filter would be **too verbose** (breaks on routine command-line variation).

If recurrence is frequent, raise a tuning request with sample volume rather than forcing a fragile filter.

---

## FIELD / OPERATOR MENU
Operators: `Match`, `Contains`, `In`, `Not In`, `Does not contain`, `Exists`, `Does not exist`.

Select only the 2–4 rows that apply; order the strongest anchor first. Absent fields are expected — do not pad.

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

Notes:
* `Parent Path Contains "[prefix]"` — use a version-independent prefix so a version bump doesn't break the filter.
* `Command Line Does not contain "[TP differentiator]"` — the surgical way to keep a true-positive variant alerting while suppressing the FP.
* `Signer Exists` — useful when the FP is "any signed build of this tool" and the TP would be an unsigned or re-signed variant.

## CONSTRUCTION CHECKLIST
1. Listed the available stable anchors; confirmed at least two before building (else → manual closure).
2. Chose 2–4 fields, strongest anchor first; did not enumerate the whole roster.
3. No volatile identifier used as a field or scope value; no internal IP in `Hostname`.
4. Stated the FP/TP boundary the selection preserves; confirmed a TP variant still alerts.
5. Ran the same-entity test; if same-source recurrence is the likely TP, did not build a standing source-scoped filter.
6. Scope matches the FP pattern without over-narrowing or over-broadening.
7. Filter rationale matches the justification comment the orchestration skill produced (same anchors, same boundary).
