# Fairness Audit of Open LLMs for Library Reference Services ⚖

This repository supports our chapter **Equitable Intelligence in Practice: A Fairness Audit of Open Large Language Models for Library Reference Services** (Haining Wang, Jason Clark, Angelica Peña), in *Artificial Intelligence and Social Justice Intersections in Library and Information Studies: Challenges and Opportunities*, edited by **Hengyi Fu, Souvick (Vic) Ghosh, Darra Hofman, and Bharat Mehra** (*Advances in Librarianship*, Emerald).

We evaluate whether **open-weight LLMs**, when prompted as helpful librarians, **systematically vary** their email-style reference responses by **sex** or **race/ethnicity** cues conveyed through names. This repo holds the study artifacts (data, scripts, figures). For the general Fairness Evaluation Protocol (FEP), see https://github.com/AI4Library/FEP.

---

## Main finding

![Summary of classification margins across models, settings, and demographic dimensions](phase1_overview_forest_sig.png)

Across **academic** and **public** library settings, no classifier detected race/ethnicity-linked differentiation in the responses of any model (0 of 18 tests significant after Bonferroni correction). For sex, detectable differentiation appeared only for **Llama-3.1 in the academic setting** (logistic regression and XGBoost), and it was carried by a single salutation word, **"dear"**. No other word met both salience thresholds.

The protocol detects word-level (lexical) differentiation. It does not measure factual accuracy, depth of engagement, or other aspects of service quality.

---

## Study design

- **Settings**
  - **Academic libraries:** three reference templates (special collections, sports team name origin, historical population), each filled with a randomly chosen ARL member institution (`run.py`).
  - **Public libraries:** three practical-assistance templates (print/sign/scan/email a form, upload a resume, recover an email account), with institutions sampled from 50 California county library systems in the Califa member directory (`public_run.py`).

- **Identity cues**
  - Patrons are represented as **email senders** signing with a synthetic English name.
  - A quota design balances **12 groups**, sex (male/female) × race/ethnicity (six U.S. Census categories): 41 or 42 interactions per group in every run of 500.
  - First names come from the **SSA baby names** data (aggregated across years; names with fewer than five occurrences excluded).
  - Surnames come from the **2010 U.S. Census surname** table, drawn in proportion to their frequency. A race/ethnicity label is sampled from each surname's distribution, and draws are repeated until the label matches the target group.
  - Each email also ends with a randomly assigned patron-type line (`[User type: …]`), recorded as `patron_type` in the outputs.

- **Models (open)**
  - **Llama-3.1 8B Instruct**
  - **Gemma-2 9B Instruct** (system prompt prepended to the user message)
  - **Ministral 8B Instruct**

- **Generation:** temperature 0.7, up to 4,096 new tokens.

- **Scale:** 5 seeds × 500 interactions per seed per (model × setting), giving 2,500 responses per model per setting. No generation failed.

---

## Evaluation (`probe.py`)

We treat fairness as an empirical question. If responses systematically differ by a protected attribute, a classifier should predict the attribute from the response text above chance.

- **Representation:** TF-IDF over the 50 most frequent content words (English stop words and the honorifics Mr/Ms/Mrs/Miss removed). The same vocabulary size is used for every model and setting.
- **Diagnostic classifiers:** logistic regression, MLP, and XGBoost, with fixed hyperparameters.
- **Validation:** leave-one-seed-out cross-validation (five folds, one per seed).
- **Inference:** one-sample tests against chance (50% for sex, 16.7% for race/ethnicity), Bonferroni-corrected across the 9 tests per setting and attribute (α = 0.0056).
- **Interpretation:** when a signal is detected, an unpenalized statistical logistic regression on all 2,500 responses (male coded 1) gives word-level coefficients and p-values. A word is salient when p < 0.001 (Bonferroni across the 50 terms) and |β| > log 2. Results are shown as volcano plots.

Results are written to `probe.json`.

---

## Repository layout

- `run.py`, `public_run.py`: generate responses for the academic and public settings.
- `academic_runs/`, `public_runs/`: SLURM job scripts.
- `academic_outputs/`, `public_outputs/`: one JSON file per model and seed.
- `probe.py`, `probe.json`: diagnostic classification and statistical logistic regression.
- `data/NationalNames.csv`: SSA first-name data.

---

## License

MIT
