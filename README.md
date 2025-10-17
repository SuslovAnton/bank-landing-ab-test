# 🧪 Bank Loan Landing Page A/B Test  

### 🎯 Objective  
Evaluate whether a new **landing page** design increases the conversion rate for bank-loan applications compared to the existing control page.  

---

### 📂 Dataset  
File: `ab_data.csv`  
Each record represents a user landing on either the control or treatment page, along with whether they converted (submitted an application).  

| Column | Description |
|---------|--------------|
| `user_id` | Unique visitor ID |
| `group` | Control / Treatment |
| `landing_page` | Version shown |
| `converted` | 1 = User converted, 0 = Did not convert |

---

### 🔍 Experimental Overview  
- **Control group:** current landing page  
- **Treatment group:** redesigned landing page  
- **Metric:** conversion rate (proportion of users who applied)  
- **Hypothesis:**  
  - **H₀:** pₜₑₐₜ = p꜀ₒₙₜᵣₒₗ  
  - **H₁:** pₜₑₐₜ ≠ p꜀ₒₙₜᵣₒₗ  

---

### 🧮 Key Results  

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

### 📊 Visual Insights  
- 📈 Bar chart comparing conversion rates (grey–pink palette)  
- 📊 Conversion counts by group  
- 🎯 95 % confidence-interval plot confirming overlap  
- 💹 Lift plot quantifying relative change  

*(Screenshots or chart images can be embedded here.)*

---

### 🧠 Business Takeaways  
- The new landing page **did not outperform** the control.  
- Since the difference is not statistically significant, **keep the control page** for now.  
- Consider:  
  - Running a **power analysis** to check if the sample size was large enough.  
  - Testing **smaller, targeted design changes** rather than a full redesign.  

---

### ⚙️ Tech Stack  
`Python 3` · `pandas` · `numpy` · `matplotlib` · `seaborn` · `scipy`  

---

### 🧩 Next Steps  
- 🔍 Perform **power analysis** to determine optimal sample size for future tests.  
- 📊 Segment users by device type, region, or traffic source for deeper insights.  
- 🧰 Turn this notebook into a **reusable A/B testing template** for portfolio and job applications.  
