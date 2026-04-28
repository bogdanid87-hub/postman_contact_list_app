# Contact List App — API Test Suite

A Postman API test collection for the [Contact List App](https://thinking-tester-contact-list.herokuapp.com/), built as part of a QA portfolio project targeting fintech roles.

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
| User | Full user lifecycle — get, update credentials, logout, delete account |
| Negative Tests - Contact - Authenticated | Missing fields, invalid ID, invalid postcode validation |
| Negative Tests - Logged Out | Verify all user and contact endpoints are protected after logout |
| Security | NoSQL injection, BOLA testing — verify User 2 cannot access or modify User 1's data |
| Cleanup | Delete test contacts and accounts after each run |

## Types of Tests

- **Functional** — status codes, response values, field validation
- **Contract** — JSON schema validation on key endpoints to catch breaking API changes
- **Security** — BOLA/IDOR checks verifying data isolation between users
- **Security** - NoSQL Injection — MongoDB operator injection attempts (`$gt`, `$where`, `$regex`) on login and contact endpoints to verify inputs are sanitised before reaching the database
- **Negative** — invalid inputs, missing auth, expired tokens, duplicate data

## How to Run

1. Download the `Contact List App — QA Portfolio.postman_collection.json` file from this repository
2. Import the collection into Postman
3. Use **Collection Runner** to execute all tests in order

## Collection Variables

| Variable | Description | Set by |
|---|---|---|
| `base_url` | API base URL | Pre-configured |
| `token` | JWT token for User 1 | Login request |
| `userId` | User 1's ID | Login request |
| `contactId` | User 1's first contact ID | Add Contact request |
| `contactId2` | User 1's second contact ID | Add Contact 2 request |
| `token2` | JWT token for User 2 | Login User 2 request |
| `userId2` | User 2's ID | Login User 2 request |

## Known Bugs Found

| # | Endpoint | Description | Severity |
|---|---|---|---|
| 1 | `PATCH /users/me` | Raw MongoDB error exposed on duplicate email instead of clean 400 | Medium |
| 2 | `PATCH /contacts/` | Server returns 503 Service Unavailable when contact ID is missing from the URL — empty path parameter causes unhandled exception instead of a clean 400 | Medium |


## Notes

- Tokens and IDs are automatically saved and chained between requests via collection variables
- No environment required — all variables are stored at collection level
- The Cleanup folder runs at the end of every Collection Runner execution to remove test data
- Security tests use a second user account (`hacker@example.com`) to simulate BOLA attacks
- NoSQL injection via `$gt` operator in contact fields is blocked by Mongoose schema type validation rather than explicit input sanitisation. Fields accepting object types may remain vulnerable.
- Bug #2 reproduction steps:
  > Clear the `contactId` collection variable, 
  > Send `PATCH {{base_url}}/contacts/` with a valid auth token. 
  > The server returns a Heroku HTML error page instead of a `400 Bad Request`
