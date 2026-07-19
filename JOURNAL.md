## Week 7 — Issue selection

**Issue link:** (https://github.com/ascherj/pathreview/issues/89)

**Issue title:** API reference doc is missing the POST /profiles request body schema

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
[In 3–5 sentences, in your own words: what the issue is (not a copy-paste of
the title), what is currently broken or missing, and what a successful fix
would accomplish. Naming the part of the codebase it affects is helpful context.]

The issue is that there is documentation missing for the POST /profiles endpoint, specifically the request body schema. The bug was recreated via `localhost:8000/docs` as the schema was nowhere to be found. A successful fix would include a request body schema. The affected part of the codebase is in `docs/API.md`.

**Branch name:** docs/CONTRIBUTING.md

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger