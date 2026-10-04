# E-Commerce-Customer-Intelligence-Campaign-Decision-Engine

Turns raw E
-commerce data into **decisions**: who the customers are, which abandoned carts are worth chasing, what message each customer should get, and how much discount is safe.

> **Live demo:** _add your Streamlit Cloud link here_  ·  **Dashboard screenshot:** _add Power BI / Tableau image here_

## Data
| File | Description |
|---|---|
| `sales_dataset.xlsx` (not committed, 14 MB) | 128,949 Amazon-India order lines, Mar–Jun 2022 |
| `data/raw/Ecommerce.csv` | 25,000 customer sessions · 8,442 customers |

The two datasets are **not linked** (the sales file has no customer ID).

## What was built
1. **Sales EDA + SQL** — cleaning, gross vs net revenue, SQL (MySQL; re-run in-notebook via sqlite with `assert`s against pandas).
2. **Customer 360 → RFM → K-Means** — 4 named buyer segments (silhouette 0.39).
3. **Purchase-prediction audit** — leakage detection, honest cart-stage model, shuffled-label control, time split.
4. **Recommender** — item-item collaborative filtering evaluated against random / popularity baselines.
5. **Review sentiment** — rating-based sentiment with chi-square tests.
6. **Decision engine** — rule-based policy (action + discount), LLM *copy* writer, Python guardrails, automated tests.
7. **Streamlit dashboard + Flask API**, plus CSV exports for Power BI / Tableau.

## Key results (all reproducible in the notebook)
- Gross revenue ₹78.6M vs **net ₹69.6M**; 14.2% of order lines cancelled; merchant-fulfilled orders cancel more (17.5% vs 12.8%).
- Only ~50% of customers ever bought; 4,176 buyers split into Recent One-Time (1,587), Lapsed (1,461), Loyal (775), Champions (353).
- A "near-100% accuracy" model exists only through leaked columns (`revenue`, `rating`, `review_text`). Honest cart-stage model: **AUC 0.57** vs 0.50 for shuffled labels (time-split 0.571).
- Recommenders perform at random-chance level (~0.5% Hit-Rate@5): the data has no co-purchase signal.
- Decision engine: 7 campaign actions over 8,442 customers; 2,911 Cart Recovery targets holding ₹9.0M of abandoned carts.

## Run it
```bash
pip install -r requirements.txt
streamlit run app.py        # dashboard   (reads data/customer_marketing_actions.csv)
python api.py               # JSON API on http://localhost:5000
```
Re-run the analysis: open `notebook/Project_Final.ipynb` in Google Colab (`pip install -r requirements-notebook.txt` locally).

**API:** `GET /health` · `GET /campaign/summary` · `GET /customer/<id>`

**Optional real LLM writer:** set `ANTHROPIC_API_KEY` (and optionally `ANTHROPIC_MODEL`) or `OPENAI_API_KEY`. Without a key the safe template is used. Never commit keys.

## Design choice: Python decides, the LLM only writes
Action and discount come from deterministic rules. The LLM can only change wording; guardrails reject banned phrases, any number/percentage, and enforce the discount cap and opt-out line (falls back to a template).

## Limitations (stated openly)
- Purchase prediction is weak on this data; `recovery_score` is a **ranking**, not a probability.
- Discount sizes and score thresholds are business assumptions → need an A/B test.
- Review text in the source is encoded; sentiment comes from ratings.
- The LLM path was unit-tested with fake LLMs; it still needs a run with a real API key.

## Structure
```
notebook/Project_Final.ipynb   app.py   api.py   data/   bi_exports/   docs/INTERVIEW_GUIDE.md
```
