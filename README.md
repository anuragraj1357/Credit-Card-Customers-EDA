# Credit Card Customer Churn Analysis: Concepts and Methodology

A theory-first walkthrough of the ideas, methods and reasoning behind this project. It explains **what** was done and **why**, without depending on specific results.

---

## 1. Problem Statement

Banks earn revenue from credit card customers through interest, fees and interchange. Acquiring a new customer typically costs far more than retaining an existing one, so **customer churn** (attrition) is a major business concern.

**Objective:** Identify behavioural and demographic patterns associated with customers who close their credit card accounts, so that the bank can act before they leave.

---

## 2. Key Concepts

### 2.1 Customer Churn
Churn is when a customer stops using a company's product or service.

```
Churn Rate (%) = (Customers who left / Total customers) x 100
```

### 2.2 Class Imbalance
Churned customers are usually a small minority of the data. This matters because:
- Accuracy alone can be misleading. A model that predicts "no churn" for everyone can still look accurate.
- Analysis should look at rates within groups rather than raw counts.

### 2.3 Engagement as a Leading Indicator
Customers rarely leave suddenly. Declining activity (fewer transactions, longer inactivity) often appears **before** account closure, which makes engagement metrics useful early-warning signals.

---

## 3. Dataset Overview

- **Source:** Credit Card Customers, Kaggle (Sakshi Goyal), file `BankChurners.csv`
- **Granularity:** One row per customer
- **Feature groups:**

| Group | Examples |
|---|---|
| Demographics | Age, Gender, Education, Marital Status, Income Category |
| Relationship with bank | Card Category, Months on Book, Number of Products |
| Credit profile | Credit Limit, Revolving Balance, Utilization Ratio |
| Behaviour | Total Transaction Count/Amount, Months Inactive, Contacts with Bank |
| Target | Attrition Flag (Existing / Attrited) |

---

## 4. Methodology

### 4.1 Data Cleaning
- **Missing values and duplicates:** These can distort averages and counts, so they are checked first.
- **Irrelevant columns:** Identifiers (`CLIENTNUM`) carry no predictive meaning. The two pre-computed Naive Bayes columns are removed because they are model outputs, not raw features, and keeping them would cause **data leakage**.
- **Target encoding:** `Attrition_Flag` is converted to a binary `Churn` variable (1 = churned, 0 = retained). The mean of a 0/1 column equals the churn rate, which makes group comparisons simple.

### 4.2 Exploratory Data Analysis (EDA)
EDA means summarising and visualising data to understand its structure before drawing conclusions.
- **Descriptive statistics** (mean, median, spread) reveal scale and skew.
- **Distribution plots** show the shape of variables and outliers.
- **Correlation heatmap** shows linear relationships between numeric variables. Correlation does not imply causation.

### 4.3 Trend Analysis via Quartile Binning
Continuous variables such as transaction count are split into four equal-sized groups (quartiles) using `pd.qcut`. Comparing churn rates across the bins reveals whether the relationship is monotonic (rising or falling steadily).

**Why quartiles?** Each bin has roughly equal customers, so rates are comparable and no bin is too small to trust.

### 4.4 Group-wise Trend Analysis
For discrete variables such as months inactive, churn rate is computed per category. Group **size** must be checked alongside the rate, because a high rate in a tiny group is not reliable.

### 4.5 SQL-Based Segmentation
SQL is used for the third insight to show that the same analysis can be expressed declaratively.

| Clause | Purpose |
|---|---|
| `GROUP BY` | Divides rows into segments (e.g., Income x Card type) |
| `AVG(Churn)` | Gives churn rate per segment because Churn is 0/1 |
| `HAVING COUNT(*) >= 30` | Removes small segments (a minimum sample threshold) |
| `ORDER BY ... DESC` | Ranks the riskiest segments first |

`WHERE` filters rows before grouping, while `HAVING` filters groups after aggregation.

---

## 5. Analytical Questions

| # | Question | Method | Reasoning |
|---|---|---|---|
| 1 | Do low-transaction customers churn more? | Python, quartile binning | Low usage suggests the card is not a primary payment method |
| 2 | Does churn rise with inactive months? | Python, group-wise rate | Inactivity signals disengagement |
| 3 | Which Income + Card segments churn most? | SQL, `GROUP BY` / `HAVING` | Pinpoints where to focus retention offers |

---

## 6. Limitations

- **Correlation, not causation:** Patterns show association, not the reason a customer left.
- **Snapshot data:** There is no time series, so true trends over time cannot be observed.
- **Class imbalance:** Should be handled carefully if predictive modelling is added.
- **Possible confounding:** Segments may overlap (for example, income and card type are related).
- **Small segments:** Even with a threshold, rates in small groups are noisy.

---

## 7. Business Applications

- **Early-warning system:** Flag customers whose activity is falling or who are inactive for several months.
- **Targeted retention:** Tailor offers to high-churn segments instead of using blanket campaigns.
- **Resource prioritisation:** Focus effort where churn risk and customer value are both high.

---

## 8. Possible Extensions

- Build a churn prediction model (Logistic Regression, Random Forest, XGBoost) and evaluate it with precision, recall, F1 and ROC-AUC instead of accuracy.
- Handle imbalance with SMOTE or class weights.
- Test differences between groups for statistical significance (chi-square, t-test).
- Build a dashboard in Power BI or Tableau.

---

## 9. Tech Stack

Python (Pandas, NumPy, Matplotlib, Seaborn) | SQL (SQLite) | Jupyter Notebook

---

## 10. Glossary

| Term | Meaning |
|---|---|
| Churn | Customer leaving the service |
| EDA | Exploratory Data Analysis |
| Quartile | One of four equal-sized groups of ranked data |
| Data leakage | Using information that would not be available at prediction time |
| Class imbalance | One outcome far more common than the other |
| Aggregation | Summarising many rows into one value (COUNT, AVG) |

---

## Author

**[Your Name]** | [LinkedIn](https://linkedin.com/in/your-profile) | [GitHub](https://github.com/your-username)
