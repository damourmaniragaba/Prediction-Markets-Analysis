# What Makes a Successful Trader on Polymarket?

**Entry prices, sports betting, and returns in a prediction market**

INFO 2950 Final Project, Cornell University, Fall 2026
Ha An Pham Le (hp482), Alanna Yang (ay444), Jean D'Amour Maniragaba (jm2876)

---

## Table of contents
1. [Overview](#overview)
2. [Research questions and hypotheses](#research-questions-and-hypotheses)
3. [Key findings](#key-findings)
4. [Data](#data)
5. [Methods](#methods)
6. [Repository structure](#repository-structure)
7. [How to run](#how-to-run)
8. [Project timeline: phase by phase](#project-timeline-phase-by-phase)
9. [Limitations](#limitations)
10. [Ethics and privacy](#ethics-and-privacy)
11. [AI use](#ai-use)
12. [References](#references)

---

## Overview

Polymarket is an online prediction market where users buy and sell shares on whether future events will happen. A share pays $1 if its outcome happens and $0 if it doesn't, so its price (between $0 and $1) works like the market's estimate of the probability. Because the market is zero-sum, every dollar a winner makes is lost by someone else, and because every trade is recorded on the public Polygon blockchain, we can see the losses of ordinary users and not only the profits of the winners.

This project asks **which characteristics of a position, and which behaviors of a trader, are associated with higher returns on Polymarket.** We collected one year of positions (Oct 1, 2025 to Oct 1, 2026) for 1,000 leaderboard wallets and 1,000 wallets sampled from random markets, cleaned them into a position-level table (579,160 rows) and a trader-level table (1,844 rows), and tested two preregistered hypotheses with linear regression.

**Success is measured with return on investment (ROI):** profit or loss ÷ money spent. A share bought at price *p* returns (1 − p)/p if it wins and −1 if it loses. We use ROI instead of dollar profit because dollar profit mostly reflects how much money a trader has, not how good their trades are.

---

## Research questions and hypotheses

**Main question:** Which characteristics of a position, and which behaviors of a trader, are associated with higher returns on Polymarket?

**Sub-questions**
1. **Position level:** Is a position's ROI related to its entry price, category (sports, politics, crypto, weather, tech, culture, econ), size, holding time, and the event's trading volume?
2. **Trader level:** Is a trader's ROI related to how much of their money they put on sports, how much risk they take, how often they trade, and how long they have been on Polymarket, after accounting for whether they are a leaderboard trader or an ordinary trader?

**Hypothesis 1 (position level, favorite–longshot bias).** High-risk positions (bought under $0.20) have a lower ROI than medium-risk positions (bought from $0.20 to under $0.80). Studies of betting markets have long found that long shots lose more per dollar than favorites (Thaler and Ziemba; Snowberg and Wolfers).

$$
\text{ROI}_i = \beta_0 + \beta_1\text{highRisk}_i + \beta_2\text{lowRisk}_i + \beta_3\ln(\text{CostBasis}_i) + \beta_4\ln(\text{HoldingDays}_i) + \beta_5\ln(\text{EventVolume}_i) + \beta_{6..12}\,\text{category}_i + \varepsilon_i
$$

mediumRisk is the reference price group and sports is the reference category. Test: one-tailed t-test of β₁ < 0.

**Hypothesis 2 (trader level, sports betting).** Traders who spend a larger share of their money on sports have a lower ROI than traders who spend it in other categories. Sports bettors are known to favor popular sides in ways bookmakers exploit (Levitt), and our EDA showed sports had the lowest average position ROI.

$$
\text{traderROI}_j = \gamma_0 + \gamma_1\text{sportsShare}_j + \gamma_2\text{topTrader}_j + \gamma_3\text{highRiskShare}_j + \gamma_4\ln(\text{totalSpent}_j) + \gamma_5\text{positionsPerMonth}_j + \gamma_6\text{tenure}_j + u_j
$$

Test: one-tailed t-test of γ₁ < 0.

---

## Key findings

**Neither hypothesis was supported.**

| | Coefficient | 95% CI | One-tailed p | n | R² |
|---|---|---|---|---|---|
| H1: β₁ (highRisk) | **2.504** | [2.414, 2.594] | 1.000 | 552,451 positions | 0.011 |
| H2: γ₁ (sportsShare) | **0.047** | [−0.023, 0.117] | 0.906 | 1,752 traders | 0.163 |

**Hypothesis 1: cheap shares did better on average, but only because of a few huge wins.**
- Holding the controls fixed, a highRisk position's average ROI was about 2.50 higher than a mediumRisk position's, the *opposite* of what we predicted.
- The typical cheap position still lost: the median highRisk ROI was −0.95, and only 28% of highRisk positions made any money (mean ROI +2.83).
- The advantage comes almost entirely from the biggest wins. Removing the top 1% of ROI (above 12.64) drops β₁ from **2.50 to 0.08**; capping instead of removing gives 0.59; removing the top 5% turns it negative (−0.47).
- Clustering standard errors by trader makes the SE about 6 times larger (0.046 → 0.285), but β₁ is still about 8.8 SEs above 0.
- A win-rate-vs-price plot (no cutoffs) shows cheap positions made money more often than their price implied, the reverse of the favorite–longshot bias, though this is partly due to selling early for a small profit.

**Hypothesis 2: no clear relationship between sports betting and trader ROI.**
- γ₁ was small and positive (0.047) and stayed between −0.016 and 0.107, never significant, across every robustness check: traders with 10+ positions, top 1% of trader ROI removed, robust (HC3) SEs, random traders only, and leaderboard traders only.
- Leaderboard traders who specialized in sports did slightly *better* (0.107, 95% CI [−0.001, 0.216]).

**Exploratory findings (not preregistered).** In the trader model, being on the leaderboard (+0.25, significant by construction), putting more money on cheap shares (highRiskShare, +0.79 for all vs. none), and trading more often (−0.005 per extra position per month) were statistically significant. We treat these as exploratory.

**Answer to the main question.** In our data, how a trader handles price risk matters more than which topics they bet on. But "on average" is doing a lot of work: the gains from cheap shares come from rare huge payoffs, so most individual cheap bets still lose everything. These are associations, not causes.

---

## Data

### Source
All data comes from Polymarket's two free, public APIs (no key needed):
- **Data API** (`https://data-api.polymarket.com/v2`): leaderboard, trades, positions, and user stats. Docs: https://docs.polymarket.com/api-reference/data-api/overview
- **Gamma API** (`https://gamma-api.polymarket.com`): each event's tags and trading volume.

Data was collected on **10/11/2026**. Polymarket's data keeps changing, so rerunning collection later would give slightly different numbers; this is why the raw files are saved.

### Sampling
1. **Top traders:** the 1,000 wallets with the highest all-time profit on the leaderboard.
2. **Random traders:** 1,500 events chosen at random from 2,000 events that closed between Oct 1, 2025 and Oct 1, 2026. For each, one random trader from the most recent trades in the event's first market (excluding top traders), keeping the first 1,000 different wallets.
3. **Positions:** up to 500 of each wallet's most recent positions for each status (`CLOSED`, `REDEEMABLE`, `REDEEMABLE_LOST`), kept only if active in the study window. All three statuses are needed because losing bets held to resolution are often never cashed in.
4. **Join dates** from `/user-stats`, and **event tags and volume** from the Gamma API.

Collection produced five raw files (745,709 raw positions and 132,321 events) and takes over an hour to run.

### Cleaning
| Step | Rows |
|---|---|
| Raw positions | 745,709 |
| Remove duplicate positions (same resolved position under two statuses) | 612,956 |
| Remove prices outside $0–$1 | 610,701 |
| Compute ROI and remove impossible values (below −1 or above (1 − p)/p) | **579,160** |

The cleaning notebook then computes holding time, price group, and category; adds event volume (a volume of $0 is treated as missing); replaces wallet addresses with trader numbers; and summarizes positions into one row per trader. 1,844 of the 2,000 wallets still have at least one position.

### Clean files
- **`positions_clean.csv`**: 579,160 rows × 19 columns, one row per position (one trader's holdings in one outcome of one market).
- **`traders_clean.csv`**: 1,844 rows × 15 columns, one row per trader, built by summarizing that trader's positions.

Full column definitions are in [`writeup/column_definitions.md`](column_definitions.md). The most important variables:

| Variable | Meaning |
|---|---|
| `roi` | position ROI (`total_pnl` ÷ `cost_basis`) |
| `price_group` | highRisk (< $0.20), mediumRisk ($0.20 to < $0.80), lowRisk (≥ $0.80) |
| `category` | crypto, sports, weather, econ, tech, culture, politics, other, or unknown |
| `trader_roi` | trader's total PnL ÷ total money spent |
| `sports_share` | share of a trader's money spent on sports positions |
| `highRisk_share` | share of a trader's money spent on shares under $0.20 |
| `top_trader` | 1 if from the leaderboard, 0 if from the random sample |
| `tenure_months` | months from join date to Oct 1, 2026 |

### Where the data lives
- Raw files: https://drive.google.com/drive/folders/1qBlrM1k48SQcTbSTcElOIv_jracyIzQe
- Clean files: https://drive.google.com/drive/folders/1wDT1XpYTAMKCWmaXdpMVM4nS-b_bMojz?usp=sharing
- Collection and cleaning notebook (Colab): https://colab.research.google.com/drive/1ebqInnQ4C9gF8P11VGwltkmy8kyVQKcE?usp=sharing

---

## Methods

- **Models:** ordinary least squares with `statsmodels`, one model per hypothesis (positions for H1, traders for H2).
- **Transforms:** money spent, holding time, and event volume are very right-skewed, so they enter as natural logs. Holding times under one hour are set to one hour before logging.
- **Exclusions:** H1 leaves out positions with an unknown category or missing event volume (552,451 of 579,160 remain). H2 leaves out the 92 traders with no join date (1,752 of 1,844 remain).
- **ROI is not capped in H1**, because the rare big wins on cheap shares are exactly what the hypothesis is about.
- **Significance:** one-tailed t-tests (alternative: coefficient < 0) at α = 0.05.
- **Robustness checks (exploratory, added in Phase 4):** top 1% of ROI removed, ROI capped at the 99th percentile, alternative trimming cutoffs (top 5%, 1%, 0.1%), SEs clustered by trader (H1); 10+ positions only, top 1% of trader ROI removed, HC3 robust SEs, and each sample group separately (H2).
- **Model reading:** each model is read in three steps: summarizing (coefficients), predicting (a typical position or trader), and outliers and oddities (residual plots and the biggest misses).

> **Note on the preregistration draft.** [`writeup/preregistration.md`](preregistration.md) is an earlier Phase 2 draft. It describes 95% bootstrap confidence intervals, a trader sample limited to 10+ positions, trader ROI capped at the 1st/99th percentiles, and a $0.10/$0.90 cutoff check. The final reports use one-tailed t-tests from statsmodels on all traders with a join date, and the 10+ positions and trimmed-ROI versions appear as robustness checks.

---

## Repository structure

```
.
├── INFO2950 Phase 1-1.pdf                 # Phase 1: project idea, data source, questions for TAs
├── prediction markets project ideas.pdf   # early brainstorming: draft equations, hypotheses, limitations
├── Phase2_rubric__INFO2950_FA26.pdf       # course rubric for Phase 2
├── Phase2_Part1.ipynb                     # Phase 2: research questions, collection, cleaning appendix, data description, EDA
├── INFO2950 Phase 3 Feedback.pdf          # graded Phase 3 feedback (Gradescope)
├── positions_clean.csv                    # clean position-level data (579,160 rows)
└── writeup/
    ├── README_project.md                  # this file
    ├── column_definitions.md              # every column in the two clean files
    ├── preregistration.md                 # Phase 2 preregistration draft
    ├── prereg_review.md                   # review notes on the preregistration
    ├── notebook_review_natural_log.md     # Phase 2 notebook fixes (natural log, holding time)
    ├── phase3/
    │   ├── Data_Collection_and_Cleaning.ipynb   # Part A: collection (run 10/11/2026), Part B: cleaning
    │   ├── Phase3_Report.ipynb                  # Phase 3 report
    │   ├── 1_cleaning_notebook_changes.md       # change notes for the cleaning notebook
    │   ├── 2_phase3_report_notebook.md          # change notes for the Phase 3 report
    │   ├── final_fixes.md                       # last fixes before Phase 3 submission
    │   ├── introduction_rewrite.md              # rewritten introduction
    │   └── mla_citations.md                     # MLA bibliography
    └── phase4/
        ├── Phase4_Report.ipynb                  # FINAL report, run with all outputs
        ├── Phase4_Report.pdf                    # PDF preview of the final report
        ├── Phase4_changes_cell_by_cell.md       # every Phase 4 change, cell by cell
        ├── build_phase4_notebook.py             # script that built the Phase 4 notebook (reference only)
        ├── outlier_cells_to_paste.md            # outlier robustness cells
        └── README.md                            # Phase 4 submission checklist
```

`traders_clean.csv` and the raw files are on Google Drive (links above); they are not in this folder.

---

## How to run

The notebooks are designed for **Google Colab** (any Jupyter environment with Python 3 works).

**Requirements:** `numpy`, `pandas`, `matplotlib`, `seaborn`, `statsmodels`, and `requests` (collection only). All are preinstalled in Colab.

**To reproduce the final analysis (recommended):**
1. Download `positions_clean.csv` and `traders_clean.csv` from the clean-files Drive folder.
2. Open `writeup/phase4/Phase4_Report.ipynb` in Colab.
3. Upload both CSVs so they sit next to the notebook (the notebook reads them by file name).
4. Run all cells. The notebook does no further cleaning; it only builds dummies and logs right before each model.

**To rebuild the clean files from the raw data:**
1. Download the raw files from the raw-files Drive folder.
2. Open `writeup/phase3/Data_Collection_and_Cleaning.ipynb` and run **Part B** (cleaning). It writes `positions_clean.csv` and `traders_clean.csv`.

**To recollect from the API:** run **Part A** of the same notebook. It takes over an hour, and because Polymarket's data changes, the results will differ slightly from our 10/11/2026 snapshot.

---

## Project timeline: phase by phase

### Phase 1: Project idea
- Proposed studying top Polymarket traders: what prices they trade at, how risky their trades are, and what separates them from others.
- Identified the Polymarket Data API as a free, unauthenticated source (`/leaderboard`, `/trades`, `/positions`), verifiable against the Polygon blockchain.
- Asked the course staff whether correlational (not causal) analysis was acceptable and whether a single data source was enough.

### Early design (brainstorming)
- First drafts of two equations: a trade-level model of profit and a person-level model of total profit, with candidate hypotheses (tenure, risk-taking, sports, category focus).
- Key design changes from this stage:
  - **Fix selection on the outcome:** a leaderboard-only sample has no losers to compare against, so we added a random comparison group of 1,000 wallets from random markets.
  - **Use positions, not trades,** as the unit, since a single buy has no profit until it is sold or the market resolves.
  - **Use ROI, not dollar PnL,** so that results aren't driven by bet size.
  - **Drop selling price** as an input (it makes the regression nearly an identity) and **use price groups** because entry price is nonlinear.

### Phase 2: Data collection, cleaning, EDA, and preregistration
- Wrote the collection code for both samples and all three position statuses, plus join dates and event information from the Gamma API.
- Built the cleaning pipeline and data description (motivation, composition, collection process, people and uses) and a data limitations section.
- EDA covered summary statistics, missing data, the ROI distribution, entry prices, categories, holding time, risk and category focus, and correlations. Three results shaped the hypotheses: ROI is extreme (most positions lose everything or break even, a few cheap ones win big), random traders buy at the price extremes more than leaderboard traders, and sports is the most traded category but has the lowest average ROI.
- Switched skewed variables to natural logs and wrote the preregistration for H1 and H2.

### Phase 3: Report and analysis
- Split the work into a separate collection and cleaning notebook and a report notebook.
- Added an introduction with literature, definitions, a data reliability section, focused EDA, preregistration statements with the reasoning behind each hypothesis, both regressions, significance tests, conclusions, limitations, and an MLA bibliography.
- Received 14 comments from the TA and peer reviewers (in `INFO2950 Phase 3 Feedback.pdf`).

### Phase 4: Revisions and final report
Every Phase 3 comment was addressed (full list in Section 11 of the final report and in `Phase4_changes_cell_by_cell.md`). The main changes:
- Added a descriptive title, linked table of contents, and a "Why this matters" section with two new sources.
- Rewrote both equations with consistent notation (β₀–β₁₂ for positions, γ₀–γ₆ for traders).
- Fit both models once with `statsmodels` and moved the full summary tables into Data analysis.
- **Added outlier robustness checks**, which showed the highRisk advantage depends on the biggest wins, plus a mean-vs-median figure.
- Added SEs clustered by trader (H1), HC3 robust SEs and per-sample models (H2).
- Replaced the overplotted trader scatterplot with median ROI by sportsShare group, put the H1 residual plot on a symlog axis, and replaced the correlation heatmap with a one-line check.
- Discussed which control variables were significant and why they are treated as exploratory.
- Reframed "survivorship bias" as selection on the outcome, and expanded conclusions with takeaways for different readers and five next steps.

---

## Limitations

- **The random group isn't fully random.** Frequent traders, and those who trade near the end of a market, are more likely to be picked, so results describe active traders rather than all users.
- **The leaderboard group is selected on the outcome.** Its high ROI is built into the sample, so `top_trader` is only a control.
- **One person can have several wallets,** which can make traders look more focused than they are.
- **Many top wallets are probably bots,** so `top_trader` may partly measure automated strategies.
- **Positions are not independent.** Positions by the same trader or in the same market are related, so ordinary SEs are too small. Clustering by trader addresses the first problem but not the second.
- **Missing data is not evenly spread.** Event volume is missing for 4.6% of positions (14.4% of crypto), and tenure for 92 of 1,844 traders (mostly random traders).
- **Combos (parlay-style sports bets) are not included.**
- **Limits of the tests.** Position ROI is far from normal (residual skew ≈ 37.9), position-level R² is only about 0.01, and many unregistered control coefficients were tested.

---

## Ethics and privacy

- Wallet addresses were replaced with trader numbers and display names were dropped. Even so, a large position could be matched back to its public wallet, so we do not publish individual traders' results.
- Our results describe what happened to past traders, not a strategy to copy, and could be misread as encouraging gambling, especially because the sample over-represents successful traders.
- Markets on wars, disasters, and deaths are included in category averages but not analyzed separately.

---

## AI use

We used Claude, an AI assistant made by Anthropic, during this project. **[Edit this to match exactly how the group used AI, consistent with the AI declaration in Section 10 of the final report.]** The research questions and hypotheses are our own; we ran all code and checked every reported number against the notebook output.

---

## References

Barber, Brad M., and Terrance Odean. "Trading Is Hazardous to Your Wealth: The Common Stock Investment Performance of Individual Investors." *The Journal of Finance*, vol. 55, no. 2, 2000, pp. 773–806.

Levitt, Steven D. "Why Are Gambling Markets Organised So Differently from Financial Markets?" *The Economic Journal*, vol. 114, no. 495, 2004, pp. 223–46.

"Polymarket vs. Polling." *Polymarket Help Center*, 11 Jan. 2026. Accessed 8 Nov. 2026.

Rothschild, David M., and Rajiv Sethi. "Trading Strategies and Market Microstructure: Evidence from a Prediction Market." *The Journal of Prediction Markets*, vol. 10, no. 1, 2016, pp. 1–29.

Snowberg, Erik, and Justin Wolfers. "Explaining the Favorite–Long Shot Bias: Is It Risk-Love or Misperceptions?" *Journal of Political Economy*, vol. 118, no. 4, 2010, pp. 723–46.

Thaler, Richard H., and William T. Ziemba. "Anomalies: Parimutuel Betting Markets: Racetracks and Lotteries." *Journal of Economic Perspectives*, vol. 2, no. 2, 1988, pp. 161–74.

Wolfers, Justin, and Eric Zitzewitz. "Prediction Markets." *Journal of Economic Perspectives*, vol. 18, no. 2, 2004, pp. 107–26.

Full bibliography, including data and software sources, is in Section 10 of `writeup/phase4/Phase4_Report.ipynb`.
