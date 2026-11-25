# Commit plan (every member commits from their own GitHub account)
30% of the Project 3 grade is individual: commits and peer evaluations (Week 12 peer review looks at commits, standups, and who ran which evaluation). Suggested split: swap if the team prefers.

| Member | Runs | Commits (own account) |
|---|---|---|
| Shoaib ur Rehman | Parts 1–3 (chat(), sampling, tokens/cost) with own key | `notebooks/week9.ipynb` Parts 1–3 outputs, cost section in README |
| Raheel Ijaz | Parts 4–5 (prompts, eval) with own key | `prompts/system_v1.md`, `prompts/system_v2.md`, `evals/questions.csv`, `evals/results_v1_v2.csv`, `evals/summary.csv` |
| Muhammad Hanzala | Part 6 (attacks, canary, guard) with own key | `docs/attack_report.md`, `evals/attacks.csv`, `docs/standup_notes.md`, LinkedIn post file |

Rules: each member **also runs the whole notebook once** with their own key (note the result in standup notes). One owner merges prompt changes (team decides who the ML lead is: write the name in standup notes). No prompt change without rerunning the eval; commit the prompt and its results together. Never commit a key.

Commit message examples: `week9: add system_v2 with scope + canary rules`, `week9: freeze 30-question eval set`, `week9: attack report and output guard`.
