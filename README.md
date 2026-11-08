# Contract Court

When a team changes a data column, the people who depend on it usually find out too late. Contract Court
measures who a change would really break, lets agents for each affected team negotiate a safe plan, and
turns the deal into a contract that blocks rogue changes.

Built for the TAMU Datathon "Best Use of Databricks" challenge.

## What it does
1. **Measure.** In Databricks, every column of every table in `samples.wanderbricks` is dropped, renamed,
   retyped and rescaled in a sandbox copy. Every downstream report is re-run with its own SQL and compared with
   its baseline. A report is affected if it crashes or silently returns different numbers.
2. **Negotiate.** Each affected report becomes an agent with its own needs (notice, overlap, freeze windows).
   The owners concede in a fixed order, or a Databricks model (Llama 3.3 70B) negotiates for some changes.
   Fixed rules always referee the result.
3. **Enforce.** The deal becomes a versioned contract, and a migration that ignores it is blocked.

## Run the app
```bash
pip install streamlit plotly pandas numpy
cd app && python3 -m streamlit run app.py
```
The app reads `app/data/snapshot.json`, which the Databricks notebooks produced. It does not connect to
Databricks or to the AI model. The AI negotiations are transcripts computed in advance on Databricks.

## Where the Databricks work is
- `notebooks/01_build_workload.py`: copies sample tables and builds the downstream reports.
- `notebooks/02_measure_and_export.py`: runs the impact tests and exports the snapshot.
- `notebooks/03_llm_negotiations.py`: model-negotiated deals, with prompts and replies logged to MLflow.
- `engine/`: the measurement, negotiation and enforcement logic. `tests/` runs it on a local DuckDB stand-in.

## Honest limits
- The data is Databricks' simulated sample data, and the downstream reports are generated from the schema.
- Each team's needs are derived defaults (from refresh rate and chain depth) that you can edit in the app.
- Four kinds of change are measured: rename, drop, retype and rescale.
- A migration is only blocked if it calls `check_change` first.
- Not endorsed by Databricks.

See `GUIDE.md` for the full setup.
