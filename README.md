# Vanguard A/B Test: Did the new design help more clients finish?

Module 2 project · Ironhack Data Analytics bootcamp

**Team:** [Name 1] · [Name 2] · [Name 3] · [Name 4]

---

## Table of contents

1. [Context](#context)
2. [Key findings](#key-findings)
3. [Data](#data)
4. [Data cleaning](#data-cleaning)
5. [Who the clients are](#who-the-clients-are)
6. [KPIs: Control vs Test](#kpis-control-vs-test)
7. [Hypothesis testing](#hypothesis-testing)
8. [Experiment evaluation](#experiment-evaluation)
9. [Tableau dashboard](#tableau-dashboard)
10. [Conclusion and recommendations](#conclusion-and-recommendations)
11. [Repository structure](#repository-structure)
12. [How to run](#how-to-run)
13. [Project management](#project-management)
14. [Challenges and learnings](#challenges-and-learnings)
15. [Links](#links)

---

## Context

We play newly hired data analysts in the Customer Experience (CX) team at **Vanguard**, a US investment management company.

Vanguard tested a redesigned online process: a more modern interface with in-context prompts (small hints shown during each step). Between **15 March and 20 June 2017**, clients were randomly split into two groups:

| Group | Experience |
|---|---|
| **Control** | The traditional online process |
| **Test** | The new, redesigned interface |

Both groups go through the same five pages:

```
start → step_1 → step_2 → step_3 → confirm
```

Reaching `confirm` means the client finished the process.

**Main question:** do more clients reach `confirm` with the new design?

---

## Key findings

- ✅ **The new design increases completion:** 65.6% (Control) → 69.3% (Test), **+3.7 percentage points** (+5.7% relative). The difference is statistically significant (Welch t-test, p ≈ 6.8 × 10⁻¹⁹).
- ⚠️ **The gain is not proven to clear Vanguard's 5% cost-effectiveness threshold:**
  - read as +5 points, the gain is clearly below it (p = 0.999);
  - read as +5% relative (≈ +3.3 points), it is above on paper but not significantly (p = 0.149).
- ✅ **Every age group gains**, most of all the youngest (<30: +6.8 pts) and the oldest (70+: +5.2 pts).
- ⚠️ **More back-and-forth with the new design:** 11.6% of step moves go backwards in Test vs 8.9% in Control, and the gap grows with age.
- ⏱️ **Faster overall** (median 3:57 vs 4:31 for completed visits), but **Step 1 is 7 seconds slower**.

**Verdict:** the new design works, but it isn't yet proven to be worth its cost.

---

## Data

Three datasets, linked by `client_id`:

| Dataset | Content | One row = |
|---|---|---|
| `df_final_demo` | Client profiles: age, gender, tenure, balance, number of accounts, calls and logons in the last 6 months | one client |
| `df_final_web_data` (pt_1 + pt_2) | Digital footprint: which step, which visit, what time | one click |
| `df_final_experiment_clients` | Experiment roster: Control or Test | one client |

**Source:** [data-bootcamp-v4 / files_for_project](https://github.com/data-bootcamp-v4/lessons/tree/main/5_6_eda_inf_stats_tableau/project/files_for_project)

<details>
<summary>Column descriptions</summary>

| Column | Meaning |
|---|---|
| `client_id` | Unique client ID |
| `Variation` | Control or Test |
| `visitor_id` | Unique ID per client and device |
| `visit_id` | Unique ID per web session |
| `process_step` | `start`, `step_1`, `step_2`, `step_3`, `confirm` |
| `date_time` | Timestamp of the click |
| `clnt_tenure_yr` / `clnt_tenure_mnth` | Years / months with Vanguard |
| `clnt_age` | Client age |
| `gendr` | Gender (M, F, U = unknown) |
| `num_accts` | Number of accounts |
| `bal` | Total balance across accounts |
| `calls_6_mnth` | Calls in the last 6 months |
| `logons_6_mnth` | Logons in the last 6 months |

</details>

---

## Data cleaning

Notebook: `day1-2_cleaning.ipynb`

| Step | What we did | Result |
|---|---|---|
| Web data | Merged pt_1 and pt_2, removed exact duplicates, converted `date_time` to datetime | **10,764** duplicate clicks removed → 744,641 rows |
| Experiment roster | Removed clients with no `Variation` (not part of the test) | **20,109** removed → 50,500 clients |
| Client profiles | Removed rows with missing values; recoded gender `X` (3 clients) as `U` | **15** removed |
| Merge | Joined the three tables on `client_id` | **50,487 clients** and **317,235 clicks** |

We also created a `completed` column: `True` if the client reached `confirm` at least once.

Two clean files are saved for the next steps and for Tableau:

- `df_clients.csv`: one row per client (group, profile, `completed`)
- `df_clean.csv`: one row per click (with the client's group)

---

## Who the clients are

Notebook: `day1-2_cleaning.ipynb`

| | Value |
|---|---|
| Median age | 48 years |
| Median tenure | 11 years with Vanguard (only 7.5% under 5 years) |
| Median balance | ≈ $65,700 |
| Gender | Three near-equal thirds: M 34%, U 34%, F 32% |
| Visits | 74% of clients did the process in a single visit |
| Overall completion | 67.6% |

**Older clients finish less:** about 52–57% for clients aged 70+, against 64–74% for younger groups.

**The two groups are comparable:**

| Average | Control | Test |
|---|---|---|
| Age | 47.5 | 47.2 |
| Tenure (years) | 12.1 | 12.0 |
| Balance | $150,152 | $148,963 |

---

## KPIs: Control vs Test

Notebooks: `day3_kpis.ipynb`, `day3_4_completion_rate_with_distribution.ipynb`, `KPI_error_rate_and_age_correlation.ipynb`, `vanguard-ab-test.ipynb`

### 1. Completion rate (main KPI)

| | Control | Test | Difference |
|---|---|---|---|
| Completion rate | 65.6% | 69.3% | **+3.7 pts** (+5.7% relative) |

### 2. Funnel: % of clients reaching each step

| Step | Control | Test |
|---|---|---|
| start | 100% | 100% |
| step_1 | 86.1% | 91.0% |
| step_2 | 79.7% | 83.4% |
| step_3 | 74.5% | 78.3% |
| confirm | 66.0% | 70.0% |

The new design keeps more clients at every step. The biggest gap is the first move, from `start` to `step_1`.

### 3. Time per step (median)

| Step | Control | Test | Change |
|---|---|---|---|
| step_1 | 20 s | 27 s | **+7 s** |
| step_2 | 64 s | 61 s | −3 s |
| step_3 | 72 s | 58 s | −14 s |
| Whole process (completed visits) | 4 min 31 s | 3 min 57 s | −34 s |

We use the **median** because some clients leave a page open for hours, which would pull an average up.

### 4. Error rate (backward navigation)

An "error" is a move back to an earlier step (for example `step_3 → step_2`). It is a proxy for confusion, not a confirmed technical error.

| | Control | Test |
|---|---|---|
| % of step moves that go backwards | 8.9% | 11.6% |

The gap grows with age: +0.6 pts for under-30s, +5.8 pts for clients aged 75+ (age × interface interaction significant, p ≈ 2.8 × 10⁻¹⁴).

> Note: the error rate depends on how it is counted. Per **step move** we get 8.9% vs 11.6%. Per **visit** (visits with at least one step back) we get 20.5% vs 27.0%. Both show more backward navigation in Test.

### 5. Completion by age group

| Age | Control | Test | Gain |
|---|---|---|---|
| <30 | 67.6% | 74.4% | +6.8 pts |
| 30–39 | 67.3% | 70.1% | +2.9 pts |
| 40–49 | 68.1% | 70.0% | +1.9 pts |
| 50–59 | 66.6% | 68.6% | +2.0 pts |
| 60–69 | 63.6% | 68.2% | +4.6 pts |
| 70+ | 51.9% | 57.0% | +5.2 pts |

Balance and tenure make no real difference: every balance and tenure group gains about +3.3 to +4.1 points.

---

## Hypothesis testing

Notebook: `day4_hypothesis_testing.ipynb`

- Significance level: **α = 0.05**
- Test: **Welch's t-test** on `completed` (1 = finished, 0 = not). Its mean is the completion rate. With groups this large, the rate is normally distributed (Central Limit Theorem, checked by bootstrap in `day3_4_completion_rate_with_distribution.ipynb`).

### Test 1: Is the completion rate different?

- H0: completion Test = completion Control
- H1: completion Test ≠ completion Control

| Difference | t | p-value | 95% CI | Decision |
|---|---|---|---|---|
| +3.71 pts | 8.88 | 6.8 × 10⁻¹⁹ | +2.89 to +4.53 pts | **Reject H0** |

The one-sided test (Test > Control) gives the same conclusion (p ≈ 3.4 × 10⁻¹⁹).

### Test 2: Is the gain above the 5% cost-effectiveness threshold?

- H0: gain ≤ threshold
- H1: gain > threshold

"5%" can be read two ways, so we tested both:

| Threshold | Value | t | p-value | Decision |
|---|---|---|---|---|
| +5 percentage points | 5.00 pts | −3.07 | 0.999 | Cannot reject H0 |
| +5% relative | 3.28 pts | 1.04 | 0.149 | Cannot reject H0 |

The new design is significantly better, but **we cannot prove it is better enough** to pass the threshold.

### Test 3: Are the two groups similar?

| Variable | Control | Test | p-value |
|---|---|---|---|
| Age | 47.50 | 47.16 | 0.016 |
| Tenure (years) | 12.09 | 11.98 | 0.087 |
| Balance | $150,152 | $148,963 | 0.658 (Mann-Whitney: 0.15) |
| Number of accounts | 2.26 | 2.25 | 0.036 |
| Calls (6 months) | 3.13 | 3.06 | 0.0006 |
| Logons (6 months) | 6.17 | 6.10 | 0.0009 |

Some differences are "significant", but they are tiny in practice. With 50,000 clients, even a 0.3-year age gap gets a small p-value. **Statistically significant ≠ practically important.** The randomization worked.

### Test 4: Completion gain by age group

The gain is positive in every age group and significant in 5 of 6 (all except 40–49, p = 0.057). With a stricter Bonferroni correction (α = 0.05 / 6 ≈ 0.008), it stays significant for <30, 30–39, 60–69 and 70+.

### Other tests

- **Age vs error rate** (Spearman): very weak positive correlation (ρ = 0.068, p ≈ 3.3 × 10⁻⁴⁹). Significant, but weak.
- **Time after step_1 by age group** (Kruskal-Wallis): significant (p < 0.05). Median time rises with age: 16 s (18–39), 25 s (40–59), 35 s (60+).

---

## Experiment evaluation

**What worked**

- Random assignment produced comparable groups (age, tenure, balance).
- About 50,000 clients give a precise estimate (95% CI of about ± 0.8 points).
- Three months of data.

**Watch out**

- The groups are not the same size: 23,526 (Control) vs 26,961 (Test).
- 20,109 clients in the files had no group, and we don't know why.
- A novelty effect (a boost just because the design is new) can't be ruled out.

---

## Tableau dashboard

🔗 **[Tableau Public link]**

The dashboard uses `df_clients.csv` and `df_clean.csv` and shows:

- KPI tiles: completion rate for Control and Test, and the gain in points
- The funnel by step, Control vs Test
- Completion by age group
- Filters: age group, gender, tenure, balance

![Dashboard screenshot](images/dashboard.png)

---

## Conclusion and recommendations

**The new design works, but it isn't yet proven to be worth its cost.**

1. **Clarify what "5%" means** with the business. A +5% relative gain is within reach; +5 points is not.
2. **Fix Step 1.** It is the one step that got slower, and the new design sends more clients backwards, especially those over 60.
3. **Re-test after the fix** with equal group sizes.

---

## Repository structure

```
vanguard-ab-test/
├── README.md
├── data/
│   ├── df_clients.csv
│   └── df_clean.csv
├── notebooks/
│   ├── vanguard-ab-test.ipynb                          # main notebook
│   ├── day1-2_cleaning.ipynb                           # cleaning + EDA
│   ├── day3_kpis.ipynb                                 # KPIs
│   ├── day3_4_completion_rate_with_distribution.ipynb  # completion rate by segment
│   ├── KPI_error_rate_and_age_correlation.ipynb        # error rate + age
│   └── day4_hypothesis_testing.ipynb                   # hypothesis tests
├── src/
│   └── [functions.py]
├── tableau/
│   └── [vanguard_dashboard.twbx]
└── images/
    └── [dashboard.png]
```

---

## How to run

```bash
git clone https://github.com/[your-username]/vanguard-ab-test.git
cd vanguard-ab-test
pip install pandas numpy matplotlib seaborn scipy statsmodels
jupyter notebook
```

Run `day1-2_cleaning.ipynb` first. It loads the raw data from GitHub and saves `df_clients.csv` and `df_clean.csv`. The other notebooks read those two files.

> Some notebooks use an absolute local path (`FOLDER = '/Users/...'`). Change it to `data/` before running.

---

## Project management

We organised the work on a Trello board (To do → Doing → Review → Done).

🔗 **[Trello board link]**

| Team member | Main tasks |
|---|---|
| [Name 1] | [cleaning, merging] |
| [Name 2] | [KPIs, funnel] |
| [Name 3] | [hypothesis tests] |
| [Name 4] | [Tableau, README] |

---

## Challenges and learnings

- **"5%" can mean two things.** 5 percentage points or 5% relative change the verdict, so we tested both.
- **Significant is not the same as important.** With 50,000 clients, tiny differences get small p-values, so we always looked at the size of the effect too.
- **A KPI needs one definition.** Our error rate changed depending on whether we counted visits or step moves, so we agreed on one and documented both.
- **Medians over means for time.** Some sessions stay open for hours and distort the average.

---

## Links

- 📊 Slides: [presentation link]
- 📈 Tableau: [Tableau Public link]
- 📋 Trello: [Trello board link]
- 💾 Data: [data-bootcamp-v4 / files_for_project](https://github.com/data-bootcamp-v4/lessons/tree/main/5_6_eda_inf_stats_tableau/project/files_for_project)
