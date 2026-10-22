# Glassdoor Tech Job Market & Salary Analysis (Python & Pandas)

## 📌 Project Overview
This project focuses on end-to-end **Data Cleaning, Feature Engineering, and Exploratory Data Analysis (EDA)** using Python (`pandas` and `numpy`) on a real-world dataset of **956 tech job postings** scraped from Glassdoor. 

Raw web-scraped job data contains significant data quality issues—including duplicate job listings, text-heavy salary strings, inconsistent geographic formatting, and multi-format missing value placeholders (`-1`, `'Unknown'`, `'Unknown / Non-Applicable'`). This project transforms the raw dataset into an analysis-ready format to uncover compensation benchmarks across US states, core data roles, industry sectors, and company attributes.

---

## 🛠️ Tech Stack & Tools
* **Language:** Python 3
* **Libraries:** `pandas`, `numpy`, Regular Expressions (`re`)
* **Environment:** VS Code, Jupyter Notebook, Git/GitHub

---

## 🧹 Data Cleaning & Feature Engineering Pipeline

Followed a structured **Inspect $\rightarrow$ Clean $\rightarrow$ Transform $\rightarrow$ Verify** workflow across all 15 raw columns:

1. **Deduplication & Index Hygiene:**
   * Dropped the redundant `Unnamed: 0` CSV index column to expose true row-level duplicates.
   * Identified and removed **356 duplicate job postings** (~37.2% of the raw dataset), leaving **600 unique job listings**, and reset the DataFrame index (`drop=True`).

2. **Salary Parsing & Numeric Casting (`Salary Estimate` $\rightarrow$ `Min Salary`, `Max Salary`, `Avg Salary`):**
   * Stripped non-numeric text and currency/unit symbols (e.g., `"$53K-$91K (Glassdoor est.)"`) using regex (`r'[^\d\-]'`).
   * Replaced `'-1'` missing-value placeholders (**214 raw rows / 133 unique rows**) with `np.nan`.
   * Split salary ranges into `Min Salary` and `Max Salary` using `.str.split('-', expand=True)`, cast both to `float64` via `pd.to_numeric()`, and engineered an **`Avg Salary`** target metric (in `$K`).

3. **Company Name Cleaning (`Company Name`):**
   * Removed trailing newline characters and attached rating numbers (e.g., `"Tecolote Research\n3.8"` $\rightarrow$ `"Tecolote Research"`) using `.str.split('\n').str[0]`.

4. **Geographic Standardization (`Location` & `Headquarters` $\rightarrow$ `Job State`, `HQ State`, `Same State`):**
   * Inspected 237 unique location strings, identifying edge cases including multi-comma entries (`"Santa Fe Springs, Los Angeles, CA"`) and unformatted full state/country names (`"Michigan"`, `"Maryland"`, `"Virginia"`, `"New Jersey"`, `"Oregon"`, `"United States"`).
   * Extracted state codes via reverse-index splitting (`.str.split(',').str[-1].str.strip()`), standardized full state names to 2-letter USPS abbreviations, and mapped country-level entries (`"United States"`) to `np.nan`.
   * Cleaned `Headquarters` by converting **7** `'-1'` entries to `np.nan`, extracted `HQ State` (preserving international headquarters), and engineered a binary **`Same State`** feature (`1` if the job is in the headquarters state, `0` if in a satellite office).

5. **Company Metrics & Categorical Missing Value Standardization:**
   * Audited `Rating` (valid range: `1.9` to `5.0`) and `Founded` (valid range: `1744` to `2019`), converting **21** and **62** `-1` placeholders to `np.nan`, respectively.
   * Engineered **`Company Age`** (`2026 - Founded`), yielding a median company age of **30 years** (mean: **49.7 years**).
   * Standardized missing values across all categorical features (`Size`, `Type of ownership`, `Industry`, `Sector`, `Revenue`, `Competitors`) by replacing `'-1'`, `'Unknown'`, and `'Unknown / Non-Applicable'` with `np.nan`.

6. **Role Classification (`Job Title` $\rightarrow$ `Role Category`):**
   * Categorized messy job titles into core data professions (**`Data Scientist`**, **`Data Engineer`**, **`Data Analyst`**, and **`Other`**) using case-insensitive keyword matching with `.loc[]` and `.str.contains()`.

