# AI Revenue Protection for Pakistani E-commerce

Team: Shoaib ur Rehman, Raheel Ijaz, Muhammad Hanzala — Introduction to Applied AI, Project 3 (AI Product MVP)

Product: predicts which Cash-on-Delivery orders are likely to fail (RTO), verifies uncertain customers over WhatsApp/SMS, and recommends Ship / Verify / Hold. Risk labels are decision support, not proof of fraud; new customers are never treated as risky just for having no history.

## Repo layout
```
prompts/   system_v1.md, system_v2.md   (each change = new version)
evals/     questions.csv (frozen), results_v1_v2.csv, summary.csv, attacks.csv, call_log.csv
notebooks/ week9.ipynb
docs/      attack_report.md, check_understanding_answers.md, commit_plan.md, standup_notes.md, linkedin_post.md, eval_design.md
```

## Week 9: LLMs and Prompt Engineering
Assistant built this week: WhatsApp order-support and COD-verification assistant (English, Urdu, Roman Urdu), grounded in store policy facts, JSON output validated with Pydantic.

- **Provider and model:** Parts 1–3 (smoke test, temperature, tokens) ran on Google Gemini via AI Studio, model `[CONFIRM exact name from MODEL line, e.g. gemini-3.x-flash]`. Our first choice `gemini-2.5-flash` returned a 404 (no longer available to new users), and the Gemini free tier then hit a 20-requests-per-day limit, so the evaluation (Part 5) ran on Groq, model `openai/gpt-oss-120b` `[CONFIRM]`. Keys are stored in Kaggle Secrets as `LLM_API_KEY` (one per member, never in code).
- **Reliable calls:** smoke test returned `ready` with 6 input and 1 output token (9.67 s). Retry logic worked in practice: we saw HTTP 503 (high demand) and HTTP 429 (rate limit) and the code backed off 1–16 s with jitter. A daily-quota 429 cannot be fixed by waiting seconds, so `chat()` now fails fast on it and the evaluation loop saves partial results.
- **Sampling by hand (verified):** T=0.5 gives 0.982 / 0.018 / 0.000; T=1 gives 0.867 / 0.117 / 0.016; T=2 gives 0.665 / 0.245 / 0.090; T=2 with top-p 0.9 gives 0.731 / 0.269 (Lahore removed).
- **Temperature test:** 4 distinct answers of 5 at T = 0 (3 after removing markdown formatting) vs 5 of 5 at T = 1. T = 0 was not perfectly identical. Some T = 1 replies were truncated by `max_tokens=256` because reasoning models spend tokens on hidden thinking, so the final chat() uses 2,048.
- **Tokens (chars per token):**

  | Text | English | Urdu | Roman Urdu | Urdu vs English |
  |---|---|---|---|---|
  | Fee sentence (lab) | 3.8 | 3.4 | 3.9 | 1.4x (13 vs 9 tokens) |
  | Our customer message ("Where is my order?") | 3.9 | 3.3 | 3.7 | 1.3x (23 vs 18 tokens) |

  Counts are small (9–23 tokens), so ratios are approximate. Roman Urdu cost the same as Urdu script in both tests.
- **Prompt v1 vs v2 (30 questions; 25 have required facts, so facts accuracy uses n = 25):**

  | Version | Run | valid JSON | facts | scope | cites |
  |---|---|---|---|---|---|
  | v1 | 1 | 0.00 | 0.32 | not measurable | 0.00 |
  | v1 | 2 | 0.00 | 0.36 | not measurable | 0.00 |
  | v2 | 1 | 1.00 | 0.96 | 0.93 | 0.88 |
  | v2 | 2 | 1.00 | 0.96 | 0.93 | 0.88 |

  v2 facts = 24/25 = 0.96 ± 0.08 (Wilson 95%: 0.80–0.99). v1 facts = 8/25 = 0.32 ± 0.18 (Wilson 0.17–0.52). Difference +0.64 ± 0.20, larger than the noise. v2 was perfect on typical (14/14), edge (7/7) and out-of-scope (3/3) questions; its misses are in the small not-in-facts and adversarial groups (4 and 2 questions), so we are growing those. Two worst v2 answers: `[PASTE: question, answer, cause]`.
