# Hotel Bookings Data Cleaning, Preprocessing, and Statistical Analysis Notebook

## Overview

**Project_1_Data_Cleaning_&_Preprocessing_Challenge.ipynb**, performs data cleaning, preprocessing, exploratory data analysis (EDA), and statistical analysis on a hotel bookings dataset. The primary focus is on understanding factors influencing booking cancellations, using the "hotel_bookings.csv" dataset (commonly sourced from public repositories like Kaggle). The analysis includes handling missing values, encoding categorical variables, computing correlations, hypothesis testing, effect sizes, and multicollinearity checks.

The notebook aims to:
- Clean and preprocess the data for analysis.
- Conduct EDA to summarize key statistics and visualize data quality.
- Perform statistical tests to identify significant differences between canceled and non-canceled bookings.
- Interpret results to provide insights into cancellation drivers, such as lead time, average daily rate (ADR), and customer segments.

This README explains the notebook from start to finish ("A to Z"), including code explanations, interpretations of EDA and statistical outputs, and key insights. The notebook is written in Python using libraries like Pandas, Matplotlib, Seaborn, Missingno, SciPy, and Statsmodels.

## Dataset Description

The dataset contains 119,390 rows and 32 columns representing hotel booking records from two types of hotels (City Hotel and Resort Hotel). Key columns include:
- **hotel**: Type of hotel (City or Resort).
- **is_canceled**: Binary indicator (1 = canceled, 0 = not canceled).
- **lead_time**: Days between booking and arrival.
- **adr**: Average daily rate.
- **market_segment**: Booking segment (e.g., Online TA, Direct).
- **distribution_channel**: Booking channel (e.g., TA/TO, Direct).
- **deposit_type**: Deposit policy (e.g., No Deposit, Non Refund).
- Other features: Arrival details, guest counts (adults, children, babies), room types, special requests, etc.

The target variable is `is_canceled`, with an overall cancellation rate of approximately 27.49%. The data spans 2015–2017 and includes missing values in columns like `agent`, `company`, `country`, and `children`.

## Requirements

- Python 3.x
- Jupyter Notebook or Google Colab
- Libraries: 
  - pandas
  - matplotlib
  - seaborn
  - missingno
  - scipy
  - statsmodels

Install via pip:  
```
pip install pandas matplotlib seaborn missingno scipy statsmodels
```

## How to Run

1. Download the notebook and the dataset ("hotel_bookings.csv") to the same directory.
2. Open the notebook in Jupyter or Google Colab.
3. Run cells sequentially. Note: The dataset must be in the working directory for `pd.read_csv("hotel_bookings.csv")` to work.
4. Outputs include printed summaries, statistical results, and (implied) visualizations like missing value matrices (embedded as base64 images in the provided document).

## Notebook Structure and Step-by-Step Explanation

The notebook is divided into sections: Data Loading, Data Cleaning, and Statistical Analysis. Below is a detailed walkthrough.

### 1. Data Loading and Initial Inspection (Cell 1)

- **Code Explanation**:
  - Import necessary libraries: `pandas` for data manipulation, `matplotlib.pyplot` and `seaborn` for visualization, `missingno` for missing value analysis.
  - Load the dataset: `df = pd.read_csv("hotel_bookings.csv")`.

- **Outputs and Interpretation**:
  - No explicit outputs here, but this sets up the DataFrame `df` for subsequent analysis.

### 2. Data Cleaning Section (Cells 2–3)

- **Code Explanation** (Cell 2):
  - Print dataset shape: `(119390, 32)`.
  - Print data info: Shows data types (20 numerical, 12 categorical) and non-null counts, highlighting missing values (e.g., `company` has only 6,797 non-null out of 119,390).
  - Display descriptive statistics: `df.describe(include="all")` summarizes numerical (mean, std, min/max) and categorical (unique values, top/freq) features.
    - Example insights: Mean lead_time = 104 days, mean ADR = 101.83, most common market_segment = "Online TA".

- **Outputs and EDA Interpretation**:
  - **Shape and Info**: Confirms a large dataset with missing values in `children` (4 missing), `country` (488 missing), `agent` (16,340 missing), and `company` (112,593 missing—likely because most bookings are individual, not corporate).
  - **Descriptive Statistics**:
    - Numerical: Lead time ranges 0–737 days (outliers possible); ADR has a negative min (-6.38, potential error); stays_in_week_nights up to 50 (unusual but possible).
    - Categorical: 2 hotel types (City Hotel: 79,330 bookings); 12 months (August peak); 178 countries (PRT top).
    - **EDA Insights**: Data is skewed (e.g., ADR std=50.54); high missingness in `company` suggests it may be droppable or imputable as 0. Visualizations (implied by missingno import and base64 image) likely include a missing values matrix/bar, showing sparsity in `company` and `agent`. This EDA reveals data quality issues like negatives in ADR and zeros in guest counts, warranting cleaning (though not fully implemented here—e.g., no imputation shown).

