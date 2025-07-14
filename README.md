# S&P 500 Total-Return Attribution (Feb 2025)

> **Analyst :** Junho Kim  
> **Assignment date :** Feb 2025  
> **Data vendor :** WRDS (CRSP + Compustat + S&P DJI)  
> **Language/Tools :** Python · pandas · NumPy · Matplotlib · JupyterLab  

---

## 1 . Project Goal
Quantify the **2023** performance of the S&P 500 at three levels  

1. **Headline index** (total return)  
2. **Market-capitalisation tiers** (cumulative-percentile buckets)  
3. **GICS sectors**

All calculations rely exclusively on *point-in-time* data delivered through WRDS, eliminating survivorship and look-ahead bias.

---

## 2 . Datasets (WRDS)

| Table | Purpose |
|-------|---------|
| **CRSP `msp500list`** | Daily S&P 500 membership with entry/exit dates (historical constituents). |
| **CRSP `msi`** | Daily S&P 500 **total-return** and **price-return** series. |
| **CRSP `dsf`** | Daily security-level prices, shares outstanding, and volume (delisting returns included). |
| **Compustat `g_secd`** | Historical GICS codes (gvkey–iid match). |
| **CRSP–Compustat Link (`ccmxpf_linktable`)** | Maps PERMNO ↔︎ GVKEY for GICS join. |

> **Why not Yahoo / Bloomberg?**  
> WRDS offers *survivor-inclusive* coverage and explicit index membership flags, so back-tests reflect only information available at any given timestamp.

---

## 3 . Methodology

### 3.1 Headline Index
* 2023 total-return = **26.29 %**  
  ```python
  tr = msi.loc['2023-12-29', 'spxtr']/msi.loc['2023-01-03', 'spxtr'] - 1
  ```

### 3.2 Market-Cap Buckets  
1. **Clean the equity tape**  
   ```python
   dsf = dsf[(dsf.prc > 0) & (dsf.vol > 0)]      # drop stub quotes
   dsf['mcap'] = dsf.prc.abs() * dsf.shrout / 1e6
   ```
2. **Aggregate share classes** – sum `mcap` by company (`permco`).  
3. **Join to daily index membership** (`msp500list`) to isolate in-index names.  
4. **Rank by cumulative mcap** each rebalance date (S&P effective days).  
   *Top 70 %*, *70–90 %*, *90–97.5 %*, *97.5–100 %* chosen for **Large / Mid / Small / Micro**.  
5. **Value-weight returns** inside each bucket and **chain-link** across rebalances.

### 3.3 GICS Sectors  
1. Attach 8-digit GICS codes via Compustat; fill < 1 % missing gvkeys using the CRSP header’s SIC→GICS concordance—no manual typing.  
2. At each rebalance freeze the constituent list, compute sector-level cap-weighted returns, then geometric-link.  
   ```python
   sector_daily = panel.groupby(['date','gsector']).apply(vw_ret)
   sector_tr = sector_daily.groupby('gsector').apply(chain_link)
   ```

---

## 4 . Results Snapshot (2023)

| Slice | TR % | Notes |
|-------|------|-------|
| **S&P 500** | **26.29** | Gross dividend reinvestment. |
| Large-Cap (top 70 %) | 29.8 | Dominated by mega-cap tech rally. |
| Mid-Cap | 14.2 | Lagged as regional-bank stress hit mid-financials. |
| Small-Cap | 11.6 | Energy and staples drag. |
| Micro-Cap | 7.5 | Illiquidity amplified downside in Q1. |
| Info Tech | 56.4 | AI-related multiple expansion. |
| Comm Svcs | 44.1 | FAANG streaming & ads rebound. |
| Utilities | -7.0 | Rate-sensitive, dividend proxy. |

*(Full tables & charts live in the notebook.)*

---

## 5 . Repro Steps

```bash
git clone https://github.com/<user>/sp500-attribution.git
cd sp500-attribution
conda env create -f environment.yml        # or pip install -r req.txt
export WRDS_USER=<your_id>
jupyter lab notebooks/01_analysis.ipynb     # run All ▶︎
```

---

## 6 . Key Takeaways
* **Mega-cap skew** – Top-10 names contributed ~70 % of the index’s 2023 gain.  
* **Sector dispersion** – 63 pp gap between best (Info Tech) and worst (Utilities).  
* **Methodology matters** – Fixed-count or naïve equal-weighting would overstate small-cap influence; point-in-time buckets avoid this distortion.

---

## 7 . Next Enhancements
* Add **transaction-cost “implementation shortfall”** to convert theoretical returns into investable net performance.  
* Compare **net total return** to gauge cross-border tax drag.  
* Extend to **rolling five-year windows** for persistence tests.

---

## 8 . Contact
Junho Kim · junho.kim@example.com · [LinkedIn](https://linkedin.com/in/…)
