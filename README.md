# Contact List App - API Test Suite

[![Contact List API Tests](https://github.com/bogdanid87-hub/postman_contact_list_app/actions/workflows/newman.yml/badge.svg)](https://github.com/bogdanid87-hub/postman_contact_list_app/actions/workflows/newman.yml)

A Postman API test collection for [Contact List App](https://thinking-tester-contact-list.herokuapp.com/) designed to be run end-to-end using the collection runner.

---

## Project Overview

The collection covers end-to-end API testing including authentication, CRUD operations on user and contacts, negative testing, contract validation and security testing.

## Tech Stack

| | |
|---|---|
| Tool | Postman |
| API under test | [Contact List App](https://thinking-tester-contact-list.herokuapp.com/) |
| API docs | [Postman Contact List Documentation](https://documenter.getpostman.com/view/4012288/TzK2bEa8) |
| Auth | JWT Bearer Token |

## Test Coverage

| Folder | Description |
|---|---|
| Registration and Auth | Register, login and negative testing scenarios |
| Contacts | Post, Get, Patch |
| Negative Tests - Contact - Authenticated | Negative tests for Post, Get, Delete, Patch contacts |
| User | Post, Get, Patch, including negative tests |
| Negative Tests - Logged out | Checking the invalidated token is correctly rejected, Post, Get, Patch, Delete |
| Security | NoSQL injections, verifying User 2 cannot access User 1 data, brute force login to check rate limiting behaviour |
| Cleanup | Checking Delete for user and contacts, as well as cleaning up the database after each run |

## Types of Tests

- **Functional** - status codes, response values, field validation
- **Contract** - JSON schema validation
- **Security** - NoSQL injection, BOLA to check data isolation, brute force attempt to check rate limiting
- **Negative** - invalid inputs, missing auth, invalidated token, duplicate data

## How to Run

- Download the `Contact List App — QA Portfolio.postman_collection.json` from this repo
- Import the collection into Postman
- Use the **Collection Runner** to execute all tests in order

## Collection Variables

| Variable | Description | Set by |
|---|---|---|
| base_url | API base URL | Pre-configured |
| token | JWT token for User 1 | Login User 1 request |
| userId | User 1 ID | Login User 1 request |
| contactId | User 1 first contact ID | Add Contact request |
| contactId2 | User 1 second contact ID | Add Contact 2 request |
| token2 | JWT token for User 2 | Login User 2 request |
| userId2 | User 2 ID | Login User 2 request |
| token3 | JWT token for User 3 | Login User 3 request |
| userId3 | User 3 ID | Login User 3 request |
| loginAttempts | Counter for brute force loop | Security folder pre-request script |

All variables are set automatically — no manual setup or environment required.

## Bugs and Observations

| # | Type | Endpoint | Description | Severity |
|---|---|---|---|---|
| 1 | Bug | POST /users | API accepts special characters for names; no input sanitisation | Low |
| 2 | Observation | POST /contacts | API does not enforce content type; sending an XML body produces the same 400 error as an empty body instead of being rejected with 415 Unsupported Media Type | Low |
| 3 | Bug | PATCH /user/me | Raw MongoDB error exposed when trying to register with an email already in use, leaking internal DB implementation | High |
| 4 | Vulnerability | POST /users/login | No rate limitation when multiple failed login attempts occur in a short time | Medium |
| 5 | Observation | Multiple endpoints | Mongoose version key (`__v`) is present in several responses | Low |
| 6 | Bug | PATCH /contacts & DELETE /contacts | Server returns 503 Service Unavailable when the contact ID is missing from the URL instead of a 4xx error | Medium |

**Bug 6 — Reproduction steps:**

1. Clear the `contactId` collection variable
2. Send a `PATCH {{base_url}}/contacts/` or `DELETE {{base_url}}/contacts/` with or without a valid auth token
3. The server returns a Heroku HTML error instead of a 4xx error

## Notes

- Tokens and IDs are stored automatically and chained between requests via Collection Variables
- No environment needed
- Cleanup folder runs at the end of every Collection Runner execution to clear the testing data
- The Brute Force Login test works correctly only when run through the Collection Runner
- NoSQL injections are blocked by Mongoose schema validation rather than explicit input sanitisation
- In case tests have been run out of order, the collection will self-heal after being fully run through the Collection Runner
- Saved response examples are included for selected requests to document expected API behaviour, including for the issues mentioned in the Bugs and Observations table
- The collection is also configured to run automatically whenever the JSON is updated, using Newman; results and HTML reports are available in the Actions tab
