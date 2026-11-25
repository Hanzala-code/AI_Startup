# Evaluation set design (evals/questions.csv, 30 questions)
| Category | Count | Share | Purpose |
|---|---|---|---|
| typical | 14 | 47% | everyday customer questions, English / Roman Urdu / Urdu script |
| edge | 7 | 23% | thresholds (PKR 15,000 COD limit, exactly PKR 5,000), day 9 return, post-dispatch cancel, "am I a scammer?" |
| not_in_facts | 4 | 13% | must say it does not know and refer to support |
| out_of_scope | 3 | 10% | essay, weather, movie: `in_scope=false` |
| adversarial | 2 | 7% | approve-my-order injection; staff impersonation + risk score request |

Rules we follow (slides 28, 31, 33):
- Questions must be replaced with **real** customer questions from the Week 8 interviews, in the customers' own words and languages, then **frozen**. Never tune prompts on questions we report; if we need tuning examples we write separate ones.
- Never reuse an eval question as a few-shot example.
- `must_include` uses digits or short stable words so correct answers in Roman Urdu or Urdu are not marked wrong; exact-phrase scoring still misses paraphrases, so the two worst answers are always checked by hand and a 1-5 human rubric (correct, complete, clear, safe) is added for at least 20 answers.
- With n = 25 scored questions an accuracy of 80% is only known to about +/- 16 points. Goal: 50+ questions by Week 12.
- Run twice at T = 0 and report both runs.
