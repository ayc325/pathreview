## Solution plan

**Issue:** [issue title and link]

### Understand
What is the root cause of this issue? What behavior is expected vs. actual?

The root cause is that `docs/API.md` lists `POST /profiles` with only a one-line description and no request body schema, even though the endpoint accepts a multipart form with three fields (`github_username`, `portfolio_url`, `resume_file`) that carry real constraints — max lengths enforced in `api/schemas/profile.py`'s `ProfileCreate` model, and an allowed-MIME-type check for `resume_file` enforced manually in `api/routes/profiles.py`. Expected behavior: a developer reading `docs/API.md` should be able to construct a valid request (correct field names, types, and constraints) without needing to read the source code. Actual behavior: the doc gives no field-level detail at all, so a developer would only discover the constraints by trial and error — e.g., hitting an undocumented `422` when uploading a resume in an unsupported file format.

### Map
Which files, functions, or modules are involved?
List the specific files you expect to touch.

### Plan
What are the steps to fix this issue?
Break it into 3–5 concrete sub-tasks.

### Inputs & outputs
What does your fix take as input? What should it produce or change?

### Risks & unknowns
What could go wrong? What are you still unsure about?

### Edge cases
What inputs or states should your fix handle gracefully?