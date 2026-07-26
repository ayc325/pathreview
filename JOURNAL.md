## Week 7 — Issue selection

**Issue link:** (https://github.com/ascherj/pathreview/issues/89)

**Issue title:** API reference doc is missing the POST /profiles request body schema

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
[In 3–5 sentences, in your own words: what the issue is (not a copy-paste of
the title), what is currently broken or missing, and what a successful fix
would accomplish. Naming the part of the codebase it affects is helpful context.]

The issue is that there is documentation missing for the POST /profiles endpoint, specifically the request body schema. The bug was recreated via `localhost:8000/docs` as the schema was nowhere to be found. A successful fix would document the multipart form fields the endpoint actually accepts — `github_username`, `portfolio_url`, and `resume_file` — along with their types and constraints, since the endpoint uses `Form` and `File` parameters rather than a JSON body. The affected part of the codebase is in `docs/API.md`.

**Branch name:** docs/89-add-missing-request-body-schema

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger

**Issue fit and selection reasoning:**
[X] "Is this right for me?" checklist reviewed

I know that Tier 1 is right for me because it's my first open source contribution. I know what fixes to make, and I know what done looks like based on the other examples of a complete API schema. There is a good number of people that also chose this issue, but I don't have a problem with that. Also, based on the hours I am able to commit for this project, 3–6 hours is sufficient especially since I have a lot of other stuff going on during these last few weeks. The only dependency is that I should check if the code for the POST /profiles endpoint is complete before writing the reference doc.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [link to commit documenting the reproduced issue](https://github.com/ayc325/pathreview/commit/8993fa22e63ba9aa98ea4a4d09facadd49aea3ba)

**Reproduction summary:**
I ran the app locally and opened the interactive API docs at `localhost:8000/docs`, then navigated to the `POST /profiles` endpoint. I observed that the endpoint's request body section did not list the expected schema (the `github_username`, `portfolio_url`, and `resume_file` multipart form fields), confirming that `docs/API.md` is missing this documentation and needs to be updated to match the actual `Form`/`File` parameters accepted by the endpoint.

**PLAN.md link:** [PLAN.md](https://github.com/ayc325/pathreview/blob/docs/89-add-missing-request-body-schema/PLAN.md)

**Walkthrough video (recommended):** Skipped — not part of the grade, and the reproduction commit + PLAN.md already capture what the video would have covered:

- Reproduced the missing schema by hitting `localhost:8000/docs` and confirming `POST /profiles` shows no request body schema.
- Walked through the planned fix: document the three actual fields (`github_username`, `portfolio_url`, `resume_file`) and their constraints in `docs/API.md`, based on `api/routes/profiles.py` and `api/schemas/profile.py`.
- Happy to do a live walkthrough in office hours instead if early feedback would help before I start building.

**Blockers or open questions:**
[Anything you're still uncertain about going into Week 9, or leave blank]
