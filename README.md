<h1 align="center">🫀 Fetal Health Analysis using Cardiotocography (CTG) — Exploratory Data Analysis</h1>

> Cardiotocography (CTG) is a medical technique used to monitor fetal heart rate and uterine contractions during pregnancy. This project performs end-to-end Exploratory Data Analysis on CTG recordings to understand what distinguishes normal, suspect, and pathological fetal health states.

---

# 📌 Problem

The dataset contains CTG measurements classified into 3 fetal health states:

| NSP Value | State | Priority |
|-----------|-------|---------|
| 1 | Normal | Routine monitoring |
| 2 | Suspect | Daily monitoring needed |
| 3 | Pathological | Highest priority — baby at risk |

The goal of this EDA is to understand feature distributions, handle missing values with domain-aware imputation, detect and treat outliers, and surface correlations that could inform downstream classification.

---

# 📂 Dataset

| Property | Details |
|----------|---------|
| Source | [UCI Machine Learning Repository — Cardiotocography Dataset](https://archive.ics.uci.edu/dataset/193/cardiotocography) |
| File | `Cardiotocographic.csv` |
| Records | 2,126 CTG recordings |
| Features | 13 fetal heart rate measurements |
| Target | `NSP` — fetal health state (1, 2, 3) |

**Features:**

| Feature | Description |
|---------|-------------|
| `LB` | Baseline fetal heart rate (FHR) |
| `AC` | Accelerations per second |
| `FM` | Fetal movements per second |
| `UC` | Uterine contractions per second |
| `DL` | Light decelerations per second |
| `DS` | Severe decelerations per second |
| `DP` | Prolonged decelerations per second |
| `ASTV` | % time with abnormal short-term variability |
| `MSTV` | Mean short-term variability |
| `ALTV` | % time with abnormal long-term variability |
| `MLTV` | Mean long-term variability |
| `Width` | Width of FHR histogram |
| `Tendency` | Histogram tendency |

> **Medical context:** Fetal heart rate *variability* — how much the heart rate changes beat-to-beat — is a key health indicator. High variability = healthy. Decelerations (sudden drops in FHR) can signal reduced oxygen supply to the baby.

---

# 🔍 Approach

**Missing Value Treatment — Distribution-aware imputation:**

Rather than blindly using mean or median everywhere, each column was inspected individually:

| Column | Nulls | Distribution | Treatment |
|--------|-------|-------------|-----------|
| `LB` | 21 | Bell-shaped, few outliers | **Mean** |
| `AC` | 20 | Right skewed | **Median** |
| `DS` | 21 | Right skewed | **Median** |
| `DP` | 21 | Right skewed | **Median** |
| `MLTV` | 21 | Right skewed | **Median** |
| `Width` | 21 | Right skewed | **Median** |
| `Tendency` | 21 | Left skewed | **Median** |
| `NSP` | 21 | Right skewed | **Median** |

> `FM`, `UC`, `DL`, `ASTV`, `MSTV`, `ALTV` had no nulls.

**Dtype Fix:** `AC`, `DS`, `DP`, `MLTV`, `Width`, `Tendency`, `NSP` were stored as `object` dtype — converted to numeric using `pd.to_numeric(errors='coerce')` before analysis.

**Outlier Detection — IQR method:**

| Feature | Outliers |
|---------|---------|
| `FM` | 347 |
| `ALTV` | 318 |
| `DP` | 284 |
| `DL` | 125 |
| `DS` | 120 |
| `MLTV` | 81 |
| `MSTV` | 80 |
| `AC` | 40 |

**Outlier Treatment:** 5% Winsorization (`scipy.stats.mstats.winsorize`) — caps bottom and top 5% of each column. Before/after KDE plots compared for every feature.

**Visualizations produced:**
- Histplots + KDE for distribution shape per feature
- Boxplots for outlier detection per feature
- Before vs After Winsorization KDE overlays
- Correlation heatmap (full 14-feature)
- Two pairplots — features split into groups for readability

---

# 📊 Results

**Key correlation findings:**
- `ASTV` ↔ `ALTV`: **Strong negative** — when short-term variability is high, long-term variability is reduced
- `Width`, `Max`, `Min`: **Strong positive** among themselves — consistent histogram shape
- `Tendency`: **Weak correlation** with most other variables

**Medical insights from EDA:**
- Pathological cases (NSP=3) show high fetal movement counts and abnormal heart rate variability
- Severe and prolonged decelerations (`DS`, `DP`) are rare in normal cases but elevated in pathological ones — these are strong discriminative features for any downstream classifier
- `FM` and `ALTV` had the highest outlier counts (347 and 318) — these are clinically meaningful extremes, not data errors

---

# 🖼️ Key Visualizations

### Correlation Heatmap
![Heatmap](images/heatmap.png)

### Feature Distribution (Before vs After Winsorization)
![Winsorization](images/winsorization_kde.png)

> To reproduce all plots, run `fetal_health_eda.py` — all visualizations are generated and displayed inline.

---

# ▶️ How to Run

```bash
git clone https://github.com/Chaithanya449/Fetal-Health-EDA.git
cd Fetal-Health-EDA
pip install pandas numpy matplotlib seaborn scipy
python fetal_health_eda.py
```

> Ensure `Cardiotocographic.csv` is in the same directory as the script.

---

# 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.11-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-lightgrey?logo=pandas)
![NumPy](https://img.shields.io/badge/NumPy-Numerical-blue?logo=numpy)
![SciPy](https://img.shields.io/badge/SciPy-Winsorization-blue)
![Seaborn](https://img.shields.io/badge/Seaborn-Visualization-9cf)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Plots-yellow)

---

# 📁 Project Structure

```
Fetal-Health-EDA/
├── fetal_health_eda.py    # Main EDA script
├── Cardiotocographic.csv      # Dataset (2,126 records, 14 columns)
└── README.md
```

---

# 🔮 Next Steps

The EDA surfaced some really clear signals — `DS`, `DP`, and `ALTV` look like strong predictors of pathological cases just from the distributions alone. Want to build a classification model (Random Forest or XGBoost) on top of this cleaned data and see if those features rank high in feature importance.

Also want to address the class imbalance properly before modeling — Normal cases are 1,546 vs only 164 Pathological. A model trained without handling this will likely just predict Normal for everything and still look accurate. SMOTE or `class_weight='balanced'` would be the first things to try.

---

# 👤 Author

**Chaithanya Krishna** · [LinkedIn](https://www.linkedin.com/in/chaitanyakrishna-profile) · [GitHub](https://github.com/Chaithanya449)
