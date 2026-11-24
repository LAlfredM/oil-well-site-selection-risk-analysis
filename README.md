# Oil Well Site Selection: Predictive Modeling & Bootstrap Risk Analysis

A data-driven approach to choosing the best region for developing 200 new oil wells, combining regional regression models with bootstrap simulation to quantify expected profit and the risk of loss.

## Business Problem

**OilyGiant**, an oil extraction company, needs to decide where to drill its next 200 wells. Geological survey data is available for three candidate regions, and the decision must satisfy the following constraints:

| Parameter | Value |
| :--- | :--- |
| Exploration sample per region | 500 candidate points |
| Wells developed | Top 200 by predicted reserves |
| Development budget | 100,000,000 USD (500,000 USD per well) |
| Revenue | 4,500 USD per thousand barrels |
| Break-even | ≈ 111.11 thousand barrels per well on average |
| Risk tolerance | Probability of loss **strictly below 2.5%** |

Among the regions that meet the risk threshold, the one with the highest average expected profit is selected.

## Approach

1. **Data audit & EDA**: schema checks, missing values, duplicate rows and repeated well IDs, summary statistics, reserve distributions, and feature-target correlations for each region.
2. **Predictive modeling**: one `LinearRegression` model per region (75:25 train/validation split, `random_state=12345`), evaluated with RMSE on the validation set.
3. **Deterministic profit benchmark**: net profit from the 200 highest-predicted wells, calculated with reusable selection and profit functions.
4. **Bootstrap risk simulation**: 1,000 resampling iterations of 500 candidate wells each, selecting the top 200 by predicted reserves and recording realized profit. Reports the mean profit, 95% confidence interval, and probability of loss.

## Key Results

| Metric | Region 0 | Region 1 | Region 2 |
| :--- | :---: | :---: | :---: |
| Validation RMSE (thousand barrels) | 37.58 | **0.89** | 40.03 |
| Deterministic top-200 profit (million USD) | **33.21** | 24.15 | 27.10 |
| Bootstrap mean profit (million USD) | 3.962 | **4.612** | 3.930 |
| 95% confidence interval (million USD) | [-1.112, 9.098] | **[0.781, 8.630]** | [-1.122, 9.346] |
| Probability of loss | 6.90% | **0.70%** | 6.50% |
| Meets the < 2.5% risk threshold | No | **Yes** | No |

## Recommendation

**Region 1** is the recommended location.

- It is the only region whose probability of loss (0.70%) falls below the 2.5% threshold, and its 95% confidence interval stays above zero.
- It also has the highest bootstrap mean profit (4.612 million USD).
- The `f2` feature is almost perfectly correlated with reserves in this region (r = 0.999), which gives the model very accurate predictions (RMSE of 0.89 thousand barrels) and makes the ranking of candidate wells far more reliable than in Regions 0 and 2.

The deterministic benchmark, which ranks wells across the full 25,000-observation validation set, favors Region 0. That benchmark does not reflect the real exploration process, where only 500 candidate points are surveyed before 200 wells are chosen. Once that constraint and the risk threshold are applied through bootstrapping, Region 1 is the only viable choice.

## Repository Contents

```
├── oil_well_site_selection.ipynb   # Full analysis notebook
└── README.md
```

## Data

The notebook expects three CSV files, one per region: `geo_data_0.csv`, `geo_data_1.csv` and `geo_data_2.csv`. Each contains 100,000 records with these columns:

| Column | Description |
| :--- | :--- |
| `id` | Unique well identifier |
| `f0`, `f1`, `f2` | Geological features of the survey point |
| `product` | Reserve volume in thousands of barrels (target) |

Place the files in the same folder as the notebook, or in a `datasets/` subfolder.

## Getting Started

```bash
pip install numpy pandas matplotlib seaborn scikit-learn jupyter
jupyter notebook oil_well_site_selection.ipynb
```

## Tech Stack

Python · pandas · NumPy · scikit-learn · matplotlib · seaborn

## Limitations

The bootstrap results are empirical estimates based on 1,000 resamples of the available validation data. They quantify sampling variability but do not guarantee future field performance. Before committing capital, the geological assumptions, operating costs, commodity price assumptions, and model stability should be validated with additional field and financial due diligence.
