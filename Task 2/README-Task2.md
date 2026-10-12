# Task 2: Request Chaining & State Management (Postman)

Part of the Barakah TechLabs **API Testing Specialist** internship (QA & Testing).

This project builds a Postman workflow where data returned by one request (an auth token and a newly created booking ID) is extracted automatically and reused in later requests. No values are hardcoded between steps.

**API under test:** [Restful-Booker](https://restful-booker.herokuapp.com), a public practice API that really persists data, so the full create, read, delete lifecycle can be verified.

---

## The chain

| # | Request | Purpose | Tests | Variable set |
|---|---------|---------|-------|--------------|
| 1 | `POST /auth` | Log in and get a token | Status 200; token is a non-empty string | `authToken` |
| 2 | `GET /booking` | List bookings (array parsing) | Status 200; response is a non-empty array | `existingId` |
| 3 | `POST /booking` | Create a booking | Status 200; response contains the data that was sent | `bookingId` |
| 4 | `GET /booking/{{bookingId}}` | Fetch the booking just created | Status 200; fields match what was created | none |
| 5 | `DELETE /booking/{{bookingId}}` | Delete it (uses `authToken` as a cookie) | Status 201 | none |
| 6 | `GET /booking/{{bookingId}}` | Confirm it is gone | Status 404 | none |

**Result:** 9 tests, 9 passed, 0 failed.

```
POST /auth ──► authToken ─────────────────────────┐
GET /booking ─► existingId                         │
POST /booking ─► bookingId ─► GET /booking/{id}    │
                         └──► DELETE /booking/{id} ◄┘ (Cookie: token=authToken)
                         └──► GET /booking/{id} → 404
```

## Techniques demonstrated

- **Extracting a value from nested JSON.** The create response is shaped like `{ "bookingid": 868, "booking": { ... } }`. The script reads `bookingid` and stores it with `pm.environment.set("bookingId", ...)`.
- **Parsing an array.** `GET /booking` returns an array of objects. The script maps it to a list of IDs and stores the first one.
- **Passing state through URLs and headers.** `{{bookingId}}` is used in the request paths, and `{{authToken}}` is sent in a `Cookie: token={{authToken}}` header.
- **Asserting on dynamic data.** Tests use `pm.expect` (Chai BDD syntax) against the response of the previous step.
- **Verifying cleanup.** The final GET proves the DELETE actually removed the record.

## Notes on API behaviour

- `DELETE /booking/{id}` returns **201 Created** on this API instead of the more common 200 or 204. The test asserts 201 on purpose.
- Restful-Booker is a shared public server, so it can occasionally be slow or reset. If a run fails on the first request, rerun the whole collection rather than a single request, because the token and booking ID are regenerated on every full run.

## How to run

1. Import both files from this repo into Postman:
   - `Task2-RestfulBooker.postman_collection.json`
   - `Booker-Env.postman_environment.json`
2. Select the **Booker-Env** environment (top-right dropdown).
3. Open the collection, click **Run**, keep all requests ticked in their original order, and click **Run** again.
4. Expected output: 9 of 9 tests passing, with `authToken` and `bookingId` populated in the environment afterwards.

Always run the whole collection in order. Running the requests out of order will fail, because later requests depend on variables set by earlier ones.

## Repository structure

```
.
├── Task2-RestfulBooker.postman_collection.json
├── Booker-Env.postman_environment.json
├── screenshots/
│   ├── runner-results.png
│   └── environment-variables.png
└── README.md
```

## Tools

Postman (Collection Runner), JavaScript (Postman scripting with `pm.*` and Chai assertions)

## Author

**M Faizan Khan**
GitHub: [faixankh07-dvader](https://github.com/faixankh07-dvader)
