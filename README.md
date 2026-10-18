# CS5228 · Singapore HDB Rental Prediction

Run the three notebooks in order. The full dataset is used by default; the code and environment are designed for Python 3.12.

## File Layout

- `notebooks/01_eda.ipynb`: Raw-table inspection, rental distributions, temporal trends, categorical-value cleaning diagnostics, address matching, and auxiliary-data coverage.
- `notebooks/02_feature_engineering.ipynb`: Target-independent features, building matching, distances to amenities, monthly economic proxies, and a feature dictionary.
- `notebooks/03_model_training.ipynb`: Four-tier feature ablation, seven baselines/models, temporal validation, an independent holdout, final retraining, prediction, and submission-template generation.
- `requirements.txt`: Pins the main dependency versions.

Copy the following eight raw files (which all can be downloaded from the Kaggle website) into `data/raw/` under the project root:

`train.csv`, `test.csv`, `sg-hdb-block.csv`, `sg-mrt-stations.csv`,
`sg-shopping-malls.csv`, `sg-schools.csv`, `sg-coe-prices.csv`, `sg-stock-prices.csv`.

The six auxiliary tables may also be placed in `data/raw/auxiliary-data/`.
Raw files are not included in this delivery to avoid duplicate distribution and accidental modification. The notebooks do not overwrite raw data.

## Installation and Execution

```bash
conda create -n HDBRent python=3.12 pip -y
conda activate HDBRent
python -m pip install -r requirements.txt
python -m ipykernel install --user --name hdbrent --display-name "Python (HDBRent)"
```


Open the entire project folder in VS Code, install the Microsoft Python and Jupyter extensions, and select the `Python (HDBRent)` kernel.
Use Run All for notebooks 01 → 02 → 03 in that order. The first two notebooks are independent of each other; notebook 02 must be run before notebook 03.
The notebooks can be launched from either the project root or the notebooks folder, without changing any hard-coded machine-specific paths.

When integrating these files into an existing repository, retain any required content from its current README, add the notebooks folder and requirements.txt to the repository, and place the raw CSV files in data/raw.

## Dataset and Validation Design

- Training data: 150,000 rows, from 2021-01 to 2025-03; test data: 50,000 rows, from 2025-03 to 2026-07.
- Flat-type variants such as `4 room`/`4-room` are standardized. Addresses are normalized for Unicode, letter case, leading/trailing whitespace, and repeated spaces.
- `furnished` and `fee` are constant in the current training file and are excluded; duplicate public records are retained by default.
- Buildings are matched using Town + Block + Street. After normalization, matching coverage is 100% for the current files.
- Tuning: Fit on 2021–2022 data and validate on 2023 data to select models, parameters, and features.
- Independent holdout: Retrain the fixed configuration on data through 2023-12 and evaluate on 2024-01 through 2025-03.
- Final training: Retrain the fixed configuration on all labeled records and generate predictions in the original test-row order.

RMSE is the default metric for model selection; MAE, RMSLE, and R² are also reported.

## Information Availability

No label-based aggregate features are constructed using the full dataset. Imputation, encoding, and scaling are refitted for each training run.
The static landmark tables do not include historical opening or operating dates. The corresponding features represent the spatial snapshot in the supplied files; they do not claim to accurately reflect historical amenity availability.
Macroeconomic data use completed months only. By default, fixed_origin freezes macroeconomic information for validation and future periods at the training-end month of each fold.
rolling_monthly is an optional alternative assumption, to be used only when newly available economic data may be incorporated each month; it does not access test-set rents.

## Optional Settings and Computational Cost

In the configuration cell of notebook 03:

- `QUICK_MODE=False` is the default. When True, fitting and evaluation data are sampled for execution checks only, and outputs are written to the quick directory.
- `RUN_ABLATION=True` compares four feature tiers—core / building / amenities / macro—using the same fixed CatBoost configuration.
- `RUN_OPTUNA=False`: Enabling this adds 10 CatBoost fits by default; adjust OPTUNA_TRIALS as needed.
- `RUN_ROLLING_BACKTEST=False`: Enabling this adds retrospective stability diagnostics, rather than a new independent test.
- `RUN_SHAP=False`: Enabling this additionally explains a small subset of validation samples for the selected tree model.
- `N_JOBS` is capped at 4 by default. Adjust it based on your CPU and memory. The full experiment includes multiple tree-model training runs, so execution time depends on hardware.
- Random forest excludes the two high-cardinality address categories, building_id / street, while retaining location inputs such as latitude/longitude, region, and postal district. Other models use all features in the selected tier. The leaderboard records the actual number of input features.

## Outputs

- `data/processed/`: Training and test features, labels, macroeconomic tables, a field dictionary, input hashes, and schemas.
- `outputs/eda/`: Data-inspection summaries.
- `outputs/training/full_fixed_origin/`: Ablation results, the validation leaderboard, independent-holdout metrics, detailed errors, models, and run metadata.
- `submissions/full_fixed_origin/test_predictions.csv`: row_id and MONTHLY_RENT in the original row order; row_id is an internal row index for verification.
- Official submission: Download the actual `example-submission.csv`, place it in data/raw, and rerun the submission-template cell in notebook 03. The code preserves all other template columns and their order.

## Reference Results from Completed Runs

`reference_results/` contains results from the full-data run completed before delivery, along with row-level test predictions, for immediate inspection. It is neither an input-data directory nor the official Kaggle submission format. See VALIDATION.md for the run and validation records.
