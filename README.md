# Contact List App — API Test Suite

A Postman API test collection for the [Contact List App](https://thinking-tester-contact-list.herokuapp.com), built as part of a QA portfolio project.

## Project Overview

This collection covers end-to-end API testing including authentication, full CRUD operations on contacts, user lifecycle management, and negative/security testing for both authenticated and unauthenticated scenarios.

## Tech Stack

- **Tool:** Postman
- **API Under Test:** [Contact List App](https://thinking-tester-contact-list.herokuapp.com)
- **Auth:** JWT Bearer Token
- **API Docs:** [Swagger](https://thinking-tester-contact-list.herokuapp.com/api-docs/)

## Test Coverage

| Folder | Description |
|---|---|
| Registration and Auth | Register, login, negative login scenarios |
| Contacts | Full CRUD — create, read, update, delete |
| Negative Tests - Contact - Authenticated | Missing fields, invalid ID, invalid postcode |
| Negative Tests - Logged Out | Verify user endpoints and contacts are protected after logout |
| User | Full user lifecycle — update credentials, logout, delete account |

## How to Run the tests

1. Download the `Contact List App — QA Portfolio.postman_collection.json` file from this repository.
2. Open the collection in Postman
3. Use **Collection Runner** to execute all tests in order

## Collection Variables

| Variable | Description | Set by |
|---|---|---|
| `base_url` | API base URL | Pre-configured |
| `token` | JWT auth token | Login request |
| `userId` | Logged-in user ID | Login request |
| `contactId` | Created contact ID | Add Contact request |

## Known Bugs Found

| # | Endpoint | Description | Severity |
|---|---|---|---|
| 1 | `PATCH /users/me` | Raw MongoDB error exposed on duplicate email | Medium |

## Notes

- Tokens are automatically saved and chained between requests via collection variables
- No environment required — all variables are stored at collection level