- **Cell 3**: Markdown header for "Statistical Analysis" (transition to next section).

### 3. Statistical Analysis Section (Cells 4–11)

This section focuses on feature engineering, correlations, hypothesis testing, effect sizes, and multicollinearity.

- **One-Hot Encoding (Cell 4)**:
  - **Code**: Encodes categorical columns like `hotel`, `arrival_date_month`, etc., into dummies (drop_first=True to avoid multicollinearity).
  - **Interpretation**: Prepares data for correlation and modeling by converting categories to binary features (e.g., `hotel_Resort Hotel`).

- **Correlation Analysis (Cell 4, repeated in Cell 9)**:
  - **Code**: Computes Pearson correlations with `is_canceled` and sorts them.
  - **Outputs and Interpretation**:
    - Positive correlations: `market_segment_Online TA` (0.21), `lead_time` (0.18), `deposit_type_Non Refund` (0.17)—suggest these increase cancellations.
    - Negative correlations: `required_car_parking_spaces` (-0.18), `market_segment_Offline TA/TO` (-0.12), `total_of_special_requests` (-0.12)—suggest these reduce cancellations.
    - **EDA Insights**: Cancellations are moderately linked to online bookings and longer lead times, but weakly overall (no |r| > 0.3). This implies multifactorial influences; no single feature dominates.

- **Baseline Cancellation Rate (Cell 5)**:
  - **Code**: `cancel_rate = df["is_canceled"].mean()` → 27.49%.
  - **Interpretation**: Establishes a benchmark—nearly 1 in 4 bookings cancel, highlighting a business problem (revenue loss).

- **Normality and Mann-Whitney U Test for ADR (Cell 6)**:
  - **Code**: Samples 500 from canceled/not_canceled ADR; Shapiro-Wilk test for normality (p < 0.001 for both, non-normal). Then Mann-Whitney U (non-parametric) test.
  - **Outputs**: p-value = 0.0000 (significant difference).
  - **Interpretation**: ADR distributions are skewed (non-normal, as expected for prices). Canceled bookings have significantly different ADR (likely higher, per correlations), suggesting price sensitivity or speculative high-rate bookings.

- **Mann-Whitney U Tests for Other Numerical Features (Cell 7)**:
  - **Features**: lead_time, adr, total_guests, total_nights.
  - **Outputs**: All p-values = 0.0000 (significant).
  - **Interpretation**: Canceled bookings differ significantly in these metrics. Longer lead times and higher ADR/guest counts may correlate with cancellations (e.g., more time to cancel, larger groups more volatile).

- **Chi-Squared Tests for Categorical Features (Cell 8)**:
  - **Features**: hotel, meal, market_segment, distribution_channel, deposit_type.
  - **Outputs**: All p-values < 0.0001 (significant associations).
  - **Interpretation**: Cancellation rates vary by category (e.g., higher for Online TA/TO channels, non-refundable deposits). This supports segmentation strategies (e.g., target direct bookings to reduce cancellations).

- **Cohen's d Effect Sizes (Cell 10)**:
  - **Code**: Custom function for Cohen's d (standardized mean difference).
  - **Outputs**: lead_time (0.42, medium), adr (0.30, small-medium), total_guests (0.23, small), total_nights (0.19, small).
  - **Interpretation**: Practical significance—lead_time has the largest effect, meaning canceled bookings have notably longer lead times (business implication: early bookings riskier).

- **Variance Inflation Factor (VIF) for Multicollinearity (Cell 11)**:
  - **Code**: Computes VIF for numerical features.
  - **Outputs**: All VIF < 7 (low multicollinearity; total_guests highest at 6.38).
  - **Interpretation**: Features are independent enough for modeling (VIF < 10 threshold). No need to drop variables due to collinearity.

## Key Insights and Interpretations

- **EDA Summary**: The dataset is large but has quality issues (missing values, outliers like negative ADR). Visuals (e.g., missingno matrix) highlight sparsity; descriptives show seasonal peaks (August) and dominance of City Hotel bookings.
- **Statistical Analysis Interpretations**:
  - Cancellations are driven by lead time, online segments, and non-refundable deposits (positive correlations/associations).
  - Protective factors: Parking requests, special requests, offline/direct channels (negative correlations).
  - All tested features show significant differences (p < 0.001), but effect sizes are small-medium, indicating subtle influences.
  - Business Implications: Hotels could reduce cancellations via flexible policies for long-lead bookings, incentives for direct channels, or targeted pricing for high-ADR segments.
- **Limitations**: No handling of missing values/outliers shown; assumes cleaned data for stats. Non-parametric tests appropriate for skewed data, but causal inference requires modeling (e.g., logistic regression).

## Conclusions

This notebook provides a comprehensive pipeline from data loading to advanced stats, revealing actionable insights on hotel booking cancellations. Future extensions: Full cleaning (imputation, outlier removal), visualizations (e.g., heatmaps), and predictive modeling.

For questions, contact [Your Name/Email]. Contributions welcome!

Last Updated: September 05, 2025.
