# API Reference

Base URL: `http://localhost:8000`

## Endpoints

### Health

`GET /health` — Returns service status and dependency health.

### Authentication

`POST /auth/register` — Create a new account.
`POST /auth/login` — Obtain a JWT access token.

### Profiles

`POST /profiles` — Create a profile with resume and GitHub username.

**Request body** (`multipart/form-data`):

| Field | Type | Required | Constraints |
| --- | --- | --- | --- |
| `github_username` | string | No | Max length 255 |
| `portfolio_url` | string | No | Max length 500 |
| `resume_file` | file | No | Accepted types: `application/pdf`, `text/markdown`, `text/plain`. Returns `422` with `"Resume must be a PDF or Markdown file"` for other types. |

`GET /profiles/{profile_id}` — Retrieve a profile.
`DELETE /profiles/{profile_id}` — Delete a profile and associated data.

### Reviews

`POST /reviews` — Request a new portfolio review for a profile.
`GET /reviews/{review_id}` — Retrieve a completed review.
`GET /reviews` — List reviews for the authenticated user (paginated).

## Interactive Docs

When the API is running, visit:
- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc
