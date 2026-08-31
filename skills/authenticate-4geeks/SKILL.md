---
name: "authenticate-4geeks"
description: "Verify 4Geeks API token validity and active session status"
---

# Authenticate 4Geeks

Validates the `TOKEN_4GEEKS` from `.env` against the 4Geeks BreatheCode API and reports session status.

## When to use

- Before making any 4Geeks API call
- When diagnosing connectivity issues with 4Geeks
- On startup or after token rotation

## API Details

- **Base URL:** `https://breathecode.herokuapp.com`
- **Auth header:** `Authorization: Token <TOKEN_4GEEKS>`
- **Token source:** `.env` file in workspace root, variable `TOKEN_4GEEKS`

## Workflow

1. Read the token from `.env` (`TOKEN_4GEEKS`).
2. Call `GET /v1/admissions/user/me` with the auth header.
3. If HTTP 200 → token is valid. Report:
   - User: `first_name`, `last_name`, `email`
   - Academy: from `profile_academy` or first cohort's `academy`
   - Active cohorts: filter where `educational_status == "ACTIVE"` and `completion.is_complete == false`
4. If HTTP 401 → token is invalid/expired. Report the error.
5. For deeper session check, call `GET /v1/admissions/academy/cohort/me` with `educational_status=ACTIVE` to list active enrollments.

## Constraints

- Never log or echo the raw token value.
- Never hardcode the token; always read from `.env`.
- Don't make unnecessary calls — one `user/me` call is enough for validation.
