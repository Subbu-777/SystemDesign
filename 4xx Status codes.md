HTTP 4XX Status Codes – Scenarios & Answers

This file contains realistic API scenarios, the correct 4xx status codes, and explanations.
Ideal for interview prep, API design discussions, or self-testing.


---

Scenario 1

Description: Client sends malformed JSON:

{ "name": "Subbu",

Answer: 400 – Bad Request
Explanation: Server cannot parse the request; syntax is invalid.


---

Scenario 2

Description: Valid JSON, but required field accountId is missing.
Answer: 400 – Bad Request
Explanation: Request structure is invalid; required field missing.


---

Scenario 3

Description: Field mode accepts only A or B, client sends C.
Answer: 422 – Unprocessable Entity
Explanation: Request is well-formed, but value violates validation rules.


---

Scenario 4

Description: Client sends 1000 requests per minute; API limit is 100.
Answer: 429 – Too Many Requests
Explanation: Rate limit exceeded; client must slow down.


---

Scenario 5

Description: User is not authenticated (missing or invalid token).
Answer: 401 – Unauthorized
Explanation: Authentication required; client not identified.


---

Scenario 6

Description: User is authenticated but does not have permission to access the resource.
Answer: 403 – Forbidden
Explanation: Client is known, but access is denied.


---

Scenario 7

Description: Resource with given ID does not exist.
Answer: 404 – Not Found
Explanation: Requested resource cannot be found on the server.


---

Scenario 8

Description: Request is valid, but business rule fails (withdrawal amount > account balance).
Answer: 422 – Unprocessable Entity
Explanation: Request understood, but violates business logic.


---

Scenario 9

Description: Well-formed JSON with all required fields, but one string field exceeds maximum length.
Answer: 422 – Unprocessable Entity
Explanation: Field value violates validation constraints.


---

Scenario 10

Description: Client sends GET request to an endpoint that only allows POST.
Answer: 405 – Method Not Allowed
Explanation: HTTP method is invalid for this endpoint; include Allow header.


---

Scenario 11

Description: Client request conflicts with server state (duplicate user, stale version).
Answer: 409 – Conflict
Explanation: Server cannot process request due to conflict with current state.


---

Scenario 12

Description: Client took too long to send the request (upload stalled).
Answer: 408 – Request Timeout
Explanation: Client-side delay; request not fully received in time.


---

Quick Reference Table

Status	When to use

400	Bad request: syntax / missing fields
401	Unauthorized: missing / invalid authentication
403	Forbidden: authenticated but not allowed
404	Not Found: resource does not exist
405	Method Not Allowed: invalid HTTP method
409	Conflict: request conflicts with server state
422	Unprocessable Entity: validation / semantic errors
429	Too Many Requests: rate limiting exceeded
408	Request Timeout: client too slow



---

