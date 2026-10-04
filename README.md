# drive-failure-predictor

Predicts hard drive and SSD failures from SMART telemetry. Trained and benchmarked on open datasets such as Backblaze, with calibrated risk scores so data centers can replace drives before data is lost.

> **Status:** early development. The data pipeline, models and benchmarks are being built in the open. See the [Roadmap](#roadmap).

---

## Why this exists

Storage drives are among the most common parts to fail in a data center. A sudden failure can mean lost data, degraded service and emergency repairs. Drives report their own health through **SMART** (Self-Monitoring, Analysis and Reporting Technology) attributes such as reallocated sectors, pending sectors and temperature, but turning these numbers into a reliable warning is hard:

- **Failures are rare.** Only a small fraction of drives fail in a given period, so most naive models look accurate while missing the failures.
- **Many failures give little warning.** Some drives fail with no SMART signal at all.
- **Evaluation is often flawed.** Random train and test splits let the model see the future or the same drive in both sets, which makes results look better than they are.
- **Results are hard to compare.** Different papers use different drive models, time windows and metrics.
- **Raw scores are hard to act on.** Operators need to know "how likely is this drive to fail in the next 30 days?" and whether that number can be trusted.

`drive-failure-predictor` aims to provide a clean, reproducible starting point:

- **Unified data loaders** for open drive telemetry.
- **Time-based and drive-based splits** so evaluation reflects real use.
- **Several model families** compared under the same protocol.
- **Calibrated risk scores,** so a score of 0.2 really means about a 20% chance.
- **Operations-focused metrics:** warning lead time, false alarms and missed failures.
- **One command** to reproduce every reported result.

---

## What it predicts

| Output | Meaning |
|---|---|
| **Failure risk** | Probability that a drive fails within the next N days (for example, 7 or 30) |
| **Risk ranking** | A list of drives ordered by risk, so limited replacement effort goes to the right drives |
| **Lead time** | How many days before failure a useful alert is raised |
| **Top signals** | Which SMART attributes and trends drove the risk score |

---

## Data

The project is built around openly available drive telemetry, mainly:

- **Backblaze Drive Stats:** daily SMART data and failure labels for a large fleet of hard drives and SSDs, published openly by Backblaze

Other open datasets may be added later. Data is **not** redistributed in this repository. Scripts download it from the original source, and the dataset keeps its own license and citation requirements.

Note that the data covers specific drive makes and models in one company's data centers, so results may not carry over to other fleets.

---

## Planned approach

### Models compared

- **Baselines:** threshold rules on a few well-known SMART attributes
- **Classical ML:** logistic regression, random forest, gradient boosting on engineered features (rolling averages, recent changes, days since first anomaly)
- **Survival analysis:** time-to-failure models that handle drives that have not failed yet
- **Deep learning:** LSTM / GRU sequence models on daily telemetry windows

### Evaluation design

- **Time-based split:** train on earlier months, test on later months
- **Drive-based split:** a drive never appears in both train and test
- **Per-model-family results:** drive models behave differently, so results are reported separately

### Metrics

- **Precision and recall** at a fixed false-alarm rate
- **Area under the precision-recall curve** (better than accuracy for rare events)
- **Warning lead time:** days between the alert and the failure
- **Calibration:** do predicted probabilities match observed failure rates?
- **Cost view:** a simple cost model comparing the cost of a missed failure with the cost of an unnecessary replacement

---

## How it works

```
daily SMART telemetry + failure labels
                 |
                 v
     cleaning + per-drive time series
                 |
                 v
  features (rolling stats, trends, recent changes)
                 |
                 v
   leakage-safe split (by time and by drive)
                 |
                 v
   model (rules / ML / survival / deep learning)
                 |
                 v
 calibrated failure risk + ranking + top signals
                 |
                 v
 metrics: lead time, false alarms, missed failures
```

---

## Planned repository structure

```
drive-failure-predictor/
├── data/
│   ├── raw/              # downloaded datasets (git-ignored)
│   └── processed/        # cleaned per-drive time series
├── drive_failure/
│   ├── datasets/         # loaders for open telemetry
│   ├── features.py       # feature engineering
│   ├── models/           # baselines, ML, survival, deep models
│   ├── calibration.py    # probability calibration
│   └── evaluate.py       # metrics and splits
├── benchmarks/           # scripts and saved results
├── notebooks/            # exploration and plots
├── tests/
├── docs/
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## Quickstart (planned)

```bash
git clone https://github.com/amoltathesoftwareengineer/drive-failure-predictor.git
cd drive-failure-predictor
pip install -r requirements.txt

# download and prepare a quarter of Backblaze data
python -m drive_failure.datasets.download --source backblaze --quarter 2024Q1

# train a model and print metrics
python -m drive_failure.train --model gbm --horizon 30

# reproduce the benchmark table
python -m benchmarks.run_all
```

---

## Results

Results will be added once the first benchmark is complete.

| Model | Horizon | Precision at 1% false alarms | Average lead time | Calibration error |
|---|---|---|---|---|
| _coming soon_ | - | - | - | - |

---

## Roadmap

- [ ] Write dataset download and loader scripts
- [ ] Build per-drive time series and feature pipeline
- [ ] Implement time-based and drive-based splits
- [ ] Add threshold and classical ML baselines
- [ ] Add survival analysis models
- [ ] Add deep learning models
- [ ] Add probability calibration and cost-based evaluation
- [ ] Publish the first reproducible benchmark
- [ ] Add a simple dashboard for fleet risk ranking
- [ ] Add SSD-specific attributes and more datasets

---

## Limitations and responsible use

- Some drives fail without any warning in SMART data. No model can predict those failures from this data alone.
- Models trained on one fleet and set of drive models may not work on another. Retrain and validate on your own data before relying on predictions.
- A prediction should guide **replacement planning and backups,** not replace them. Never rely on a model alone to protect important data.
- This project is for research and education. It is not a certified storage monitoring product.

---

## Contributing

Contributions are welcome, especially:

- New datasets and loaders
- New models and baselines
- Better calibration and cost-based evaluation
- Bug reports and reproducibility fixes

Please open an issue to discuss before sending a large pull request.

---

## License

MIT License. See [LICENSE](LICENSE).

---

## Author

**Amol Tathe**
[LinkedIn](https://linkedin.com/in/amoltathesoftwareengineer) · [GitHub](https://github.com/amoltathesoftwareengineer)
