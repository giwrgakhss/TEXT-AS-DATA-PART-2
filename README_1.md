# NLP-Economics Competition
## Phase B: Hawk-Dove Index
### Semi-Innovation Question — *Text as Data in Economics: The Power of AI*

---

## 1. Project Overview

This notebook constructs a **Hawk-Dove Monetary Policy Index** from the BIS Central Bankers' Speeches dataset and investigates its statistical relationship with the **2-Year US Treasury Yield** — a market-based proxy for Fed policy expectations.

The core idea is to convert unstructured speech text into a directional numeric signal that captures whether a central banker is leaning toward monetary **tightening (hawkish)** or **easing (dovish)**, and then test whether this signal contains predictive information about financial market variables.

---

## 2. Research Question

> *"Does the tone of Federal Reserve speeches — measured via a Hawkish/Dovish dictionary — predict movements in the 2-Year US Treasury Yield?"*

This question is grounded in the central banking communication literature. **Forward guidance** — the practice of signaling future policy intentions through speech — is known to move financial markets. By quantifying speech tone, we attempt to capture this signal systematically.

---

## 3. Methodology

### 3.1 Text Preprocessing
- Convert all text to lowercase
- Remove numbers and punctuation
- Remove English stopwords (NLTK)
- Tokenize into word lists for negation-aware matching

### 3.2 Dictionary Construction

Two domain-specific dictionaries are built, informed by the **Loughran-McDonald Financial Sentiment Dictionary** — the academic gold standard for NLP in Finance/Economics *(Journal of Finance, 2011)*:

| Direction | Signal | Example Words |
|-----------|--------|---------------|
| 🦅 **Hawkish** | Monetary tightening | `inflation, tighten, hike, restrictive, normalize, vigilant, tapering` |
| 🕊️ **Dovish** | Monetary easing | `accommodative, easing, stimulus, cut, unemployment, slack, recession` |

### 3.3 Negation Handling

A key robustness feature: before counting any dictionary match, the **5 preceding tokens** are scanned for negation words (`not, no, never, cannot, n't`). If a negation is found, the match sign is **flipped** — e.g. *"not tightening"* is treated as dovish rather than hawkish.

```
Hawk_Score = (hawkish_hits_adjusted - dovish_hits_adjusted) / total_words × 1000
```

- **Positive score** → Hawkish speech
- **Negative score** → Dovish speech
- **Near zero** → Neutral speech

### 3.4 Time Series Construction

1. Filter speeches to **Federal Reserve only**
2. Exclude months with **< 3 speeches** (reliability threshold)
3. Compute **monthly mean** of Hawk-Dove Score
4. Run **ADF Test** for stationarity — apply first-differencing if non-stationary

### 3.5 Financial Variable

The **2-Year US Treasury Yield** is downloaded via `yfinance`. It is resampled to monthly frequency and also tested for stationarity.

**Why 2Y Yield instead of VIX:**
The 2-year yield is the market's forward-looking expectation of Fed policy over the next 2 years, making it a direct and economically motivated counterpart to Fed speech tone. VIX reflects general market fear and is influenced by many unrelated factors.

### 3.6 Statistical Analysis — Three Levels

| Level | Method | What It Tests |
|-------|--------|---------------|
| 1 | Pearson Correlation | Contemporaneous linear association between the two series |
| 2 | OLS Regression with Lags | Whether last month's tone predicts this month's yield |
| 3 | Granger Causality Test | Whether speech tone improves forecast of yield beyond its own history |

---

## 4. Robustness Checks

| Check | Method | Why It Matters |
|-------|--------|----------------|
| Length normalization | Divide by total word count × 1000 | Prevents longer speeches from dominating |
| Negation handling | Flip sign within 5-token window | Avoids misclassifying "not hawkish" as hawkish |
| Stationarity | ADF Test on both series | Prevents spurious correlations from trends |
| Min. speech threshold | Exclude months with < 3 speeches | Ensures monthly averages are reliable |
| Dictionary quality | Loughran-McDonald based | Academically justified, not arbitrary |

---

## 5. Historical Sanity Check

To validate that the index captures real-world monetary policy stances, we compare it against known historical episodes:

| Period | Known Policy Stance | Expected Index Signal |
|--------|--------------------|-----------------------|
| **2022** | Aggressive Fed rate hikes (inflation surge) | High positive (hawkish) ✅ |
| **2020 Q1-Q2** | COVID — rates cut to 0, QE launched | Negative (dovish) ✅ |
| **2018** | Gradual tightening cycle | Moderate positive ✅ |
| **2015-2016** | Very slow normalization post-GFC | Slightly positive ✅ |

If the index correctly identifies all four periods, this provides strong **visual evidence of validity** — beyond just p-values.

---

## 6. Full Pipeline

```
CBS Dataset (Central Bank Speeches)
        ↓
Text Cleaning (lowercase, stopwords, tokenization)
        ↓
Hawk / Dove Dictionary + Negation Handling
        ↓
Hawk-Dove Score per Speech
        ↓
Filter: Federal Reserve only
        ↓
Monthly Aggregation (min. 3 speeches threshold)
        ↓
ADF Stationarity Test → first-difference if needed
        ↓
2-Year Treasury Yield (Yahoo Finance)
        ↓
Pearson Correlation
OLS Regression with Lags (t-1, t-2, t-3)
Granger Causality Test
        ↓
Visualization + Historical Validation
```

---

## 7. Repository Structure

```
📦 TEXT-AS-DATA-PART-2/
├── 📓 phase2.ipynb          # Main analysis notebook
├── 📄 README.md             # This file
└── 📁 data/
    └── CBS_dataset_v1.0.csv # BIS Central Bankers' Speeches dataset (place here)
```

---

## 8. Dependencies

```bash
pip install pandas numpy nltk yfinance statsmodels matplotlib seaborn scipy
```

| Library | Purpose |
|---------|---------|
| `pandas` | Data manipulation and time series |
| `numpy` | Numerical computations |
| `nltk` | Stopwords and tokenization |
| `yfinance` | Download 2Y Treasury Yield data |
| `statsmodels` | OLS regression and Granger causality |
| `matplotlib` / `seaborn` | Visualization |
| `scipy` | Pearson correlation |

---

## 9. Dataset

The CBS dataset (`CBS_dataset_v1.0.csv`) is the **BIS Central Bankers' Speeches** dataset.
Download it from: [https://www.bis.org/cbspeeches/download.htm](https://www.bis.org/cbspeeches/download.htm)

Place the downloaded CSV in the `/data/` directory before running the notebook.

**Required columns:** `Date`, `text`, `Country`, `CentralBank`

---

*NLP-Economics Competition · Phase B · Qualco Intelligent Finance · 2026*
