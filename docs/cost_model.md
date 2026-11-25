# Cost model (offline arithmetic; replace prices and tokens with your measured values)
cost = queries x (tokens_in x price_in + tokens_out x price_out) / 10^6, queries = users x per-day x 30

Check from slide 18 (illustrative): 1,000 x 20 x 30 = 600,000 queries; 800 in / 300 out; USD 0.30 / 2.50 -> 144 + 450 = **USD 594**.
Check from Check-Your-Understanding Q3: **USD 120**.

Our scenarios (queries per month): Pilot 10 merchants x 2,000 orders x 2 messages = 40,000; Growth 100 merchants = 400,000; Lab standard 600,000.
Worked example if v2 averaged 750 input / 80 output tokens at USD 0.30 / 2.50 per million: each query costs 750 x 0.30 + 80 x 2.50 = 425 USD per million queries (about USD 0.000425). Pilot (40,000 queries) = USD 17; Growth (400,000) = USD 170; Lab standard (600,000) = USD 255 per month. This is an example only: use your measured tokens and real prices.

Not included: WhatsApp Business conversation fees, SMS fees, hosting, human support time. Prices: copy the paid-tier price of your exact model from the provider page on the day; free-tier limits change often.
Business check against the startup report: Starter plan Rs. 2,999/month for up to 500 orders; at 2 messages per order that is 1,000 queries, a tiny LLM cost per merchant, so LLM cost is not the margin risk; WhatsApp fees likely are.
