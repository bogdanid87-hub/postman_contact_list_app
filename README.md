# Contact List App — API Test Suite

A Postman API test collection for the [Contact List App](https://thinking-tester-contact-list.herokuapp.com/), built as part of a QA portfolio project.

## Project Overview

This collection covers end-to-end API testing including authentication, full CRUD operations on contacts, user lifecycle management, negative testing for authenticated and unauthenticated scenarios, contract validation, and BOLA (Broken Object Level Authorization) security testing.

## Tech Stack

- **Tool:** Postman
- **API Under Test:** [Contact List App](https://thinking-tester-contact-list.herokuapp.com/)
- **Auth:** JWT Bearer Token
- **API Docs:** [Postman Contact List Documentation](https://documenter.getpostman.com/view/4012288/TzK2bEa8/)

## Test Coverage

| Folder | Description |
|---|---|
| Registration and Auth | Register, login, duplicate email, negative login scenarios |
| Contacts | Full CRUD — create, read, update, delete with contract validation |
| User | Full user lifecycle — get, update credentials, verify old credentials are invalidated, logout, delete account |
| Negative Tests - Contact - Authenticated | Missing fields, invalid ID, invalid postcode, unsupported Content-Type (XML observation) |
| Negative Tests - Logged Out | Verify all user and contact endpoints reject invalidated tokens after logout |
| Security | NoSQL injection, BOLA testing — verify User 2 cannot access or modify User 1's data; brute force login — verify rate limiting behaviour |
| Cleanup | Delete test contacts and accounts after each run |

## Types of Tests

- **Functional** — status codes, response values, field validation
- **Contract** — JSON schema validation on key endpoints to catch breaking API changes
- **Security** — NoSQL Injection: MongoDB operator injection attempts (`$gt`, `$where`, `$regex`) on login and contact endpoints to verify inputs are sanitised before reaching the database; BOLA/IDOR checks verifying data isolation between users; brute force login simulation to surface rate limiting gaps
- **Negative** — invalid inputs, missing auth, invalidated tokens, duplicate data
- **Observation** — behaviour that is not strictly a bug but deviates from hardened API standards; documented inline with `[OBSERVATION]` tags and passing tests that describe actual behaviour

## How to Run

1. Download the `Contact List App — QA Portfolio.postman_collection.json` file from this repository
2. Import the collection into Postman
3. Use **Collection Runner** to execute all tests in order

## Collection Variables

| Variable | Description | Set by |
|---|---|---|
| `base_url` | API base URL | Pre-configured |
| `token` | JWT token for User 1 | Login User 1 request |
| `userId` | User 1's ID | Login User 1 request |
| `contactId` | User 1's first contact ID | Add Contact request |
| `contactId2` | User 1's second contact ID | Add Contact 2 request |
| `token2` | JWT token for User 2 | Login User 2 request |
| `userId2` | User 2's ID | Login User 2 request |
| `token3` | JWT token for User 3 | Login User 3 request |
| `userId3` | User 3's ID | Login User 3 request |
| `loginAttempts` | Counter for brute force loop | Security folder pre-request script |

All variables are set automatically. No manual setup or environment file is required.

## Bugs and Observations Found

| # | Tag | Endpoint | Description | Severity |
|---|---|---|---|---|
| 1 | `[KNOWN BUG]` | `PATCH /users/me` | Raw MongoDB duplicate key error exposed on update with an already-registered email, instead of a clean 400 with a user-facing message. Leaks internal database implementation details | Medium |
| 2 | `[KNOWN BUG]` | `PATCH /contacts/` & `DELETE /contacts/` | Server returns 503 Service Unavailable when the contact ID is missing from the URL — empty path parameter causes an unhandled exception instead of a clean 400 | Medium |
| 3 | `[KNOWN BUG]` | `POST /users` | API accepts special characters and symbols in `firstName` and `lastName` fields — no input validation, returns 201 | Low |
| 4 | `[OBSERVATION]` | `POST /contacts` | API does not enforce `Content-Type` — sending `application/xml` with an XML body produces the same 400 validation error as an empty body. The XML is silently discarded by Express's JSON middleware rather than rejected with 415 Unsupported Media Type | Low |
| 5 | `[OBSERVATION]` | Multiple endpoints | Mongoose version key (`__v`) is present in several response bodies. ORM internals should ideally be suppressed before reaching the client | Low |

Bug #2 reproduction steps:
> Clear the `contactId` collection variable,
> Send `PATCH {{base_url}}/contacts/` or `DELETE {{base_url}}/contacts/` with a valid auth token.
> The server returns a Heroku HTML error page instead of a `400 Bad Request`.

## Notes

- Tokens and IDs are automatically saved and chained between requests via collection variables
- No environment required — all variables are stored at collection level
- The Cleanup folder runs at the end of every Collection Runner execution to remove test data
- Security tests use a dedicated second user account (User 2) to simulate BOLA attacks — intentionally obscure email addresses are used to avoid conflicts on this shared public API
- NoSQL injection via `$gt` operator in contact fields is blocked by Mongoose schema type validation rather than explicit input sanitisation. Fields accepting object types may remain vulnerable
- All bugs and observations are documented inline within the relevant test scripts — response examples will be added in a future update
