# Handoff: SOC Steering & Megaprompt Doc Studio

> **Target Agent**: Gemini
> **Repo**: SOC Agent Steering Snippet Console & Megaprompt Triage Lab
> **Status**: Production-ready base web app (Impeccable 40/40, 0 detector anti-patterns)

---

## 1. System & Architecture

Repo = open-source reference lib + interactive web studio. Steer LLM security agents (`soc_one_shot_prompt.md`) & enforce post-output compliance via 25 modular snippets.

### Core Files
- [`index.html`](file:///c:/Users/ConnorSmith/Downloads/prompt/index.html): semantic HTML5 SPA — top command bar, expandable category sidebar, search bar w/ hotkey hints, snippet card stream, right simulator drawer, export modal.
- [`index.css`](file:///c:/Users/ConnorSmith/Downloads/prompt/index.css): Obsidian dark SOC console design system (`#0b0f17` base, `#06b6d4` cyan primary, `#10b981` status accents, Fira Code mono, CRT phosphor overlay, line-by-line diff styling).
- [`data.js`](file:///c:/Users/ConnorSmith/Downloads/prompt/data.js): primary data store:
  - `MEGAPROMPT_TEXT`: 169-line zero-touch triage prompt ([`soc_one_shot_prompt.md`](file:///c:/Users/ConnorSmith/Downloads/prompt/soc_one_shot_prompt.md)).
  - `RAW_TELEMETRY_PRESETS`: raw alert payloads (Malware Dropper, Phishing, DNS Tunneling, AiTM Token Theft).
  - `FLAWED_AGENT_OUTPUT_PRESET`: uncorrected agent response — intro narration, un-defanged IPs, 8 context lines, generic advice.
  - `SNIPPETS_DATA`: 25 snippets ∀ 7 categories.
- [`app.js`](file:///c:/Users/ConnorSmith/Downloads/prompt/app.js): logic — dynamic search, categorized tag filtering, clipboard copy, IP/URL defang toggle, CRT mode, Web Audio synth, line-by-line red/green diff engine, JSON/YAML export.
- [`soc_one_shot_prompt.md`](file:///c:/Users/ConnorSmith/Downloads/prompt/soc_one_shot_prompt.md): master prompt — zero-touch triage, 8-step extraction pass, Grounding Laws, priority re-derivation, 3 route gates.
- [`steering_snippets.md`](file:///c:/Users/ConnorSmith/Downloads/prompt/steering_snippets.md): reference doc — 25 snippets w/ trigger conditions & copyable blocks.

---

## 2. Completed

1. **25 snippets & 7 categories**: categorized, tagged, indexed; trigger callouts + sim test cases.
2. **Categorized tag cloud**: tags ∈ 5 collapsible groups (`⚙️ Workflow`, `🛡️ Grounding`, `🎯 Threat Classes`, `🔀 Priority & Routing`, `📝 Formatting`).
3. **Interactive playground & diff engine**: real-time line-by-line red/green diff (`- REJECTED` vs `+ STEERED`), char-by-char typewriter playback.
4. **Base megaprompt & auto-corrector**: 5-pass sanitizer (`autoCorrectAgentOutput`) — strip intro narration, force Line 1 header, defang public IOCs, kill generic recs, enforce line budgets.
5. **Retro CRT terminal mode**: green phosphor overlay, scanlines, Web Audio API key clicks.
6. **Global shortcuts**: `/` | `Ctrl+K` focus search; `Ctrl+Enter` run sim; `Esc` close modal.

---

## 3. Roadmap (Gemini)

Build on foundation → enhance UX, telemetry parsing, Megaprompt enforcement.

### Phase 1: Multi-Vendor Telemetry Parser & Grounding Validator
- Expand `runMegapromptTriage` in `app.js` → parse nested JSON/XML ∀ major EDR & SIEM:
  - Microsoft Defender XDR (`AlertInfo`, `DeviceInfo`, `ProcessInfo`)
  - CrowdStrike Falcon (`event`, `ProcessId`, `CommandLine`)
  - SentinelOne (`agentDetectionInfo`, `threatInfo`)
  - Splunk CIM (`src`, `dest`, `user`, `signature`)
- Add real-time grounding-law validation → flag ∀ output value ∉ raw telemetry.

### Phase 2: Interactive KVP Filter Builder (Route 2 Orchestration)
- Visual UI builder in Playground ∀ Route 2 Orchestration Justification.
- Select 2–4 stable anchors (`InitiatingProcessFileName`, `ProcessCommandLine`, `ioc.iocTitle`) & validate ∉ forbidden volatile identifiers (PIDs, GUIDs, timestamps, internal IPs).

### Phase 3: Real-Time Megaprompt Compliance Scorecard
- Live 100-point scorecard drawer in Playground.
- Grade pasted responses ∀ 10 silent Self-Check gates from `soc_one_shot_prompt.md`:
  - [ ] Line 1 DISPOSITION header format (10 pts)
  - [ ] Zero intro/outro narration (10 pts)
  - [ ] Public IP/URL defang (10 pts)
  - [ ] Observed budget ≤ 8 (10 pts)
  - [ ] Risk budget == 2 (MITRE + Attack Path) (10 pts)
  - [ ] Recommended budget ≤ 5 imperative verbs (10 pts)
  - [ ] No forbidden phrases ("notify customer", "monitor") (10 pts)
  - [ ] Context bullets ≤ 2 (10 pts)
  - [ ] RFC1918 private IPs plain / untouched (10 pts)
  - [ ] Single fenced artifact block only (10 pts)

### Phase 4: One-Click SOAR & Webhook Exporter
- Modal export → routed artifacts →:
  - Cortex XSOAR / Shuffle webhook JSON
  - Microsoft Sentinel Automation Rules
  - Jira / ServiceNow security incident payload

---

## 4. Verification & Testing

```powershell
# 1. Run static design detector scan (MUST return [])
node .gemini/skills/impeccable/scripts/detect.mjs --json index.html

# 2. Check JavaScript syntax
node -c data.js; node -c app.js

# 3. Serve locally on port 3000
npx -y serve . -p 3000
```
