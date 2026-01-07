# HTTP 5XX and 2XX Status Codes – Scenarios & Answers

This file contains **realistic server-side (5xx) and success (2xx) HTTP status codes**, example scenarios, and explanations. Ideal for API design reference, interview prep, or self-testing.

---

## 5XX Server Error Scenarios

| Scenario | Status | Explanation |
|----------|-------|------------|
| Null pointer exception in server code | 500 | Generic server error; unhandled exception occurred |
| API gateway receives 500 from microservice | 502 | Bad Gateway; upstream server returned invalid response |
| Server under maintenance | 503 | Service temporarily unavailable; try later |
| Microservice does not respond in time | 504 | Gateway Timeout; upstream server took too long |
| Client requests HTTP/2, server supports only HTTP/1.1 | 505 | HTTP version not supported |
| File upload fails due to disk full | 507 | Insufficient storage |
| Client requests unsupported API endpoint (rare) | 501 | Not Implemented; server cannot handle requested functionality |

### Quick Reference Table – 5XX

| Status | Name | Memory Hint | Example |
|--------|------|------------|---------|
| 500 | Internal Server Error | Something went wrong on server | Unhandled exception, DB connection fails |
| 501 | Not Implemented | I can’t do that | Unsupported API/method |
| 502 | Bad Gateway | Upstream failure | API gateway receives 500 from microservice |
| 503 | Service Unavailable | Try later | Server under maintenance / overloaded |
| 504 | Gateway Timeout | Upstream timeout | API gateway times out waiting for DB/microservice |
| 505 | HTTP Version Not Supported | Version mismatch | Client requests HTTP/2, server only supports 1.1 |
| 507 | Insufficient Storage | Disk full / quota exceeded | File upload fails due to storage limit |

---

## 2XX Success Scenarios

| Scenario | Status | Explanation |
|----------|-------|------------|
| GET /users returns full user list | 200 | Request succeeded; returns data |
| POST /users creates a new user | 201 | Resource successfully created |
| Async request queued for processing | 202 | Accepted but processing not yet complete |
| DELETE /users/123 succeeds with no response | 204 | Success, no content to return |
| PUT form submission succeeded, client UI should reset | 205 | Success; client should reset document/view |
| Download part of a video file (range request) | 206 | Partial content; byte-range request |
| Proxy modifies response from upstream | 203 | Non-authoritative info; from third-party source |

### Quick Reference Table – 2XX

| Status | Name | Memory Hint | Example |
|--------|------|------------|---------|
| 200 | OK | All good | GET /users returns list |
| 201 | Created | New resource created | POST /users |
| 202 | Accepted | Accepted, processing later | Async queueing |
| 203 | Non-Authoritative Info | Info from third-party | Proxy-modified response |
| 204 | No Content | Nothing to return | DELETE /users/123 |
| 205 | Reset Content | Reset form/view | PUT form submission |
| 206 | Partial Content | Partial data | Byte-range file download |

---
