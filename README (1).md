# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** AKHTER HUSSAIN
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** 1 hour

## Files
- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): extraction and explanation prompts
- [ranking.json](ranking.json): expected ranking of proposals A to E
- [diagram/flow.png](diagram/flow.png): process flow from upload to ranked output

## Summary
AI reads and explains; plain code checks the pass/fail rules, does the scoring and ranks. Anything suspicious or incomplete goes to a person. Result: A first, E second, C and D flagged for review, B ineligible.

## How I used AI tools
Yes I also help with AI tool like this diagram through Claude and Chatgpt is also used for undertanding the question. I used Claude to draft the design write-up, prompts and diagram. I changed its decision on Proposal D from “flagged” to “ranked 3rd with a timeline score of 0”, because the brief lists the 16-week limit as a project requirement, not a mandatory criterion, so excluding D from the ranking seemed too strict.
