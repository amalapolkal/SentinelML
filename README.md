# SentinelML — Machine Learning-Based Network Intrusion Detection and Threat Analytics Platform

**IBM SkillsBuild Cyber Security BootCamp 2026 · Organised by BharatCares**

SentinelML is an **educational** machine-learning prototype. It trains Random Forest and Logistic Regression models on a public, labelled network-intrusion dataset, classifies connection records as **Normal** or **Attack**, names the attack family (DoS / Probe / R2L / U2R), evaluates the models on held-out data, and presents threat-analytics charts, with an optional Streamlit dashboard.

> ⚠️ **Disclaimer:** SentinelML works on recorded, labelled research data only. It does **not** monitor live networks, perform scanning or attacks, prove a threat exists on your computer, or guarantee attack detection. It is not a production security tool.

## Problem statement and objectives
Manually reviewing network traffic records is slow and needs expert knowledge. SentinelML shows how machine learning can classify traffic, how well it performs, and where it fails. Objectives: preprocess a real dataset without data leakage; train and compare models; evaluate with accuracy, precision, recall, F1 and confusion matrices; classify attack categories; visualise results; provide validated predictions and an optional dashboard.

## Dataset
* **Used: NSL-KDD** (`KDDTrain+.txt` — 125,973 records; `KDDTest+.txt` — 22,544 records; 41 features each). Source: <https://www.unb.ca/cic/datasets/nsl.html>. Files were obtained from the public mirror <https://github.com/defcom17/NSL_KDD>.
* **Why not CIC-IDS2017?** CIC-IDS2017 (<https://www.unb.ca/cic/datasets/ids-2017.html>) was the recommended dataset, but its server was not reachable from the environment where this project was built, so the documented fallback (a smaller legitimate public IDS dataset) was used. **No CIC-IDS2017 results are claimed.**
* The data files are included in `data/`. If they are missing, the notebook tries to download them automatically; if that fails, download the two files manually and put them in the `data` folder.
* Only a stratified split of the *training* file is used for fitting; the official test file is used only for final evaluation. Label grouping: `normal` → Normal; the other 39 attack names → DoS, Probe, R2L or U2R (standard NSL-KDD grouping, see notebook section 4).

## Technologies
Python · Pandas · NumPy · Scikit-learn · Matplotlib · Seaborn · Jupyter Notebook · Streamlit (optional dashboard) · joblib

## Project structure
```
SentinelML/
├── YourName_SentinelML.ipynb      # complete notebook (already executed; outputs included)
├── requirements.txt               # Python dependencies
├── YourName_ProjectReport.docx    # project report
├── README.md                      # this file
├── app.py                         # optional Streamlit dashboard
├── data/                          # KDDTrain+.txt, KDDTest+.txt
├── models/                        # rf_binary.joblib, rf_category.joblib, model_metadata.json
└── outputs/                       # charts (PNG), metrics.json, sample_traffic_for_dashboard.csv
```
The four mandatory BharatCares files are the notebook, `requirements.txt`, the report and this README. **Before submitting, replace `YourName` in the two file names with your name** (follow the bootcamp naming convention exactly) and fill in your name/date on the report's title page.

## Windows installation
1. Install **Python 3.11 or newer** from <https://www.python.org/downloads/> (tick **"Add python.exe to PATH"**). Built and tested with Python 3.12.3.
2. Unzip the project, then open **PowerShell** (or the VS Code terminal) inside the `SentinelML` folder.
3. Create and activate a virtual environment:
```powershell
python -m venv venv
venv\Scripts\activate
```
   If PowerShell blocks activation, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` and try again.
4. Install the dependencies:
```powershell
pip install -r requirements.txt
```

## Running the notebook
**VS Code:** open the `SentinelML` folder, open the `.ipynb` file, choose the `venv` interpreter as the kernel, and click **Run All**.
**Jupyter:** run `jupyter notebook` in the `SentinelML` folder, open the notebook, then *Kernel → Restart & Run All*.
The notebook must be run from inside the `SentinelML` folder. It takes about a minute, and recreates `models/` and `outputs/`.

## Running the dashboard (optional)
Run the notebook first (so models and charts exist for your environment), then:
```powershell
streamlit run app.py
```
Pages: Overview · Dataset & Threat Analytics · Model Evaluation · Predict Traffic (CSV upload or built-in sample, with downloadable predictions). A ready-made sample CSV is in `outputs/sample_traffic_for_dashboard.csv`; uploaded files need the 41 feature columns with the names used in that file (an optional `label` column is ignored).

## Actual results (from the executed notebook)
Precision / recall / F1 are for the Attack class. *Validation* = held-out 20% of the training file (same attack types as training). *Official test* = `KDDTest+`, which contains 17 attack types never seen in training.

| Model | Evaluated on | Accuracy | Precision | Recall | F1-score |
|---|---|---|---|---|---|
| Random Forest | Validation | 99.88% | 99.95% | 99.80% | 99.87% |
| Random Forest | Official test | 77.32% | 96.85% | 62.19% | 75.74% |
| Logistic Regression | Validation | 97.35% | 97.81% | 96.46% | 97.13% |
| Logistic Regression | Official test | 75.45% | 91.78% | 62.46% | 74.34% |

Random Forest confusion matrix, official test set: 9,451 normal correctly passed, 260 false alarms, 4,852 attacks missed, 7,981 attacks detected.

**Attack-category Random Forest:** accuracy 99.88% (validation, macro F1 95.00%) and 73.94% (official test, macro F1 47.79%). Per-class results on the official test set:

| Category | Precision | Recall | F1-score | Test records |
|---|---|---|---|---|
| Normal | 63.62% | 97.41% | 76.97% | 9,711 |
| DoS | 96.12% | 77.33% | 85.71% | 7,460 |
| Probe | 86.38% | 59.19% | 70.25% | 2,421 |
| R2L | 70.00% | 0.24% | 0.48% | 2,885 |
| U2R | 40.00% | 2.99% | 5.56% | 67 |

**How to read these results:** the model is nearly perfect on data resembling its training set but much weaker on the official test set, where it misses about 38% of attacks and almost never recognises R2L and U2R attacks (rare in training, and many test attacks are of unseen types). This is reported honestly rather than hidden; the numbers are exactly those printed by the notebook with `random_state=42`.

## Limitations
* NSL-KDD derives from 1999 data; modern traffic and attacks differ, so results do not transfer directly to today's networks.
* Strong drop from validation to official-test performance (unseen attack types); R2L/U2R statistics rest on few records.
* Works on pre-computed connection records, not raw packets or live traffic.
* One fixed split and seed, no cross-validation or hyper-parameter tuning.
* Built and executed on Linux (Ubuntu, Python 3.12.3). **Not executed on Windows**, and the dashboard's CSV-upload widget was not tested in a desktop browser.
* Educational use only; no guarantee of attack detection.

## Troubleshooting
| Problem | Fix |
|---|---|
| `'python' is not recognized` | Reinstall Python and tick *Add python.exe to PATH*, or use `py -m venv venv`. |
| `No matching distribution` when installing | Use Python 3.11–3.13, or relax the pinned versions in `requirements.txt`. |
| Notebook says data file not found / download failed | Put `KDDTrain+.txt` and `KDDTest+.txt` into `data/` (links above). Check you opened the notebook from inside `SentinelML`. |
| Kernel not found in VS Code | Select the `venv` interpreter (*Ctrl+Shift+P → Python: Select Interpreter*). |
| Dashboard says files are missing or models cannot be loaded | Run the whole notebook once in your environment, then restart the dashboard. |
| `streamlit` not recognised | Activate the virtual environment, or run `python -m streamlit run app.py`. |

## References
Tavallaee, Bagheri, Lu & Ghorbani (2009), *A detailed analysis of the KDD CUP 99 data set*, IEEE CISDA · Breiman (2001), *Random Forests* · Pedregosa et al. (2011), *Scikit-learn: Machine Learning in Python*.
