# Contract Court: step-by-step

Menu names in Databricks shift between versions. If a button is not where it says, find the closest one.

## 1. See it on your laptop
Uses a bundled local stand-in snapshot (labelled as such in the app).
```bash
pip install streamlit plotly pandas numpy
cd contract-court/app && python3 -m streamlit run app.py
```
Stop with Control + C.

## 2. Run the notebooks in Databricks
1. Workspace, your Home folder, `contract-court` folder. Delete the old notebooks there (`01_setup...`, `02_discover...`, `03_negotiate...`).
2. Menu (three dots) or Create, Import. Drag in four files from `notebooks/`:
   `00_install_engine.py`, `01_build_workload.py`, `02_measure_and_export.py`, `03_llm_negotiations.py`. Replace `00_install_engine` if asked.
3. Open `01_build_workload`, **Connect** to **Serverless**, **Run all**. It copies a sample of every table in `samples.wanderbricks`
   into a new schema `court_lab` (it does not touch your earlier `contract_court` schema) and builds the workload.
4. Open `02_measure_and_export`, **Connect** to **Serverless**, **Run all**. It runs a self-check, then many small queries
   (several minutes), then prints `exported /Volumes/<catalog>/court_lab/exports/snapshot.json`.
5. Open `03_llm_negotiations`, **Connect** to **Serverless**, **Run all**. A Databricks model negotiates the 6 highest-impact
   changes (chosen automatically). If it cannot reach the model, check the `ENDPOINT` name at the top against Serving, Foundation models.
   It adds the results to `snapshot.json`; the app then offers a "Who negotiates?" switch for those changes.

## 3. Deploy the app
One time:
```bash
brew tap databricks/tap && brew install databricks
databricks auth login --host https://YOUR-WORKSPACE-ADDRESS
```
Every deploy (fetches the snapshot from step 2 first; use `DATABRICKS_CATALOG=name ./deploy.sh` if your catalog is not `workspace`):
```bash
cd contract-court && ./deploy.sh
```
Open it from Compute, Apps, `contract-court`. The app header says "Measured on Databricks" when it is using your snapshot.
Free Edition apps stop after 24 hours, so redeploy before judging.

## 4. Developing locally
```bash
pip install duckdb pytest
python3 -m pytest tests          # runs the whole pipeline on a local DuckDB stand-in
python3 tools/make_snapshot.py   # writes tools/standin_snapshot.json (never touches the real app/data/snapshot.json)
python3 build_notebooks.py       # re-embeds engine/ into notebooks/00_install_engine.py
```

## 5. Before you submit
- Databricks Apps are only visible to people signed in to your workspace, so judges cannot open the link. Plan a demo video and show the live workspace.
- Ask the organizers what the Databricks challenge requires you to submit.

## Honest limits
- Measured: which consumers crash or silently change, and that the agreed plan keeps them working. Tables, columns and consumers are introspected or generated from the schema, never typed into the app.
- Sample rows only (default 2,000 per table), and the Databricks sample data is simulated.
- The consumers are generated from the schema because no real workload exists in the sandbox. Their needs are derived from refresh rate and chain depth, and can be edited in the app.
- Four kinds of change are measured: rename, drop, retype and rescale.
- LLM negotiations are precomputed for the top changes with default settings. Everything else uses the rule-based agents. In both, the rule engine decides whether a deal is acceptable.
- A migration is only blocked if it calls `check_change` first.
