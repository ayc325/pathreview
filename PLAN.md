## Solution plan

**Issue:** API reference doc is missing the POST /profiles request body schema ([#89](https://github.com/ascherj/pathreview/issues/89))

### Understand
What is the root cause of this issue? What behavior is expected vs. actual?

The root cause is that `docs/API.md` lists `POST /profiles` with only a one-line description and no request body schema, even though the endpoint accepts a multipart form with three fields (`github_username`, `portfolio_url`, `resume_file`) that carry real constraints — max lengths enforced in `api/schemas/profile.py`'s `ProfileCreate` model, and an allowed-MIME-type check for `resume_file` enforced manually in `api/routes/profiles.py`. Expected behavior: a developer reading `docs/API.md` should be able to construct a valid request (correct field names, types, and constraints) without needing to read the source code. Actual behavior: the doc gives no field-level detail at all, so a developer would only discover the constraints by trial and error — e.g., hitting an undocumented `422` when uploading a resume in an unsupported file format.

**Root cause:** Missing request body documentation in `docs/API.md` — the endpoint's actual `Form`/`File` parameters and their constraints are never surfaced to the reader.

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

**Input:** The actual field definitions and validation rules read from `api/routes/profiles.py` (the `resume_file` MIME-type check) and `api/schemas/profile.py` (the `ProfileCreate` model's field types and `max_length` constraints) — these are the source of truth the doc must match.

**Output:** An updated `docs/API.md` where the `POST /profiles` entry includes a request body schema (field names, types, required/optional, constraints), a note on the `422` file-type error, and one example request/response. No code changes — the fix only produces new/changed content in `docs/API.md`; behavior of the running API is unaffected.

**Verification (via Swagger UI at `localhost:8000/docs`) before writing each doc claim:**

- Submit `POST /profiles` with `github_username` and `portfolio_url` omitted entirely — confirm it succeeds, to verify these fields are truly optional despite the `str` (not `Optional[str]`) type hint.
- Submit with `github_username=""` (empty string) — confirm whether this behaves the same as omitting it, or differently.
- Upload a `.docx` or `.png` as `resume_file` — confirm the response is `422` with `"Resume must be a PDF or Markdown file"`.
- Upload a corrupted/invalid PDF — confirm the response is `422` with `"Failed to parse PDF resume"`, distinct from the file-type error.
- Submit `github_username` over 255 characters — confirm the request is rejected (via `ProfileCreate` validation) even though the `Form(...)` param itself declares no limit.

### Risks & unknowns
What could go wrong? What are you still unsure about?

- `github_username` and `portfolio_url` are typed `str` (not `Optional[str]`) in the `Form(...)` params in `api/routes/profiles.py:25-26`, even though they behave as optional (`default=None`) and are only truly constrained by `ProfileCreate` in `api/schemas/profile.py`. Risk of documenting "required" incorrectly — I should verify actual behavior by sending a request that omits these fields before writing the doc, not just trust the type hint.
- The MIME-type check for `resume_file` (`api/routes/profiles.py:42-53`) relies on `resume_file.content_type` first, falling back to `mimetypes.guess_type(resume_file.filename)`. A browser or client might send an empty/unexpected `content_type` for `.md` files, so the "accepted MIME types" I document could look right on paper but not match what actually gets sent in practice — I'm unsure whether to document the MIME types or the file extensions, since they may not always agree.
- `docs/API.md` has no existing convention for a request body schema (every endpoint is currently a single one-line bullet), so there's a risk a reviewer expects a different format (e.g., JSON schema block vs. Markdown table) than what I choose. I should check `docs/CONTRIBUTING.md` and/or ask a mentor before finalizing formatting.
- `PUT /profiles/{profile_id}` exists in `api/routes/profiles.py:147` but isn't documented in `docs/API.md` at all. This is out of scope for issue #89, but there's a risk of scope creep if I try to "fix everything" while I'm in the file — I'll leave it undocumented and out of this PR.

### Edge cases
What inputs or states should your fix handle gracefully?

- **No `resume_file` provided:** all three fields are optional, so the doc must make clear a profile can be created with just `github_username`/`portfolio_url`, or with neither, and no file at all (`api/routes/profiles.py:27`, `default=None`).
- **Unsupported file type uploaded:** e.g. a `.docx` or `.png` resume — the doc must state this returns `422` with `"Resume must be a PDF or Markdown file"`, not a silent failure or generic error.
- **Corrupted/unparseable PDF:** a valid-MIME-type PDF that fails `PyPDF2` parsing returns a different `422` (`"Failed to parse PDF resume"`, `api/routes/profiles.py:68-71`) — worth documenting as a distinct case from the file-type rejection so developers don't confuse the two error messages.
- **Field values exceeding max length:** `github_username` over 255 chars or `portfolio_url` over 500 chars — the doc should note these are rejected by validation (via `ProfileCreate`) even though the `Form(...)` params themselves don't declare the limit.
- **Empty string vs. omitted field:** since the params default to `None` rather than being marked `Optional[str]`, it's worth documenting whether sending `github_username=""` behaves the same as omitting it entirely, so developers don't assume the two are equivalent.
