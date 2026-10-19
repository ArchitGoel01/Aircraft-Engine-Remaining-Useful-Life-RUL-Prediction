# Aircraft Engine Remaining Useful Life (RUL) Prediction

An academic machine-learning project that estimates the Remaining Useful Life (RUL)—the number of operating cycles remaining—of turbofan engines using time-series sensor data from the **NASA C-MAPSS FD001** dataset. The project uses an LSTM neural network and includes a Random Forest baseline in the accompanying project report.

> **Important:** This is an educational research project, not a certified aircraft-maintenance system. Predictions must not be used to make real-world aviation safety or maintenance decisions.

## Project at a glance

- **Task:** Regression / predictive maintenance
- **Dataset:** NASA C-MAPSS FD001 turbofan engine degradation data
- **Sequence length:** 30 operating cycles
- **Input features:** 17 operational-setting/sensor features after removing constant columns
- **Target:** Remaining Useful Life, capped at 125 cycles for capped-target experiments
- **Deep-learning model:** Keras LSTM
- **Metrics:** MAE, RMSE; R² is also available for the included prediction CSV

## Repository structure

```text
.
├── data/
│   └── cmapss_fd001_converted.xlsx  # Converted FD001 train/test/reference-RUL tables
├── docs/
│   └── project_report.docx          # Academic project report
├── models/
│   └── aircraft_rul_lstm.keras      # Saved trained Keras model
├── notebooks/
│   └── colab_links.md               # Links to the two original Colab notebooks
├── results/
│   └── test_predictions.csv         # Actual and predicted RUL for 100 test engines
├── .gitignore
├── requirements.txt
└── README.md
```

## Dataset

The converted workbook contains three sheets:

- `Train_FD001`: training trajectories
- `Test_FD001`: test trajectories
- `RUL_FD001`: reference RUL labels for the test engines

The report describes 20,631 training records from 100 engines and 13,096 test records from 100 engines. The original NASA C-MAPSS data is a simulated benchmark dataset. Please review the dataset's original NASA terms and citation requirements before redistribution. This repository contains a converted workbook supplied with the project; it is not a replacement for the official source.

## Methodology

1. Load and inspect the FD001 engine-cycle data.
2. Check data quality and identify constant-valued columns.
3. Generate training RUL labels using `RUL = final_cycle_for_engine - current_cycle`.
4. For capped-target experiments, apply `RUL_capped = min(RUL, 125)`.
5. Remove seven constant features, leaving 17 model inputs.
6. Fit a MinMax scaler on training engines and use it to transform validation/test data.
7. Split engines into training and validation groups (80/20 in the report).
8. Create rolling windows of 30 consecutive cycles.
9. Train an LSTM regressor and compare it with a Random Forest baseline.
10. Evaluate predictions with MAE and RMSE; inspect actual-versus-predicted values and residuals.

### Saved model architecture

The supplied `.keras` model file records the following architecture:

- Input shape: `(30, 17)`
- LSTM layer: 64 units
- Dropout: 0.10
- Dense layer: 32 units with ReLU activation
- Output layer: 1 unit with linear activation
- Loss: mean squared error (MSE)
- Optimizer: Adam (the saved optimizer learning rate is approximately `3.125e-05`)

The report describes more than one model/training configuration in different sections. The architecture above is read from the **actual saved model artifact**. For reproducibility, the training notebook should be treated as the source of truth for the full preprocessing and training configuration.

## Results

The project report records the following evaluation results:

| Evaluation | Model / target | MAE (cycles) | RMSE (cycles) |
|---|---|---:|---:|
| Validation | LSTM, capped RUL | 8.55 | 11.91 |
| Validation | Random Forest, capped RUL | 13.08 | 17.56 |
| Test | LSTM, capped RUL | 9.70 | 13.03 |
| Test | LSTM, original RUL | 10.77 | 14.39 |

The supplied `results/test_predictions.csv` contains one row per test engine with `Engine ID`, `Actual RUL`, and `Predicted RUL`. Recalculating metrics directly from that CSV gives:

| Target comparison | MAE (cycles) | RMSE (cycles) | R² |
|---|---:|---:|---:|
| Original RUL values in CSV | 10.79 | 14.42 | 0.880 |
| Capped actual RUL and predictions clipped to 0–125 | 9.70 | 13.03 | — |

Small differences from the report's original-target values may arise because the CSV and report may reflect different prediction runs or post-processing. The capped metrics agree closely. When publishing, describe the metric source and target definition clearly rather than mixing these values.

## Open the notebooks

The original work was developed in Google Colab. Links are listed in [`notebooks/colab_links.md`](notebooks/colab_links.md). Make sure the sharing permissions allow viewers to open them. The `.url` shortcut files alone do not contain notebook source code, so the notebooks are linked rather than represented as editable `.ipynb` files in this repository.

## Environment and dependencies

Suggested environment: Python 3.10–3.12. The saved model was created with Keras 3.13.2; TensorFlow/Keras compatibility can vary by version.

Install dependencies:

```bash
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# macOS/Linux:
# source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
```

The repository currently includes the trained model and prediction output, but **does not include the original training/inference Python source or the fitted scaler artifact**. Consequently, a fully reproducible local retraining or raw-sensor-to-prediction pipeline is not yet self-contained. Export the Colab notebook(s) as `.ipynb` and save the exact fitted scaler/preprocessing configuration to complete that part before claiming one-command reproducibility.

## Files

- [Project report](docs/project_report.docx)
- [Saved LSTM model](models/aircraft_rul_lstm.keras)
- [Test predictions](results/test_predictions.csv)
- [Converted dataset workbook](data/cmapss_fd001_converted.xlsx)
- [Google Colab notebook links](notebooks/colab_links.md)

## Team and acknowledgement

Project report submitted to **Thapar Institute of Engineering and Technology**, under the guidance of **Dr. Jyoti**, academic year 2026–2027.

The report lists the project team as Mehul Mishra, Ananya Nagpal, Sumedha Sharma, Ansh Sharma, and Archit Goel.

Dataset acknowledgement: NASA C-MAPSS turbofan engine degradation simulation dataset. Please cite and follow the usage terms of the original dataset source when appropriate.
