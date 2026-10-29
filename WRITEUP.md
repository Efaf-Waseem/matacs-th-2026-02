# WRITEUP.md

Reference: MATACS-TH-2026-02. Proposals are referred to by letter only (A to E).

## 1. AI vs. rules

**AI is used in two places only:**
1. Reading each proposal and extracting facts into fixed fields (Prompt 1).
2. Writing a one-line reason for each result, using numbers that code already calculated (Prompt 2).

**Plain code does everything else:** loading the admin criteria, checking the JSON, mandatory pass/fail checks, score maths, ranking and the audit log.

**Why this line?** AI is good at reading messy text, so it extracts the facts. Pass/fail rules and weighted scores are business decisions that must be exact and repeatable, so they belong in code. The AI never scores or ranks.

## 2. Consistency and explainability

- Extraction runs at temperature 0 with a fixed JSON schema. Every fact must come with a short evidence quote. If a fact is not in the text, the value is null, never a guess.
- Scores come from fixed formulas, so the same facts always give the same score:
  - Experience: bands by number of installs of 100 kW or more.
  - Price: lowest eligible price / this price x 100.
  - Timeline: shortest weeks / this proposal's weeks x 100.
  - Warranty: half panel years, half workmanship years (relative to the longest).
  - Team: four checklist items (named lead, crew, safety plan, monitoring plan), 25 points each.
- **If ranked twice:** the numbers are identical if the extracted facts are identical. To catch extraction differences, the system extracts twice. If the two runs disagree on any field, the proposal is flagged for a person. Only the wording of the reason sentence may vary slightly, never a score or rank.
- **What the reviewer sees:** a score per criterion, the evidence quote behind each fact, and any flag. Price and timeline scores are relative to the eligible set, so adding or removing a bidder can change them. The screen states this.

## 3. Human in the loop

A person is involved at four points:
1. The reviewer confirms the category and the uploaded files.
2. Flagged items are reviewed. A proposal is flagged for missing data, hidden instructions, or disagreement between the two extraction runs.
3. Ineligible items are shown with the failed rule, so the reviewer can challenge a wrong extraction.
4. The reviewer checks the final list, can override a rank or status (a written reason is required), and makes the award decision. The tool only recommends.

**Audit log:** user, timestamps, file hashes, model name and settings, prompt versions, criteria version, extracted facts with evidence, scores, flags, overrides with reasons, and the final decision.

The tool only reads what a bidder claims. A reviewer must verify the licence in the official registry before any award.

## 4. Changing criteria

Criteria are versioned and never edited in place. Each edit creates a new version, and every score is stored with the version used.

- Old results stay as they were, labelled "scored under v1".
- The reviewer sees a banner when a newer version exists and can re-score. If only weights changed, code recalculates from the stored facts with no new AI call. If a new criterion needs new facts, the AI extracts again.
- Both runs are kept for comparison. Earlier overrides are shown but must be re-confirmed.
- If a mandatory rule changes, eligibility may change, so affected proposals are flagged.

## 5. Risks

| Risk | Safeguard |
|---|---|
| Wrong extraction | Evidence quote required, code checks the quote exists in the text, schema validation, double extraction, human sees evidence |
| Made-up facts | Missing data returns null, never a guess. A null is never scored, so the proposal is flagged (D has no timeline) |
| Manipulation (hidden text such as C's) | Proposal text is treated as data. Injection is detected and flagged, and C is scored on its real facts only. The AI returns JSON only, has no tools, and code ignores any rank or score the AI returns |
| Bias | AI does not score. Scoring uses only extracted criteria fields. Bidder names are masked at scoring time, and writing polish or length earns no points |
| Cost | Two small AI calls per proposal, results cached by file hash, only relevant pages sent, token limits set |
| Privacy | Use an approved provider, store only what the audit needs |
| Unverified claims | Licence and insurance are checked by a human before award |

## Assumptions and sample results

- "Last 5 years" is treated strictly. C says "since 2020", which may include work older than five years. Until a human confirms the dates, C's experience is scored in a lower band (40).
- B's electrical work is subcontracted and "to be confirmed", so the bidder does not hold the licence. B is ineligible despite the lowest price.
- Expected result: A (98) first, E (86) second. C (70) and D (not scored) are flagged for human review. B is ineligible.
- The experience bands are my own choice. The admin should be able to set them.