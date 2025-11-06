# Portfolio Lab — Efficient Frontier, Tangency vs Min-Var, and Monte Carlo Risk

**What this is:** A Streamlit app to explore portfolio construction with **Mean–Variance (Markowitz)**, optional **Ledoit–Wolf covariance shrinkage**, **sector caps**, and **Monte Carlo risk** (VaR/CVaR, drawdowns). Built to be Colab-friendly.

---

## Features
- Upload **merged_prices.csv** (wide format: `Date, T1, T2, ...`), optional **merged_metadata.csv** with a `Sector` column.
- Toggle **Ledoit–Wolf shrinkage** for more stable covariance.
- Choose **long-only** or allow **limited shorting** (−20% to 100%).
- Apply **sector caps** (e.g., `{"Crypto": 0.15, "Tech": 0.60}`).  
- View **Efficient Frontier**, **Capital Market Line**, **Tangency/MinVar weights**.
- Run **Monte Carlo** on the Tangency portfolio: **VaR(95%)** / **CVaR(95%)** at 1‑day and 21‑day horizons, simulated drawdowns.
- Export artifacts: weights, returns, and a JSON risk report.

---

## Quickstart (Colab)
1. Open a Colab notebook and run:
   ```python
   !pip -q install streamlit==1.39.0 pyngrok==6.0.0 scikit-learn pandas numpy matplotlib
   NGROK_AUTH_TOKEN = ""  # optional
   ```
2. Save the app (use the code from `app.py` in this repo) and launch via ngrok:
   ```python
   from pyngrok import ngrok
   import os, threading, time

   if NGROK_AUTH_TOKEN:
       ngrok.set_auth_token(NGROK_AUTH_TOKEN)

   for t in ngrok.get_tunnels():
       ngrok.disconnect(t.public_url)

   public_url = ngrok.connect(8501, "http")
   print("Public URL:", public_url)

   def run():
       os.system("streamlit run app.py --server.port 8501 --server.address 0.0.0.0")
   threading.Thread(target=run, daemon=True).start()
   time.sleep(3)
   ```
3. In the app sidebar, upload **merged_prices.csv** (and **merged_metadata.csv** if you want sector caps).

---

## Inputs
- **merged_prices.csv** — `Date` + one column per asset with positive prices.
- **merged_metadata.csv** (optional) — columns: `Ticker`, `Sector` (case-sensitive match to price columns).

> Tip: Use the provided **MultiCSV Preprocessor** to generate these files from a Kaggle ZIP of per-company CSVs.

---

## Controls (in the sidebar)
- **Risk-free rate (annual):** default `0.02`. Use something current (e.g., 0.04–0.045) if you want realism.
- **Ledoit–Wolf shrinkage:** ON by default. Turn OFF to see how noisy covariances distort weights.
- **Allow limited shorting:** expands bounds from `[0,1]` to `[−0.2, 1.0]`.
- **Sector caps (JSON):** e.g. `{"Crypto": 0.15, "Tech": 0.60}` (requires `Sector` in metadata).

---

## Outputs
- **KPIs:** MinVar return/vol, Tangency Sharpe, asset universe size.
- **Tables:** MinVar & Tangency **weights** (sorted).
- **Charts:** Efficient Frontier + CML; (optional) Monte Carlo **paths** & **daily return histogram**.
- **Artifacts (on export):**
  - `artifacts/weights_tangency.csv`
  - `artifacts/weights_minvar.csv`
  - `artifacts/asset_returns.csv`
  - `artifacts/risk_report.json` (settings + in-sample stats)

Optionally, after you run MC from the app or your notebook, save `artifacts/risk_report_mc.json` for VaR/CVaR & drawdowns.

---

## Example Results (replace with your actual exports)
- **Universe:** 12 assets; **Shrinkage:** ON; **Shorting:** OFF.  
- **MinVar:** ~11% return / ~13% vol (annual).  
- **Tangency:** Sharpe ≈ 2.3 (expect lower out-of-sample).  
- **MC Risk (Tangency, 95%):** 1‑d VaR ≈ −3.2%, CVaR ≈ −4.1%; 21‑d VaR ≈ −9.5%, CVaR ≈ −13.1%; median drawdown ≈ −18% (worst ≈ −40%).

> **Interpretation:** Tangency is aggressive (higher tail risk). Use sector caps (e.g., Crypto ≤ 15%, Tech ≤ 60%) and shrinkage to reduce drawdowns with modest impact on Sharpe.

---

## Reproducibility (Local)
```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
streamlit run app.py
```
Then open the local URL printed by Streamlit.

---

## Files in this repo (suggested)
```
portfolio-lab/
├─ app.py
├─ artifacts/                 # created by the app
│  ├─ weights_tangency.csv
│  ├─ weights_minvar.csv
│  ├─ asset_returns.csv
│  ├─ risk_report.json
│  └─ risk_report_mc.json     # optional (if you save MC metrics)
├─ README.md
└─ requirements.txt
```

---

## License
For educational/portfolio purposes. Kaggle data is subject to its original terms.
