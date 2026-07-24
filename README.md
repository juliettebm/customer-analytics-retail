# Customer Analytics — Segmentation, Retention & Repurchase Prediction

End-to-end customer analytics pipeline on transactional e-commerce data: RFM segmentation, cohort retention with survival analysis (Kaplan-Meier, Cox proportional hazards), and leakage-safe repurchase prediction.

Built to demonstrate methods directly transposable between clinical time-to-event analysis and customer lifecycle analytics — the same survival-analysis toolkit (censoring, Kaplan-Meier, Cox regression) used elsewhere in this portfolio for oncology survival data is applied here to customer churn.

## Business context

Online Retail II: two years (Dec. 2009 – Dec. 2011) of transactions from a UK-based online retailer selling giftware, mostly to wholesale customers. The questions addressed:

1. Who are the most valuable customers, and how much of total revenue do they represent?
2. How does retention evolve after acquisition, and how should "churn" even be defined on non-contractual data with no cancellation event?
3. Can repurchase within a 90-day window be predicted from behaviour observed before that window, without leaking future information into the features?

## Repository structure

```
customer-analytics-retail/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/          # source .xlsx (not versioned — see Setup)
│   └── processed/    # generated Parquet files (not versioned)
└── notebooks/
    ├── 01_data_preparation.ipynb
    ├── 02_customer_segmentation.ipynb
    ├── 03_cohort_retention_churn.ipynb
    └── 04_repurchase_prediction.ipynb
```

Each notebook writes its output to `data/processed/` and reads only from what the previous notebook produced — they are meant to be run in order, once, from a clean environment.

## Setup

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS/Linux

pip install -r requirements.txt
```

Download the dataset from the [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/502/online+retail+ii) and place the `.xlsx` file at `data/raw/online_retail_II.xlsx`.

```bash
jupyter lab
```

Run the notebooks in order (01 → 04). Each one caches its output as Parquet, so re-running after the first pass takes seconds rather than minutes.

## Methodology and key decisions

**01 — Data preparation.** Every exclusion (cancellations, non-product stock codes, missing customer IDs, system test data) is flagged and quantified in both row count and revenue impact before being applied, and resolved by inspecting real rows rather than acting on aggregate counts alone — three non-product codes (`B`, `TEST001`, `TEST002`) would have been missed entirely by a naive alphabetic-code filter and only surfaced by cross-checking price/quantity anomalies against actual transactions.

**02 — Segmentation.** RFM scoring is built two ways — quantile-based business rules and k-means clustering — and cross-validated against each other rather than picking one as "correct." The two approaches agree completely at the extremes (best and worst customers) and diverge on the middle segments, which is reported explicitly rather than hidden. Segmentation stability is tested against two upstream modelling choices (reference date, cancellation-netting rule).

**03 — Retention & churn.** Churn is defined via an inactivity threshold derived from the actual distribution of inter-purchase intervals, not an arbitrary round number. Customers who haven't crossed the threshold by the end of the observation period are right-censored, not assumed loyal — the same logic used for overall-survival in a clinical dataset. The Cox model's proportional-hazards assumption is formally tested; when a covariate fails it, the model is corrected by stratification rather than reporting an invalid hazard ratio.

**04 — Repurchase prediction.** Features are computed strictly from an observation window that ends before the prediction window begins, and the train/test split is time-based rather than random, to prevent a customer's own future purchases from leaking into their training-time features.

## Key findings

- **Revenue is sharply concentrated.** ~22% of customers (the "Champions" segment) generate ~69% of revenue; at a coarser 2-cluster split, 40% of customers generate 88% of revenue.
- **First-order value does not predict retention.** Neither a log-rank test nor a stratified Cox model finds a significant effect of first-purchase spend or country on time-to-churn — purchase frequency is the dominant, genuine signal, and this null result is reported as-is rather than reframed.
- **A simpler model outperforms a more complex one.** Logistic Regression beats Random Forest on both ROC-AUC (0.79 vs. 0.78) and PR-AUC (0.755 vs. 0.752) for repurchase prediction — kept as the reference model rather than defaulting to the more complex algorithm.
- **Monetary value outranks purchase frequency** as a repurchase predictor (after Recency), a more actionable finding for targeting decisions than "recency matters most."

## Tech stack

Python · pandas · scikit-learn · lifelines (survival analysis) · matplotlib / seaborn · Jupyter

## Limitations

- The train/test split for repurchase prediction spans different seasons (summer training window vs. pre-Christmas test window), which measurably shifts the base repurchase rate and likely affects generalization to other times of year.
- The business-cost assumptions used for threshold optimization in notebook 04 are illustrative placeholders, not real campaign economics — the resulting threshold should not be read as an operational recommendation without re-deriving it from actual costs.
- The dataset is UK/European wholesale-heavy; customer behaviour patterns may not generalize to a purely B2C retail context.

## Author

Juliette Bouli-Mengue — [github.com/juliettebm](https://github.com/juliettebm)
