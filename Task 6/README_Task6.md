# Task 6: Command-Line Execution with Newman

Part of the Barakah TechLabs **API Testing Specialist** internship (QA & Testing).

A 10-request Postman test suite for the [Restful-Booker](https://restful-booker.herokuapp.com) API, exported with its environment and run entirely from the terminal with **Newman**. The run produces a rich HTML report (htmlextra) and a JUnit XML file.

## Result

| Metric | Value |
|---|---|
| Requests | 10 (0 failed) |
| Assertions | 14 (0 failed) |
| Total run duration | 4.4 s |
| Average response time | 361 ms |

## The suite

| # | Request | What it checks |
|---|---|---|
| 1 | `GET /ping` | API health check returns 201 |
| 2 | `POST /auth` (bad credentials) | No token is returned for wrong credentials |
| 3 | `POST /auth` | Status 200, token is a non-empty string, saved to `authToken` |
| 4 | `GET /booking` | Status 200 on the booking list |
| 5 | `POST /booking` | Booking created, response data correct, ID saved to `bookingId` |
| 6 | `GET /booking/{{bookingId}}` | Fetched booking matches what was created |
| 7 | `PUT /booking/{{bookingId}}` | Update succeeds with a valid token and the new values are returned |
| 8 | `DELETE /booking/{{bookingId}}` (no token) | Rejected with 403 while the booking still exists |
| 9 | `DELETE /booking/{{bookingId}}` (with token) | Deleted, returns 201 |
| 10 | `GET /booking/{{bookingId}}` | Booking is gone, returns 404 |

The requests run in order and depend on each other: the token from request 3 and the booking ID from request 5 are passed to later requests through environment variables. The negative test (8) deliberately runs before the real delete, so it checks protection on an existing booking.

## Files

- `Task 6 - Newman Execution.postman_collection.json` (exported collection, v2.1)
- `Booker-Env.postman_environment.json` (exported environment; `baseUrl` is set, and `authToken` and `bookingId` start empty and are filled during the run)
- `reports/Task6-report.html` (htmlextra report)
- `reports/Task6-junit.xml` (JUnit report for CI tools)

## How to run

1. Install Node.js, then:
   ```
   npm install -g newman newman-reporter-htmlextra
   ```
2. From the repository folder:
   ```
   mkdir reports
   newman run "Task 6 - Newman Execution.postman_collection.json" -e Booker-Env.postman_environment.json -r cli,htmlextra,junit --reporter-htmlextra-export "reports/Task6-report.html" --reporter-htmlextra-title "Restful-Booker API Test Suite" --reporter-junit-export "reports/Task6-junit.xml"
   ```
3. Open `reports/Task6-report.html` in a browser.

### What each part does

| Part | Purpose |
|---|---|
| `-e` | Loads the exported environment file |
| `-r cli,htmlextra,junit` | Prints results in the terminal and writes the HTML and XML reports |
| `--reporter-htmlextra-export` | Sets where the HTML report is written |
| `--reporter-htmlextra-title` | Sets the report title |
| `--reporter-junit-export` | Sets where the JUnit XML is written |

Newman returns exit code `0` when every assertion passes and a non-zero code when any assertion fails. That is the behavior a CI pipeline uses to fail a build, and the reason the JUnit output is included.

## API behaviors noted

- `GET /ping` returns **201 Created**, not 200.
- A login with bad credentials returns **200 OK** with no token, not a 4xx error. The test checks that no token is present.
- `DELETE` returns **201 Created** on success.
- `PUT` and `DELETE` accept the token as a cookie (`Cookie: token=...`). Without it, both return 403.

## Limitations

- The suite runs a single iteration with fixed data, so it does not cover edge-case inputs. Those are covered by the data-driven suite in Task 3.
- The `GET /booking` check only asserts the status code.
- Tests run against a shared public server, so occasional slowness or resets are outside the suite's control.

## Tools

Postman, Newman, newman-reporter-htmlextra, JavaScript (Postman scripting and Chai assertions)

## Author

**M Faizan Khan**
GitHub: [faixankh07-dvader](https://github.com/faixankh07-dvader)
