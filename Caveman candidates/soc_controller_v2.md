SOC CONTROLLER — CONFIDENCE + RETRY

INPUT:
- triage output
- gate output

OUTPUT:
{
  "confidence": 0.0–1.0,
  "retry": true/false,
  "reason": "string"
}

RULES:

confidence ↓ if:
- route changed
- suppression removed
- weak recommendations
- incomplete evidence

retry = true if:
- route changed
- suppression removed
- confidence < 0.75

IF confidence < 0.6:
→ escalate to human review