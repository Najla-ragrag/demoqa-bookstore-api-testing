# DemoQA Bookstore API — Automated Test Suite with CI/CD

A complete API testing project built with **Postman + Newman + GitHub Actions**, covering the full lifecycle of the [DemoQA Bookstore API](https://demoqa.com/swagger/): user registration, token-based authentication, and CRUD operations on a protected book collection — plus a dedicated set of negative tests — running automatically on every push.

## Overview

This project demonstrates end-to-end functional and negative testing of a REST API that uses Bearer Token authentication, taken from manual exploration through to a fully automated CI/CD pipeline. It includes an 8-step positive flow (20 assertions), a 6-request **Negative Tests** folder (13 assertions), dynamic test-data generation (no hardcoded users for the main flow), and a GitHub Actions workflow that runs the whole suite headlessly on every commit.

| Suite | Requests | Assertions |
|-------|----------|-----------|
| Positive flow | 8 | 20 / 20 passing |
| Negative tests | 6 | 13 / 13 passing |
| **Total** | **14** | **33 / 33 passing** |

## Tech Stack

- **Testing:** Postman (Collections, Environments, Scripts, Console)
- **Automation/CLI:** Newman + newman-reporter-htmlextra
- **CI/CD:** GitHub Actions (Ubuntu runner, Node.js 20)
- **API under test:** `demoqa.com` — Account & BookStore REST API
- **Auth type:** Bearer Token (JWT)

## Test Flow & Coverage

### Positive flow

| # | Request | Method | Endpoint | Expected Result |
|---|---------|--------|----------|------------------|
| 1 | Register | POST | `/Account/v1/User` | 201 Created |
| 2 | GenerateToken | POST | `/Account/v1/GenerateToken` | 200 OK — returns JWT |
| 3 | Authorized Check | POST | `/Account/v1/Authorized` | 200 OK — `true` |
| 4 | Get All Books | GET | `/BookStore/v1/Books` | 200 OK — public, no auth required |
| 5 | Add Book to Collection | POST | `/BookStore/v1/Books` | 201 Created — requires Bearer token |
| 6 | Verify User Collection | GET | `/Account/v1/User/{userId}` | 200 OK — confirms book persisted |
| 7 | Remove Book | DELETE | `/BookStore/v1/Book` | 204 No Content |
| 8 | Remove User Account | DELETE | `/Account/v1/User/{userId}` | 204 No Content — clean teardown |

**20/20 assertions passing** across all 8 requests, run automatically per commit.

### Negative tests

A separate **Negative Tests** folder verifies that the API rejects invalid input and unauthorized access with the correct errors.

| # | Scenario | What it verifies |
|---|----------|------------------|
| 1 | Register with a weak password | Password-complexity rules are enforced; user is not created |
| 2 | Register a duplicate user | Existing username is rejected (`User exists!`) |
| 3 | Generate token with wrong password | Authentication fails and no token is issued |
| 4 | Add a book with an invalid ISBN | Non-existent ISBN is rejected; collection is unchanged |
| 5 | Request without Authorization header | Protected endpoint refuses access (`User not authorized!`) |
| 6 | Remove a book not in the user's collection | API returns an error instead of silently succeeding |

**13/13 assertions passing** across the 6 negative requests.

**Design notes**
- The duplicate-user test relies on a dedicated fixture account that is registered once and never re-registered or deleted, so the scenario stays repeatable.
- Positive-flow users are generated dynamically per run, so the two suites never collide.
- Negative tests run in the same Newman/GitHub Actions pipeline as the positive flow.

## CI/CD Pipeline

A GitHub Actions workflow (`.github/workflows/api-tests.yml`) runs the full suite (positive + negative) on every push to `main`:

1. Checks out the repo and sets up Node.js
2. Installs Newman + the HTML reporter
3. Runs the collection headlessly against the exported environment
4. Uploads an HTML test report as a build artifact

Because the `Register` step generates a unique username per run (`qa_tester_<timestamp>`), the whole suite is **idempotent** — it can run any number of times without manual cleanup or "user already exists" failures.

## Key Findings & Debugging Notes

Real issues encountered and resolved during this project, documented here to show root-cause analysis rather than just execution:

1. **Case-sensitive variable mismatch** — Referencing `{{userId}}` while the environment variable was actually named `userID` caused a `1207 — User Id not correct!` error. A consistent naming convention was adopted to prevent recurrence.
2. **Local Vault inaccessible on the web client** — Auto-securing the token via Postman's Secret Scanner moved it into Local Vault storage, which only resolves through the Desktop App/Agent. On the browser client this silently produced an empty Authorization header. Fixed by keeping auth values as plain environment variables and refreshing them via script instead.
3. **Environment export stripped all values** — A Postman export from a session with an unresolved sync conflict produced a `.postman_environment.json` where every variable's `"value"` field was blank, causing the CI run to fail with `password required` even though the UI displayed the values correctly. Fixed by hand-writing the environment file with explicit values rather than trusting the Export button in that state.
4. **YAML indentation broke the workflow trigger** — An extra level of indentation under `on:` merged `push` into `workflow_dispatch`, producing `No event triggers defined in on`. Fixed by aligning both keys at the same level.
5. **Hardcoded test user broke repeat runs** — The original collection used a fixed username, so any CI re-run failed with `406 — User Exists!`. Fixed with a pre-request script that generates a unique username per run from `Date.now()`.

## Automation Highlights

**Dynamic user registration** (Register → Pre-request Script):
```javascript
pm.environment.set("user name", "qa_tester_" + Date.now());
```

**Automatic token capture** (GenerateToken → Post-response Script):
```javascript
const response = pm.response.json();
pm.environment.set("token", response.token);
```

## How to Use This Collection

**In Postman:**
1. Import `My Collection.postman_collection.json`.
2. Import `Testing environment.postman_environment.json` and select it as the active environment.
3. Run the positive flow in order (1 → 8), then the **Negative Tests** folder — or use Collection Runner to execute everything at once.

**From the command line (Newman):**
```bash
npm install -g newman newman-reporter-htmlextra
newman run "My Collection.postman_collection.json" \
  -e "Testing environment.postman_environment.json" \
  -r cli,htmlextra --reporter-htmlextra-export report.html
```

**Via CI:** push to `main`, or trigger manually from the **Actions** tab (`workflow_dispatch`).

## Skills Demonstrated

- Manual & functional REST API testing (GET/POST/DELETE)
- Positive and negative test design: input validation, duplicate handling, invalid credentials, missing/invalid authorization, invalid resource references
- Bearer Token authentication flow testing, including error paths
- Environment & variable management in Postman
- Pre/post-response scripting (JavaScript) for automation and dynamic test data
- Test-data management (dynamic users plus a stable fixture account)
- CLI test automation with Newman
- CI/CD pipeline configuration with GitHub Actions (YAML)
- Root-cause debugging of tooling, environment, and pipeline issues
- Test documentation with reproducible steps and evidence
