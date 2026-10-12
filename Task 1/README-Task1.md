# Task 1: Core API Interaction & Assertions

A Postman collection that tests the public [JSONPlaceholder](https://jsonplaceholder.typicode.com) REST API using `pm.expect` (Chai BDD) assertions. It covers status codes, headers, response times, nested JSON, array validation, and one negative test.

![Run results](run-results.png)

## Summary

| | |
|---|---|
| Requests | 10 |
| Test results per run | 61 (41 request-specific + 2 collection-level tests on each of the 10 requests) |
| Methods covered | GET, POST, PUT, PATCH, DELETE |
| API under test | `https://jsonplaceholder.typicode.com` |
| Tools | Postman, JavaScript (`pm.*` scripting API, Chai assertions) |

## Collection-level tests

Two tests are defined once on the collection and run after every request:

- Response time is under 1000 ms
- `Content-Type` header includes `application/json`

## Requests and assertions

| # | Request | What is asserted |
|---|---|---|
| 1 | `GET /posts/1` | Status 200; body has exactly `userId`, `id`, `title`, `body`; `id` equals 1; `title` is a non-empty string |
| 2 | `GET /posts` | Status 200; array of 100 items; every item has the expected keys; all ids are unique numbers |
| 3 | `GET /users/1` | Status 200; top-level keys; `id` is 1 and `name` is a non-empty string; email matches a regex; nested `address` keys and `address.geo.lat/lng` exist; `company.name` exists |
| 4 | `GET /posts?userId=1` | Status 200; array of 10; every item has `userId` 1; expected keys; ids are 1 to 10 |
| 5 | `GET /comments?postId=1` | Status 200; array of 5; every item has `postId` 1; expected keys; every email matches a regex |
| 6 | `POST /posts` | Status 201; response echoes the title, body and userId that were sent; `id` is a number; exact key set |
| 7 | `PUT /posts/1` | Status 200; response reflects the updated data; `id` unchanged; exact key set |
| 8 | `PATCH /posts/1` | Status 200; patched field updated; fields not sent are still present; exact key set |
| 9 | `DELETE /posts/1` | Status 200; body is an empty object; no post data returned |
| 10 | `GET /posts/99999` (negative) | Status 404; body is an empty object; no post data returned |

## Concepts practiced

- `pm.test` and `pm.expect` with Chai: `.to.equal`, `.to.eql`, `.to.have.property`, `.to.have.all.keys`, `.to.have.members`, `.to.be.a`, `.to.match`, `.to.be.below`, `.with.lengthOf`
- Looping over arrays with `forEach` to validate every item, not just the first
- Nested JSON access (`user.address.geo`)
- Comparing a response against the request body sent (`pm.request.body.raw`)
- Collection variable (`{{baseUrl}}`) and collection-level scripts to avoid repetition
- Negative testing (a correct 404 response counts as a pass)
- Confirming that each test can fail, by temporarily changing expected values

## How to run

1. Import `Task-1-Core-Assertions.postman_collection.json` into Postman (Import, then drop the file).
2. Confirm the collection variable `baseUrl` is `https://jsonplaceholder.typicode.com`.
3. Click the collection, then **Run**, then **Run Task 1 - Core Assertions**.
4. Expected result: 10 requests, 61 tests passed, 0 failed.

## Limitations

- **JSONPlaceholder simulates writes.** POST, PUT, PATCH and DELETE return realistic responses, but nothing is actually saved. A later `GET` returns the original data. These requests therefore test the response shape and echoed values, not persistence. Persistence is covered in Task 2 with an API that stores data.
- **DELETE on a nonexistent id returns 200** instead of 404. The DELETE test reflects this API's behavior, not what a production API should do.
- Response-time checks use a generous 1000 ms threshold because the API is public and shared. Stricter SLA checks come in Task 8.
