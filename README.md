# Bank Loan Landing Page A/B Test  

# Objective  
Evaluate whether a new **landing page** design increases the conversion rate for bank-loan applications compared to the existing control page.  

<br/><br/>

# 1. Load Dataset and Quick Overview

### Dataset [`ab_data.csv`](data/ab_data.csv)

## Load, Quick Overview

```python
# import libraries
import pandas as pd
import numpy as np

# load dataset
df = pd.read_csv('../data/ab_data.csv')

# Quick structure check
df.info()

# Display first few rows
df.head()
```
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 294478 entries, 0 to 294477
Data columns (total 5 columns):
 #   Column        Non-Null Count   Dtype 
---  ------        --------------   ----- 
 0   user_id       294478 non-null  int64 
 1   timestamp     294478 non-null  object
 2   group         294478 non-null  object
 3   landing_page  294478 non-null  object
 4   converted     294478 non-null  int64 
dtypes: int64(2), object(3)
memory usage: 11.2+ MB
```

### Dataset (5 rows, 5 columns)
| user_id | timestamp | group | landing_page | converted |
|---|---|---|---|---|
| 851104 | 2017-01-21 22:11:48.556739 | control | old_page | 0 |
| 804228 | 2017-01-12 08:01:45.159739 | control | old_page | 0 |
| 661590 | 2017-01-11 16:55:06.154213 | treatment | new_page | 0 |
| 853541 | 2017-01-08 18:28:03.143765 | treatment | new_page | 0 |
| 864975 | 2017-01-21 01:52:26.210827 | control | old_page | 1 |

Each record represents a user landing on either the control or treatment page, along with whether they converted (submitted an application).  

### Description

| Column | Description |
|---------|--------------|
| `user_id` | Unique visitor ID |
| `group` | Control / Treatment |
| `landing_page` | Version shown |
| `converted` | 1 = User converted, 0 = Did not convert |

## Basic Sanity Check

```python
## Basic sanity check
print("Number of rows:", len(df))
print("Unique users:", df['user_id'].nunique())
print("Duplicated users:", df['user_id'].duplicated().sum(), "\n")

# Basic group and conversion counts
print(df['group'].value_counts(), "\n")
print(df['converted'].value_counts(), "\n")

# Confirming that `converted` has only 0/1 values, no missing data
df['converted'].value_counts(normalize=True) * 100
print("Numbere of NaN:", df['converted'].isna().sum())
```

```
Number of rows: 294478
Unique users: 290584
Duplicated users: 3894 

group
treatment    147276
control      147202
Name: count, dtype: int64 

converted
0    259241
1     35237
Name: count, dtype: int64 

Numbere of NaN: 0
```

Groups assignment is **balanced** (`treatment` = 147276, `control` = 147202).  
We have **3894** duplicated users, they should be deleted.

<br/><br/>

# 2. Initial Cleaning

- removing duplicate `user_id`s
- dropping rows with mismatching `group`/`landing_page` pairs ('control' = 'old_page', 'treatment' = 'new_page')

```python
# removing duplicated
# sorting by timestamp to leave only first occurrences
df = df.sort_values(by='timestamp')

# dropping duplicates
df = df.drop_duplicates(subset=['user_id'], keep='first')

df = df.sort_values(by='timestamp', ascending=False)

# dropping rows with mismatching group/landing_page pairs
# check mismatched combinations
mismatched = df.query(
    "(group == 'treatment' and landing_page != 'new_page') "
    "or (group == 'control' and landing_page != 'old_page')"
)

print("Number of mismatched rows:", len(mismatched))

# dropping mismatches
df = df.drop(mismatched.index)

# double-check that all combinations are now valid
# checking all the existing combinations in dataset after drop
# should have only two:
# control - old_page
# treatment - new_page
df.groupby('group')['landing_page'].value_counts()
```

```
Number of mismatched rows: 1949
group      landing_page
control    old_page        144319
treatment  new_page        144316
Name: count, dtype: int64
```

# 3. Exploratory Data Analysis (EDA)

- Group, Random Assignment Balance & Conversion Overview
- Quick Visualization

## Group & Conversion Overview

```python
# group sizes
group_counts = df['group'].value_counts()
print("Group sizes:\n", group_counts, "\n")

# percentage split
split_ratio = group_counts / group_counts.sum() * 100
print("Sample size ratio (%):\n", split_ratio.round(2), "\n")

# conversion rate by group
conversion_rates = df.groupby('group')['converted'].mean() * 100
print("Conversion rates (%):\n", conversion_rates, "\n")

# Summary DataFrame
summary = pd.DataFrame({
    'Users': df['group'].value_counts(),
    'Converted (%)': df.groupby('group')['converted'].mean() * 100
})
summary['Converted (%)'] = summary['Converted (%)'].round(2)
summary
```

```
Group sizes:
 group
treatment    147276
control      147202
Name: count, dtype: int64 

Sample size ratio (%):
 group
treatment    50.01
control      49.99
Name: count, dtype: float64 

Conversion rates (%):
 group
control      12.039918
treatment    11.891958
Name: converted, dtype: float64  
```

### Conversion Rates by Group

| group | users | Conversion Rate (%) |
| :--- | ---: | ---: |
| control | 144319 | 12.04 |
| treatment | 144316 | 11.87 |

## Conversion Rate Visualization

```python
import matplotlib.pyplot as plt

