# DemoQA Bookstore API — Manual & Scripted Test Collection

A complete, hands-on API testing project built with **Postman**, covering the full lifecycle of the [DemoQA Bookstore API](https://demoqa.com/swagger/): user registration, token-based authentication, and CRUD operations on a protected book collection.

## Overview

This project demonstrates end-to-end functional and negative testing of a REST API that uses Bearer Token authentication. It includes an 8-step test flow, environment-based variable management, and a pre-request/post-response script that automates token extraction — removing manual copy-paste and the errors that come with it.

## Tech Stack

- **Tool:** Postman (Collections, Environments, Scripts, Console)
- **API under test:** `demoqa.com` — Account & BookStore REST API
- **Auth type:** Bearer Token (JWT)

## Test Flow & Coverage

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

## Key Findings & Debugging Notes

Real issues encountered and resolved during this project (documented here to show root-cause analysis, not just execution):

1. **Case-sensitive variable mismatch** — Referencing `{{userId}}` in a request body while the environment variable was actually named `userID` caused a `1207 — User Id not correct!` error. Postman treats variable names as case-sensitive; a naming convention was adopted across the collection to prevent recurrence.
2. **Local Vault inaccessible on web client** — Automatically "securing" the token via Postman's Secret Scanner moved it into **Local Vault** storage, which is only accessible from the Postman Desktop App/Agent. On the browser client this caused the Authorization header to resolve empty, producing a `1200 — User not authorized!` error despite a valid token. Fix: kept the token as a plain (non-vault) environment variable and automated its refresh via script instead.
3. **Manual token copy-paste risk** — Long JWTs wrapped across multiple lines in the response viewer are error-prone to copy by hand. Solved by adding a post-response script to extract and persist the token automatically (see below).

## Automation Highlight — Post-response Script

Added to the `GenerateToken` request to eliminate manual copy-paste of the auth token:

```javascript
const response = pm.response.json();
pm.environment.set("token", response.token);
console.log("Token saved:", response.token);
```

## How to Use This Collection

1. Import `DemoQA-Bookstore-API.postman_collection.json` into Postman.
2. Import `Testing-environment.postman_environment.json` and select it as the active environment.
3. Run requests in order (1 → 8). Each step depends on variables (`userID`, `token`, `isbn`) set automatically by earlier steps.

## Skills Demonstrated

- Manual & functional REST API testing (GET/POST/DELETE)
- Bearer Token authentication flow testing
- Environment & variable management in Postman
- Pre/post-response scripting (JavaScript) for automation
- Root-cause debugging of tooling and environment issues
- Test documentation with reproducible steps and evidence