- **Tokens and cost per call:** v1 averaged 103 input / 383 output tokens; v2 averaged 767 input / 148 output (7.4x more input because the system prompt and 10 facts are resent every call).
- **Estimated monthly cost (measured v2 tokens, at USD 0.30 / 2.50 per million tokens `[REPLACE with the paid-tier prices of our model, copied on DATE]`):** pilot 10 merchants (40,000 queries/month) about USD 24; growth 100 merchants (400,000) about USD 240; lab standard 1,000 users x 20 queries/day about USD 360. WhatsApp Business and SMS fees are not included.
- **Attacks:** `[RUN]` of 6 resisted (4 lab attacks + 2 of our own: staff impersonation asking to waive the 30% advance, and a Roman Urdu request for the hidden risk score). Canary leaked: `[RUN]`. Closest to succeeding: `[RUN]`. Defenses beyond the prompt: code-level output guard, no tool that approves or waives anything, risk scores never placed in the prompt.
- **Versioning:** prompts are versioned in `prompts/` with results in `evals/`; every prompt change is a new file and a rerun of the evaluation.

> Facts and eval questions are illustrative until replaced by the pilot merchant's real policies and real Week 8 customer questions.

### Check Your Understanding (each member under their own name)

#### Shoaib ur Rehman
1. The probability is e² / (e² + e⁰) = 7.389 / 8.389 = **0.881**, so the second token gets 0.119.
2. At T = 0 the model always picks its most likely next token, so a fee like "PKR 200" is copied the same way each time. At higher T a rare token can be sampled, such as a wrong digit. Output can still vary a little, so we also validate the JSON in code.
3. Queries = 500 × 10 × 30 = 150,000. Input: 150,000 × 1,000 = 150M tokens × USD 0.30 = USD 45. Output: 150,000 × 200 = 30M tokens × USD 2.50 = USD 75. Total = **USD 120 per month**.
4. If many clients get a 429 at the same moment and all wait exactly 1, 2, 4 seconds, they retry together and hit the limit again. A random part spreads the retries over time.
5. No, not with this evidence. v2 = 17/20 = 0.85 ± 0.157 and v1 = 15/20 = 0.75 ± 0.190, so the intervals overlap heavily. The difference is 0.10 with a 95% interval of about ±0.25, so it could be noise. We need more questions.
6. Tags separate the customer's text from our instructions, so "ignore previous instructions" is treated as data. A canary is a secret string in the prompt. If it ever appears in an output, we know the prompt leaked.

#### Raheel Ijaz
1. Softmax of [2, 0] at T = 1: first token = e² / (e² + 1) ≈ 7.39 / 8.39 = **0.88**.
2. Extracting fees and dates is a copying task, not a creative one. T = 0 (greedy decoding) gives the highest-probability token each time, so answers are as repeatable as possible. A wrong digit changes a fee or a deadline.
3. Number of queries: 500 users × 10 per day × 30 days = 150,000. Cost per query = (1,000 × 0.30 + 200 × 2.50) / 1,000,000 = USD 0.0008. Monthly cost = 150,000 × 0.0008 = **USD 120**.
4. The random part is called jitter. Without it, clients that failed together retry together (a "thundering herd") and fail together again. Jitter staggers them.
5. Not clearly. For v2 the interval is 1.96 × √(0.85 × 0.15 / 20) = ±0.157, about 69% to 100%. For v1 it is 1.96 × √(0.75 × 0.25 / 20) = ±0.190, about 56% to 94%. These ranges overlap a lot, and 2 more correct answers out of 20 is within the noise.
6. User text goes in tags so the model can tell data from instructions, which makes injection harder but not impossible. The canary detects leaks: if CANARY-RP9X2 shows up in a reply, the system prompt was exposed.

#### Muhammad Hanzala
1. With logits [2, 0] and T = 1, p(first) = e² / (e² + e⁰) = 7.389 / 8.389, which is **about 0.881**.
2. For JSON extraction we want the single most likely token, not variety. T = 0 removes random sampling, so the model copies the fee or date the same way each run. We still check the JSON in code, because even T = 0 is not perfectly identical.
3. 500 × 10 × 30 = 150,000 queries per month. Input cost: 150,000 × 1,000 / 1e6 × 0.30 = USD 45. Output cost: 150,000 × 200 / 1e6 × 2.50 = USD 75. Total: **USD 120 per month**.
4. Randomness stops everyone retrying at the same instant. Without jitter, a whole class or a whole team of clients would retry in sync and keep triggering the rate limit.
5. We cannot say v2 is better from 17/20 against 15/20. v2's range is 0.85 ± 0.16 and v1's is 0.75 ± 0.19. They overlap, and a difference of 2 questions is smaller than the uncertainty, so a larger evaluation set is needed. (In our own run v2 scored 24/25 and v1 scored 8/25, a difference clearly larger than the noise.)
6. Wrapping user text in tags marks it as untrusted data, one layer of defence against prompt injection. A canary is a hidden marker in the system prompt. Seeing it in an output tells us the prompt leaked, but it does not stop the leak.