# Values
groups = conversion_rates.index
rates = conversion_rates.values

plt.figure(figsize=(6,4), facecolor='white')

bars = plt.bar(groups, rates, color=["#008080", "#ffd700"], width=0.6)

# Title and labels (added more top padding)
plt.title('Conversion Rate by Group', fontsize=14, weight='bold', pad=20)
plt.ylabel('Conversion Rate (%)', fontsize=12)
plt.xlabel('Group', fontsize=12)

# Keep only left (y-axis) and bottom (x-axis) spines
ax = plt.gca()
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

# Format y-axis with % sign
ax.yaxis.set_major_formatter(plt.FuncFormatter(lambda y, _: f'{y:.0f}%'))

# Remove grid lines
plt.grid(False)

# Add labels on bars
for bar in bars:
    height = bar.get_height()
    plt.text(
        bar.get_x() + bar.get_width() / 2,
        height + 0.25,
        f'{height:.2f}%',
        ha='center',
        va='bottom',
        fontsize=11
    )

plt.tight_layout()
plt.show()
```

![Conversion Rate by Group](charts/1-Conversion_Rate_by_Group.png)

# 4. Statistical Testing (Two-Proportion Z-Test)

- Two-proportion Z-test, p-value
- Distribution of Conversions (Counts)

We’re testing whether the conversion rate of the `treatment` group (`new_page`) is significantly different from that of the `control` group.

- **H0 (null)** - The new page doesn’t change the conversion rate.
- **H1 (alternativ)** - The new page does change the conversion rate (either up or down).

We’ll test it using a **z-test** for proportions — a standard test when comparing two independent proportions (conversion rates).  
Then we compute the **p-value** from the z-statistic to see if it’s below our significance level *(typically 0.05)*.

## Two-proportion Z-test, p-value

```python
from statsmodels.stats.proportion import proportions_ztest

# conversion counts
control_converted = df.query("group == 'control'")['converted'].sum()
treatment_converted = df.query("group == 'treatment'")['converted'].sum()

# sample sizes
n_control = df.query("group == 'control'").shape[0]
n_treatment = df.query("group == 'treatment'").shape[0]

# run two-proportion z-test
count = np.array([treatment_converted, control_converted])
nobs = np.array([n_treatment, n_control])

stat, pval = proportions_ztest(count, nobs, alternative='two-sided')

print(f"Z-statistic: {stat:.4f}")
print(f"P-value: {pval:.4f}")
```

```
Z-statistic: -1.4036
P-value: 0.1604
```

Since our ``p-value`` **(0.1604)** is greater than the typical significance threshold of **0.05**, we **fail to reject the null hypothesis.**  
There is no statistically significant evidence that the new landing page changed conversion rates. The observed difference could easily be due to random variation in the data.

## Distribution of Conversions (Counts)

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Custom colors
custom_palette = ["#008080", "#ffd700"]  # control, treatment

plt.figure(figsize=(6,4))
sns.countplot(
    data=df,
    x='group',
    hue='converted',
    palette=custom_palette
)
plt.title('Conversion Counts by Group', fontsize=14, weight='bold', pad=15)
plt.xlabel('Group', fontsize=12)
plt.ylabel('Number of Users', fontsize=12)
plt.legend(title='Converted', labels=['No', 'Yes'])
sns.despine()  # removes top/right borders for a clean look
plt.tight_layout()
plt.show()
```

![Conversion Counts by Group](charts/2-Conversion_counts_by_group.png)









### Experimental Overview  
- **Control group:** current landing page  
- **Treatment group:** redesigned landing page  
- **Metric:** conversion rate (proportion of users who applied)  
- **Hypothesis:**  
  - **H₀:** pₜₑₐₜ = p꜀ₒₙₜᵣₒₗ  
  - **H₁:** pₜₑₐₜ ≠ p꜀ₒₙₜᵣₒₗ  

---

### Key Results  

| Metric | Control | Treatment |
|---------|----------|-----------|
| Conversion rate | **12.04 %** | **11.87 %** |
| Lift | **−1.41 %** |
| Z-statistic | **−1.4036** |
| p-value | **0.1604** |

**Interpretation:**  
There is **no statistically significant difference** between the two landing pages (p > 0.05).  
The treatment page performed slightly worse, but the difference is small and could be due to random variation.

---

### Visual Insights  
- Bar chart comparing conversion rates (grey–pink palette)  
- Conversion counts by group  
- 95 % confidence-interval plot confirming overlap  
- Lift plot quantifying relative change  

*(Screenshots or chart images can be embedded here.)*

---

### Business Takeaways  
- The new landing page **did not outperform** the control.  
- Since the difference is not statistically significant, **keep the control page** for now.  
- Consider:  
  - Running a **power analysis** to check if the sample size was large enough.  
  - Testing **smaller, targeted design changes** rather than a full redesign.  

---

### Tech Stack  
`Python 3` · `pandas` · `numpy` · `matplotlib` · `seaborn` · `scipy`  

---

### Next Steps  
- Perform **power analysis** to determine optimal sample size for future tests.  
- Segment users by device type, region, or traffic source for deeper insights.  
- Turn this notebook into a **reusable A/B testing template** for portfolio and job applications.  
