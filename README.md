# Difference in Voting Difficulty by Political Party
**Datasci 203: Lab 1 | Authors: Sudiksha Sarvepalli & Sophie Ding**

---

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Dataset](#dataset)
3. [Approach & Methodology](#approach--methodology)
4. [Results](#results)
5. [Impact](#impact)
6. [Limitations & Challenges](#limitations--challenges)
7. [Learnings & Findings](#learnings--findings)
8. [Portfolio Significance](#portfolio-significance-for-a-data-scientist)
9. [Interview Pitch (30–40 sec)](#interview-pitch)

---

## Problem Statement

Voting is a fundamental right in the United States, yet access is not uniformly distributed. Prior research has documented various barriers to voting — voter ID requirements, state-by-state rule variation, long wait times, distance to polling stations, and transportation costs. However, **few studies have examined whether these barriers create systematic inequality in voting access along party lines**.

**Research Question:**
> *Do Democratic or Republican voters experience more difficulty voting?*

---

## Dataset

| Attribute | Detail |
|---|---|
| **Source** | 2022 American National Election Studies (ANES) Pilot Study |
| **Collection Method** | Online survey via YouGov Panel (non-probability, sample-matched) |
| **Total Respondents** | 1,585 (1,500 weighted + 85 unweighted) |
| **Analytical Sample** | 1,500 weighted respondents (unweighted 85 excluded) |
| **Final Analytical N** | 904 (478 Democrats, 426 Republicans after filtering) |
| **Key Variables** | `pid7` (7-point party ID scale), `votehard` (5-point Likert voting difficulty scale) |
| **Population** | U.S. citizens aged 18+, nationally representative |

### Variable Operationalization

**Party Identification (`pid7`):**
- **Democrats**: Strong Democrat + Not Very Strong Democrat + Lean Democrat
- **Republicans**: Strong Republican + Not Very Strong Republican + Lean Republican
- Rationale: Research (Petrocik, 2009) shows political leaners are not truly independent, so they were included with their respective parties

**Voting Difficulty (`votehard`):**
- 5-point Likert scale: 1 = *Not difficult at all* → 5 = *Extremely difficult*
- Captures overall voting difficulty (not a single barrier in isolation)

---

## Approach & Methodology

### Data Cleaning & Filtering
- Excluded 85 unweighted respondents using the `weight` field to ensure national representativeness
- Retained all 1,500 respondents regardless of registration status (since registration difficulty is itself a voting barrier)
- Dropped respondents not identifying as Democrat or Republican, leaving 904 observations

### Exploratory Data Analysis
- Built a **counts table** (Table 1) showing raw difficulty distributions by party
- Created a **clustered bar chart** (Figure 1) showing the *percentage* of voters per difficulty level for each party (necessary because group sizes differ: 478 D vs. 426 R)

| Difficulty Level | Democrats | Republicans |
|---|---|---|
| Not difficult at all | 400 (83.7%) | 388 (91.1%) |
| A little difficult | 52 (10.9%) | 26 (6.1%) |
| Moderately difficult | 19 (4.0%) | 10 (2.3%) |
| Very difficult | 3 (0.6%) | 2 (0.5%) |
| Extremely difficult | 4 (0.8%) | 0 (0.0%) |

### Statistical Test Selection

| Criterion | Reasoning |
|---|---|
| **Unpaired test** | Democrats and Republicans are independent groups |
| **Non-parametric test** | Voting difficulty is an ordinal Likert variable — CLT/normality assumptions don't apply |
| **Ranking methodology** | Ordinal data requires rank-based comparison, not difference-in-means |
| **Chosen test** | **One-sided Wilcoxon Rank-Sum Test** |

**Test Direction:** One-sided (alternative: Democrats > Republicans), motivated by descriptive data showing Democrats reporting higher difficulty across all non-trivial categories.

### Assumptions Verified
1. ✅ **Ordinal scale**: `votehard` is a Likert scale, satisfying the ≥ ordinal requirement
2. ✅ **IID sample**: YouGov panel with sample matching supports independence; both groups drawn from the same survey population

### Effect Size
- Computed **rank biserial correlation** to assess practical significance alongside statistical significance

### Code Snippet
```r
# One-sided Wilcoxon Rank-Sum Test
test <- wilcox.test(
  voter_data[voter_data$party_category == 'Democrats',]$votehard,
  voter_data[voter_data$party_category == 'Republicans',]$votehard,
  alternative = "greater"
)
```

---

## Results

| Metric | Value |
|---|---|
| **Test** | One-sided Wilcoxon Rank-Sum Test |
| **W Statistic** | 1.09392 × 10⁵ |
| **p-value** | 4.33 × 10⁻⁴ (~0.000433) |
| **Significance Level** | α = 0.05 |
| **Decision** | **Reject H₀** |
| **Effect Size (Rank Biserial Correlation)** | −0.11 (small effect) |

**Interpretation:**
- The p-value (0.000433) is well below α = 0.05, providing strong statistical evidence to reject the null hypothesis
- Since the test was one-sided, we also reject the hypothesis that Democrats experience ≤ the same difficulty as Republicans
- **Democrats experience statistically significantly more voting difficulty than Republicans**
- The effect size (r = −0.11) indicates the difference, while real, is **small in practical magnitude** — consistent with the visually similar distributions in Figure 1

---

## Impact

- **For policymakers**: Results provide evidence that voting access barriers are not politically neutral — they may disproportionately affect Democratic voters, which could inform equitable policy reform
- **For voter rights groups**: Quantifies the party-level gap in voting difficulty, supporting advocacy for targeted interventions
- **For researchers**: Establishes a statistically significant baseline finding that motivates deeper, more granular analysis of *which specific* barriers drive the difference

---

## Limitations & Challenges

| Limitation | Detail |
|---|---|
| **Geographic representativeness** | ANES data is nationally representative but NOT representative at the state or sub-state level — limits state-level policy applicability |
| **Observational data** | Cannot establish causality between party affiliation and voting difficulty; only correlation |
| **Composite difficulty measure** | `votehard` captures overall difficulty only; granular barrier-specific variables (wait time, ID, transportation) were not analyzed individually |
| **Self-reported data** | Survey responses may be subject to recall or social desirability bias |
| **Non-probability sampling** | YouGov panel uses sample matching — while carefully designed, it is not a true random sample |
| **Excluded respondents** | 85 unweighted respondents and ~596 non-Democrat/Republican identifiers were dropped, potentially limiting scope |
| **Cross-sectional snapshot** | Data reflects a single election cycle (2022 midterms); findings may not generalize across election years |

---

## Learnings & Findings

### Statistical & Methodological
- **Test selection matters**: Choosing the Wilcoxon rank-sum over a t-test was critical — a t-test would have been inappropriate for ordinal Likert data, and the paper explicitly reasons through why
- **Statistical vs. practical significance**: A statistically significant result (p < 0.001) paired with a small effect size (r = −0.11) is a nuanced and honest finding — important for real-world interpretation
- **Operationalization decisions have downstream effects**: How you define "Democrat" or "voter" materially changes your sample — explicit, theory-grounded decisions (e.g., including leaners) strengthen rigor
- **Normalizing for group size**: Recognizing that raw counts are misleading when group sizes differ (478 vs. 426) and pivoting to percentages for visualization is a key analytical judgment

- The vast majority of both Democrats (83.7%) and Republicans (91.1%) reported no difficulty — the effect is concentrated in the tail of difficulty reporters
- Democrats were more represented at every difficulty level above "not difficult at all," including being the only group with any "extremely difficult" respondents
- The finding challenges a naive assumption that rural Republican voters might face more difficulty due to geographic access issues

---

## Portfolio Significance for a Data Scientist

This project demonstrates several skills that are directly relevant to data science roles:

| Skill | How It's Demonstrated |
|---|---|
| **Hypothesis-driven analysis** | Research question → null hypothesis → test selection → interpretation pipeline |
| **Statistical rigor** | Deliberate, justified choice of non-parametric test for ordinal data |
| **Assumption validation** | Explicit verification of IID and ordinal scale assumptions |
| **Effect size reporting** | Goes beyond p-value to report rank biserial correlation — shows statistical maturity |
| **EDA & visualization** | Clustered bar chart with percentage normalization to handle unequal group sizes |
| **Operationalization** | Theory-grounded translation of abstract concepts (e.g., "Democrat") into measurable variables |
| **Communication** | Clean, structured report communicating technical findings to a non-technical audience |
| **Ethical awareness** | Acknowledges observational design limits; does not overclaim causality |
| **R / reproducible research** | Analysis conducted in R Markdown for full reproducibility |

---

*Project conducted using R and R Markdown. Data source: American National Election Studies (ANES) 2022 Pilot Study.*
