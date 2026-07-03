# Amex-Strategy-Track-26
Cardmember Profitability Modeling — Amex Campus Challenge 2026 (Round 1)
A deterministic, first-principles P&L framework that ranks 500,000 Premier Cardmembers by predicted annual profitability and flags the top 20% — built with zero labeled training data, calibrated purely through disciplined leaderboard experimentation.
Final public leaderboard accuracy: 0.901.
---
The Problem
Given ~23 masked behavioral attributes per cardmember (spend by category, revolve balance, risk score, benefit usage, service-call counts, etc.), predict which 20% of the portfolio is most profitable. The catch:
No training labels. The target is hidden; the only feedback signal is an accuracy score from a rate-limited public leaderboard (~20 total submissions).
Deterministic constraint. Round 1 rewards an explainable, scalable financial equation over black-box ML.
The evaluation metric is set-overlap accuracy on the top-20% flag, so only the 80th-percentile decision boundary matters — everything above or well below it is rank-neutral.
This turns the problem into something closer to reverse-engineering a hidden linear target under a tiny query budget than a standard supervised-learning task.
The Approach
1. Bottom-up P&L equation
Each member's 12-month profit is modeled as real issuer economics, every coefficient derived from public credit-card unit economics before any tuning:
```
Profit = InterchangeRevenue        (category-tiered discount rates)
       + RevolveInterestIncome    (net yield on revolving balance)
       + AnnualFee                 (rank-neutral constant)
       - RewardPointLiability      (accrual basis, per-point cost)
       - BenefitUtilizationCost    (lounge, travel credits, entertainment)
       - ExpectedCreditLoss        (LGD x RiskScore x Exposure)
       - ServicingPenalties        (cancellation & collection calls)
```
2. Label-free calibration loop
With no validation set possible, every leaderboard submission is treated as a scarce experiment:
One variable per submission. Strict coordinate descent; bundled changes are uninterpretable.
Flip-fraction pre-screening. Before spending a submission, a local diagnostic computes what fraction of members a candidate change moves across the top-20% boundary. Changes flipping <0.4% of members mathematically cannot move the score enough to justify the cost and are skipped.
Directional probing. When a coefficient shows a gradient (score improves as the weight shrinks/grows), follow it to convergence, then lock it.
3. Pipeline integrity engineering
Mid-competition, scores stopped responding to code changes. Root cause: a stale output artifact was being re-uploaded while edits ran elsewhere — silently corrupting score-to-config attribution. The fix became part of the methodology:
Config-stamped, timestamped output filenames (no fixed-name overwrites)
MD5 fingerprint of the prediction vector printed on every run and logged next to every leaderboard score
A rebuilt experiment ledger pairing each score with a verifiable artifact hash
Results
Milestone	Accuracy
First-principles economic baseline	0.841
Reward / discount-rate recalibration	0.855
Validated stable baseline	0.890
Over-weighted revenue term pruned (leaderboard-driven)	0.899
Servicing-penalty & revolve-yield tuning	0.901
Along the way, 6 structural hypotheses were tested and falsified at ~1 submission each — non-linear risk transformations, redemption-based (vs. accrual) reward costing, per-card fee scaling, welcome-bonus terms, and alternative spend bases — each rejection narrowing the inferred functional form of the hidden target.
Key lessons
Negative results are purchases, not failures. Every falsified hypothesis constrained the target's structure; the winning configuration was found mostly by elimination.
The metric defines the problem. Only boundary flips matter under set-overlap accuracy — rank-neutral "realism improvements" are wasted submissions.
Masked features are not what they seem. A feature labeled "total spend" proved numerically incompatible with dollar-scale spend (likely transformed/scaled), decisively falsified by a single controlled test.
Artifact integrity is a modeling problem. An hour of fingerprinting infrastructure is cheaper than three misattributed submissions.
Repository Structure
```
.
├── src/
│   ├── profitability_engine.py    # main P&L equation + submission generator
│   ├── diagnostics.py             # flip-fraction pre-screening vs. best-known config
│   └── experiments/               # one file per leaderboard probe, config-stamped
├── docs/
│   └── framework_writeup.md       # methodology documentation (submission Tab 2)
├── ledger.md                      # experiment log: config -> fingerprint -> score
└── README.md
```
> **Note:** Competition data is not included in this repository per challenge terms. Scripts expect the official dataset CSVs in an `uploads/` directory.
Running
```bash
pip install pandas numpy openpyxl
python src/profitability_engine.py     # prints diagnostics + writes config-stamped submission file
```
Every run prints the active configuration, the flip fraction against the best-known config, and an MD5 fingerprint of the prediction vector — upload nothing you haven't fingerprinted.
Stack
`Python` · `pandas` · `NumPy` · `openpyxl`
---
Built solo for the American Express Campus Challenge 2026, Round 1.