---

## 📊 Key Analytical Findings (EDA)

*(Note: Salary aggregations are calculated across the **467 unique job postings** with disclosed salary estimates).*

### 1. Top Paying US States (Minimum 5 Job Postings)
Filtering out low-sample states (`Job_Count >= 5`) reveals that **California (`CA`)** leads both in hiring volume and average compensation:

| Job State | Job Count | Mean Salary ($K) |
| :--- | :---: | :---: |
| **CA** (California) | 99 | **$124.4K** |
| **IL** (Illinois) | 22 | **$112.3K** |
| **DC** (Washington, D.C.) | 9 | **$110.2K** |
| **MA** (Massachusetts) | 60 | **$104.8K** |
| **NJ** (New Jersey) | 13 | **$102.3K** |
| **NY** (New York) | 47 | **$100.2K** |
| **NC** (North Carolina) | 11 | **$99.2K** |
| **MD** (Maryland) | 22 | **$99.0K** |
| **VA** (Virginia) | 30 | **$97.7K** |
| **PA** (Pennsylvania) | 19 | **$92.3K** |

### 2. Compensation by Core Data Role
**Data Scientists** represent the largest share of job postings and command the highest average pay, followed by **Data Engineers**:

| Role Category | Total Jobs | Avg Salary ($K) | Min Salary ($K) \vert{} Max Salary ($K) |
| :--- | :---: | :---: | :---: | :---: |
| **Data Scientist** | 210 | **$116.6K** | $15.5K | $254.0K |
| **Data Engineer** | 74 | **$103.5K** | $60.5K | $205.0K |
| **Other Tech / Analytics Roles** | 111 | **$91.2K** | $20.0K | $232.5K |
| **Data Analyst** | 72 | **$68.1K** | $13.5K | $173.0K |

### 3. Top Hiring Industry Sectors
**Information Technology**, **Business Services**, and **Biotech & Pharmaceuticals** drive over 54% of hiring. Notably, **Biotech & Pharmaceuticals** offers a higher average salary (**$113.4K**) than traditional **IT** (**$110.5K**):

| Sector | Job Postings | Avg Salary ($K) |
| :--- | :---: | :---: |
| **Information Technology** | 124 | **$110.5K** |
| **Business Services** | 70 | **$95.5K** |
| **Biotech & Pharmaceuticals** | 62 | **$113.4K** |
| **Insurance** | 41 | **$101.8K** |
| **Health Care** | 25 | **$77.7K** |
| **Finance** | 25 | **$98.1K** |
| **Aerospace & Defense** | 20 | **$99.3K** |
| **Manufacturing** | 18 | **$86.2K** |
| **Education** | 14 | **$102.5K** |
| **Retail** | 8 | **$104.8K** |

### 4. Impact of Headquarters Location, Company Rating, & Age on Pay
* **HQ vs. Satellite Office (`Same State`):** Working in the same state as a company's headquarters (`313` jobs, **$100.6K** avg) vs. a satellite office (`154` jobs, **$101.7K** avg) shows virtually no difference in pay.
* **Correlation Analysis:**
  * `Avg Salary` vs. `Rating` (**`r = 0.115`**): Weak positive correlation—higher-rated companies pay only marginally more than lower-rated ones.
  * `Avg Salary` vs. `Company Age` (**`r = 0.008`**): Near-zero correlation—company age does not influence salary levels.
  * `Rating` vs. `Company Age` (**`r = -0.192`**): Slight negative correlation, indicating younger companies tend to hold slightly higher Glassdoor ratings than older enterprises.

---

## 📁 Repository Structure
```text
glassdoor-salary-analysis/
├── data/
│   ├── glassdoor_jobs.csv           # Raw dataset (956 rows, 15 columns)
│   └── glassdoor_jobs_cleaned.csv   # Cleaned dataset (600 unique rows, 20 columns)
├── notebooks/
│   └── tech_salary_analysis.ipynb   # Full cleaning, feature engineering, & EDA notebook
└── README.md                        # Project documentation & findings
