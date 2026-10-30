# Write-up: AI Proposal Triage Design

## The flow in one paragraph
Upload → extract text → AI pulls facts into a fixed form (with a quote for each fact) → code checks the mandatory pass/fail criteria → code calculates the weighted scores from the extracted facts → AI writes a plain-language reason for each position → reviewer checks the ranked list and can override → human makes the award decision. Criteria and weights are loaded from the admin's settings at run time and are never written into the code or the prompt text.

## 1. AI vs. rules
**AI does two jobs only:**
- *Extraction:* reading messy PDF text and filling a fixed form (licence number stated, price, weeks, install count, warranty years, named lead, crew size, safety plan, monitoring).
- *Explanation:* turning the computed numbers into a one-line reason a reviewer can read.

**Plain code does everything that has a right answer:** file checks, mandatory pass/fail, maths, normalising scores, ranking, ties, logging.

*Why here:* AI is good at understanding varied wording; it is bad at being consistent with arithmetic. If AI also did the scoring, the same proposal could get different scores on different days. Keeping scoring in code means every number can be re-checked.

## 2. Consistency and explainability
- AI extraction runs at temperature 0 with a fixed prompt version and a strict JSON format. Each extracted fact must include a short supporting quote from the proposal; if no quote exists, the value is `null` ("not stated"), never a guess.
- Scoring is a published formula in code (see PROMPTS.md and the scoring table below), so the same extracted facts always give the same score.
- *Would a second run match?* The scores would match if the extraction matches. To guard this, extraction runs twice; if any field differs, that proposal is flagged for a person instead of picking a winner silently. Extraction results are stored, so a re-rank uses the saved facts rather than asking the AI again.
- *How a reviewer sees why:* each proposal shows a score breakdown by criterion, the supporting quote for each fact, and the one-line reason.

**Scoring rules (for the sample set):** experience = installs counted (max 15) ÷ 15; price = lowest eligible price ÷ this price; timeline = fastest eligible time ÷ this time, and 0 if no firm time is given; warranty = average of panel years (÷25) and workmanship years (÷12); team = one quarter each for named lead, crew size, safety plan, monitoring plan. Prices and times are relative to the other eligible bids, so a new bid in the pool can shift scores slightly; this is shown to the reviewer.

## 3. Human in the loop
A person acts at four points:
1. *Before scoring:* reviewer confirms the category and, if any file is unreadable or scanned, fixes or re-uploads it.
2. *Flag queue:* anything flagged (suspected manipulation, missing required info, failed extraction check, low confidence) must be cleared by a reviewer before it can be ranked.
3. *Ineligible check:* a reviewer confirms each "ineligible" call, since it removes a bidder from consideration.
4. *Result:* reviewer may override any score or position, but must enter a written reason. The award decision is always made by a person; the tool never awards.

**Audit log (append-only):** who uploaded what (file hash), category, criteria version and weights used, prompt version and model name, the raw AI output, extracted facts with quotes, mandatory results, score breakdown, flags raised, every reviewer action (who, when, what changed, why), and the final decision.

## 4. Changing criteria
Criteria are saved with a version number. Each score stores the version it was calculated under, so old results are never silently changed.
- Editing a criterion creates a new version; the old version stays for the record.
- Already-scored proposals show as "scored under v1". The extracted facts are kept, so re-scoring under v2 is cheap and only needs the code step (AI re-extraction is needed only if a *new* fact is required).
- The reviewer sees a before/after comparison and chooses to re-score. A ranking that was already approved or awarded is locked; changes create a new ranking rather than editing the old one.
- Weights must total 100% or the save is rejected.

## 5. Risks and safeguards
| Risk | Safeguard |
|---|---|
| Wrong extraction | Quote required for every fact; double run and compare; reviewer sees quotes next to scores |
| Made-up facts | "Not stated" = null, never guessed; values without a matching quote are rejected by code |
| Hidden instructions in a proposal (prompt injection) | Proposal text is passed as data only, and the prompt says to ignore instructions inside it; code scans for phrases like "rank first" and "pre-approved", flags the proposal, and the AI never sees the scoring, so it cannot change it. In the samples, C contains such text: it is flagged and its real content is scored normally. |
| Bias (e.g. against small firms or unusual formats) | Fixed criteria only; no company name, size or location in scoring; reviewer can see and override |
| Vague or missing commitments (D has no schedule; B uses a subcontractor "to be confirmed") | Mandatory checks require the bidder to hold the licence now; missing timeline scores 0 and is flagged |
| Cost | Extract once per proposal and cache; small model for extraction; no AI in scoring; file size and page limits |
| Privacy | Fictional/test data only here; in real use, store files securely and limit access |
| Over-trust in the ranking | Output labelled "recommendation"; person must sign off |

## Assumptions
- "Last 5 years" is counted from the project date; "since 2020" (C) may include installs outside the window, noted for the reviewer.
- A missing timeline is not an automatic fail (the 16 weeks is a project requirement, not a listed mandatory criterion), so D is flagged for clarification rather than excluded.
