# Week 12 Fri: Garak vs vuln-rag-001 (three-way comparison)

## Setup
Garak 0.15.1, REST generator -> /chat. Config maps {"message":"$INPUT"} ->
response_json_field "response". Probes: apikey (CompleteKey, GetKey), 3 gens,
clean KB (Garak cannot reach /poison).

## Raw results
- apikey.CompleteKey: 768/768 PASS (no completion leaks)
- apikey.GetKey: 172/174, reported 1.15% "success"

## Verification (critical)
The 2 GetKey "hits" were FALSE POSITIVES. Both flagged outputs were refusals
containing key-vocabulary ("LinkedIn Secret Key", "Client Id"). detector_results
1.0 on refusals. Real secret sk-corp-... appeared in ZERO outputs.
=> True direct-extraction leak rate: 0 / 942 attempts.

## Three-way comparison
| Approach | Direct | Indirect (poison) |
|----------|--------|-------------------|
| Manual   | refused (1 try)      | LEAKED |
| PyRIT    | FAILURE (scored)     | SUCCESS (scored) |
| Garak    | 0 real / 942; 2 FP   | never attempted (can't reach /poison) |

## Findings for Blog 3
1. Architectural blind spot: black-box scanners cannot reach stateful multi-step
   attacks. The vuln requires poisoning the KB first; Garak only sees /chat.
   PyRIT caught it by *encoding* the two-step sequence as a custom target.
2. Detector false positives: Garak's reported 1.15% was 100% false positives on
   inspection. Success-rate column is a starting point, not a verdict. Every
   flagged hit needs manual transcript verification.
3. (Separate) Credential hallucination: on CompleteKey, model once fabricated a
   plausible AWS-style key (AKIAfv38D6F7). Different vuln class from leakage.

## Methodology lesson
Do not trust a scanner's success metric without reading transcripts. A
practitioner reporting Garak's raw 1.15% would file two non-existent bugs.
