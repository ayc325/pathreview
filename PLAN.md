## Solution plan

**Issue:** [issue title and link]

### Understand
What is the root cause of this issue? What behavior is expected vs. actual?

The root cause is that `docs/API.md` lists `POST /profiles` with only a one-line description and no request body schema, even though the endpoint accepts a multipart form with three fields (`github_username`, `portfolio_url`, `resume_file`) that carry real constraints — max lengths enforced in `api/schemas/profile.py`'s `ProfileCreate` model, and an allowed-MIME-type check for `resume_file` enforced manually in `api/routes/profiles.py`. Expected behavior: a developer reading `docs/API.md` should be able to construct a valid request (correct field names, types, and constraints) without needing to read the source code. Actual behavior: the doc gives no field-level detail at all, so a developer would only discover the constraints by trial and error — e.g., hitting an undocumented `422` when uploading a resume in an unsupported file format.

### Map
Which files, functions, or modules are involved?
List the specific files you expect to touch.

Files/functions involved:

- `docs/API.md` — the file to edit; currently has a one-line description for `POST /profiles` (in the `### Profiles` section) with no request body schema documented.
- `api/routes/profiles.py`, `create_profile_endpoint` (lines 23-30) — source of truth for the actual parameters (`github_username`, `portfolio_url`, `resume_file`) and the manual MIME-type validation logic (lines 40-53) that needs to be reflected in the docs.
- `api/schemas/profile.py`, `ProfileCreate` — source of truth for the `max_length` constraints (255 for `github_username`, 500 for `portfolio_url`) and optionality.

Only `docs/API.md` will actually be modified — the other two files are read-only references I'm documenting from, not touching.

### Plan
What are the steps to fix this issue?
Break it into 3–5 concrete sub-tasks.

1. Read `create_profile_endpoint` in `api/routes/profiles.py` (lines 23-30, 40-53) and `ProfileCreate` in `api/schemas/profile.py` side by side to confirm the final values to document for each field.
2. In `docs/API.md`, under the existing `POST /profiles` line, add a request body schema block listing exactly these three fields:
   - `github_username` — string, optional, max length 255
   - `portfolio_url` — string, optional, max length 500
   - `resume_file` — file upload, optional, accepted MIME types `application/pdf`, `text/markdown`, `text/plain`
3. Directly below the field list, add a note that an unsupported `resume_file` type returns `422 Unprocessable Entity` with detail `"Resume must be a PDF or Markdown file"` — this comes from the manual check in the route handler, not the Pydantic schema, so it's easy to miss.
4. Add a short example `multipart/form-data` request (showing all three fields) and a successful response so the doc is usable without cross-referencing the source code.
5. Re-check the finished doc against `localhost:8000/docs` (Swagger UI) to confirm the field names, types, and constraints match exactly, then commit.

### Inputs & outputs
What does your fix take as input? What should it produce or change?

### Risks & unknowns
What could go wrong? What are you still unsure about?

### Edge cases
What inputs or states should your fix handle gracefully?