# REST API Design Principles

**74 principles across 9 sections** — a consistent, predictable standard for designing HTTP APIs across our platform.

## Contents

1. [REST API Anatomy](#1-rest-api-anatomy) — principles 1–14
2. [Error Handling](#2-error-handling) — principles 15–25
3. [Versioning](#3-versioning) — principles 26–28
4. [Security](#4-security) — principles 29–48
5. [Performance](#5-performance) — principles 49–56
6. [Idempotency](#6-idempotency) — principles 57–59
7. [Asynchronous & Long-Running Operations](#7-asynchronous--long-running-operations) — principles 60–63
8. [Know Your Consumers](#8-know-your-consumers) — principles 64–69
9. [Documentation](#9-documentation) — principles 70–74
10. Deprecation policy
11. Observability

---

## 1. REST API Anatomy

### URI — Resource-Oriented

REST is built on a **resource** paradigm rather than an **operation** paradigm. This is the key difference from RPC-style approaches such as SOAP, which are organized around operations and methods (e.g. `getUser`, `createOrder`). The name itself says it: you are designing a **Uniform _Resource_ Identifier** — the thing you identify and address is a *resource* (a noun), not an action.

1. **Hierarchy reflects ownership.** Use nested paths to express containment relationships (e.g. `/users/42/orders` — the orders belonging to user 42).

2. **Put resource identifiers in path parameters.** Identifiers that select *which* resource you're acting on belong in the path — not in the request body or query string. Reserve query parameters for filtering, sorting, and pagination.

3. **Don't nest more than 2 levels deep.** Beyond that the URL becomes unwieldy and tightly couples consumers to your hierarchy. Once a resource has its own ID, prefer the flat form (`/orders/1001` over `/users/42/orders/1001`).

### HTTP Methods & Status Codes

4. **Choose the method that fits the operation.** The HTTP verb should make the intent obvious — `GET` to read, `POST` to create, `PUT`/`PATCH` to update, `DELETE` to remove. A reader should never have to wonder *why* a particular method was chosen to operate on a particular resource.

5. **Use standard status codes; don't invent your own meaning.** Always return the most specific applicable code.
    - **Success (2xx)** — the request succeeded.
    - **Client errors (4xx)** — the caller did something wrong.
        > **`400` vs `422`:** Use `400` for malformed requests (bad JSON, missing required field). Use `422` when the request is well-formed but violates a business rule (e.g. "start date must be before end date"). Pick one convention as a team and apply it uniformly.
    - **Server errors (5xx)** — something failed on our side. Never leak stack traces or internals in the response.

### Request & Response Objects

6. **Always use JSON.** Request and response bodies are `application/json` unless there's a strong reason otherwise (file upload/download, streaming). Set `Content-Type: application/json` on requests that carry a body, and honor the `Accept` header.

7. **Use structured, nested objects — not flat key-value dumps.** Group related fields into nested objects, and represent collections as JSON arrays (`[]`), never as an object with positional or numbered keys. A real array deserializes cleanly into a typed list (`List<Item>`); an object with dynamic keys (`{ "0": {…}, "1": {…} }`) does not, and forces every consumer to special-case it. Nesting also keeps related data together and avoids long prefixed key names — `address: { street, city }` rather than `addressStreet`, `addressCity`.

8. **Model the domain, not one client's screen.** A request shaped around a single frontend form breaks the moment a second consumer appears (mobile app, partner integration, batch importer). Design the payload around the resource and its real attributes, not the UI that happens to submit it first.

9. **Return the created/updated resource.** After `POST`/`PUT`/`PATCH`, return the resulting resource so clients don't need a follow-up `GET`.

10. **Be consistent about envelopes.** Decide as a team whether responses are wrapped in a top-level envelope, then stick to it. An envelope gives you room to grow: you can add metadata later (e.g. pagination details on list/search responses) without restructuring the payload or breaking consumers.

11. **Don't return everything — return only what consumers need.**
    - Never expose database internals, ORM artifacts, or other implementation details.
    - Remember the asymmetry: **adding** a field later is easy and backward-compatible; **removing** one after consumers depend on it is a breaking change. When in doubt, leave it out.

12. **Define "field absent" vs. "field present but null" — especially for PATCH.** Does omitting a field mean leave it unchanged while null means clear it? Or are they treated the same? Conflating the two silently wipes data or silently ignores updates. Pick clear semantics (JSON Merge Patch is a sensible default) and document them.

13. **New request fields must be optional with safe defaults — and never tighten validation on an existing field.** Adding a required field, or making input that used to be valid now invalid, breaks every existing client the moment you deploy. Both are breaking changes, even though it feels like you "only added a rule." Evolve request schemas additively.

#### Standard data formats

14. **Use consistent, unambiguous representations for common data types.**

| Data type | Format | Example |
|-----------|--------|---------|
| Date & time | ISO 8601 / RFC 3339, **UTC**, with `Z` | `2026-06-20T10:30:00Z` |
| Date only | ISO 8601 | `2026-06-20` |
| Duration | ISO 8601 duration | `P1DT2H` |
| Booleans | `true`/`false`, never `"yes"`/`1` | `"isActive": true` |
| Enums | `UPPER_SNAKE_CASE` strings, not integers | `"status": "PENDING_REVIEW"` |
| Null vs. absent | Be deliberate; document the difference | — |

---

## 2. Error Handling

Graceful error handling is essential in a REST API, because APIs are frequently consumed machine-to-machine (M2M) or service-to-service (S2S). When a service returns an unclear or inconsistent error, the calling system can't react correctly — it may retry blindly, fail loudly, or propagate the failure upstream. Poor error handling is a common trigger for *cascading failures* and *retry storms* that take down not just one service but a whole ecosystem.

Two foundations make errors safe to consume:

15. **Always return a structured JSON body alongside the HTTP status code.** The status code alone is ambiguous, because the network path is full of intermediaries — gateways, proxies, load balancers, CDNs — that emit status codes of their own. A bare `404` might mean *your application couldn't find the resource*, or it might mean *an API gateway couldn't match the route and never reached your service at all*; likewise a `503` could be your app or an overloaded load balancer. A consistent, structured error body (with content type `application/problem+json`) plus a stable application `code` lets clients tell an **application-level** error apart from an **infrastructure-level** one and respond appropriately.

16. **Use a single, consistent error shape across the entire platform.** We recommend **RFC 9457 Problem Details** (the successor to RFC 7807):

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/problem+json

{
  "type": "https://errors.ourplatform.com/validation",
  "title": "Validation failed",
  "status": 422,
  "detail": "The request contains invalid fields.",
  "instance": "/orders/1001",
  "traceId": "a1b2c3d4",
  "errors": [
    { "field": "email",    "code": "INVALID_FORMAT", "message": "Must be a valid email address." },
    { "field": "quantity", "code": "OUT_OF_RANGE",   "message": "Must be between 1 and 100." }
  ]
}
```

### Rules

17. **Let the HTTP status code carry the outcome — never return `2xx` for an error.** Returning `200 OK` with an error buried in the body is a classic anti-pattern: it breaks client error handling, monitoring, alerting, and retry logic, all of which key off the status line. If it failed, say so in the status code.

18. **Every error has a stable, machine-readable `code`.** Clients branch on `code`, not on human-readable `message` text (which may be reworded or localized at any time).

19. **Include a `traceId`/correlation ID in every error** so support and engineering can find the exact request in the logs.

20. **Signal whether the error is retryable.** Distinguish transient failures the client may safely retry (typically `429`, `502`, `503`, `504`) from permanent ones it must not (most `4xx` — a retry won't help and only adds load). Include a `Retry-After` header on `429`/`503` so clients back off instead of hammering — the simplest guard against retry storms. Retries on non-idempotent operations (`POST`) still need an idempotency key to avoid duplicates.

21. **Never leak internals** — no stack traces, SQL, internal hostnames, or raw library errors in production responses. They aid attackers and confuse consumers.

22. **Return all validation failures at once.** For field-level validation, report every problem in a single response rather than failing on the first, so clients can fix everything in one round trip.

23. **Messages are for humans, codes are for machines.** Keep messages clear and non-technical; localize on the client where needed.

### Client errors (4xx)

24. Map these 4xx codes consistently to the resource and operation involved — these signal that the caller did something wrong.

| Code | Meaning | When |
|------|---------|------|
| `400 Bad Request` | Malformed syntax / invalid input | Validation failures |
| `401 Unauthorized` | Missing/invalid authentication | No or bad credentials |
| `403 Forbidden` | Authenticated but not permitted | Authorization failure |
| `404 Not Found` | Resource doesn't exist | Unknown ID (also use to hide existence) |
| `405 Method Not Allowed` | Method not supported on resource | `DELETE /reports` when not allowed |
| `409 Conflict` | State conflict | Duplicate, version conflict, concurrent edit |
| `422 Unprocessable Entity` | Syntactically valid but semantically wrong | Business-rule validation failures |
| `429 Too Many Requests` | Rate limit exceeded | Throttling; include `Retry-After` |

### Server errors (5xx)

25. Handle these key server-side failures. The caller did nothing wrong — something on our side (or a dependency) failed.

| Code | Meaning | When |
|------|---------|------|
| `500 Internal Server Error` | Unexpected failure | Unhandled exceptions (never leak stack traces) |
| `502 Bad Gateway` | Upstream returned invalid response | Proxy/gateway failures |
| `503 Service Unavailable` | Temporarily down/overloaded | Maintenance; include `Retry-After` |
| `504 Gateway Timeout` | Upstream timed out | Dependency timeouts |

---

## 3. Versioning

### Strategy: URI path versioning

26. **Put the major version in the URL path.** It's explicit, cache-friendly, and easy to route.

```
/v1/users
/v2/users
```

27. **Only the major version appears in the URL.** Minor and patch changes must be backward compatible and shipped without a version bump.

### What counts as a breaking change?

28. **Bump the major version only for breaking changes — ship everything else in place.** Classify every change before you release it.

**Breaking (requires a new major version):**
- Removing or renaming a field, endpoint, or parameter
- Changing a field's type or semantics
- Adding a new required request parameter
- Changing the meaning of a status code or error code
- Tightening validation that previously passed

**Non-breaking (safe within the current version):**
- Adding a new endpoint
- Adding a new **optional** request parameter
- Adding a new field to a response (clients must ignore unknown fields)
- Adding a new value to an enum *only if* clients are documented to tolerate unknown values

> **Golden rule for consumers:** clients must ignore fields they don't recognize. Document this expectation so additive changes stay non-breaking.

---

## 4. Security

Security is a baseline requirement, not a feature — every endpoint must clear the bar below before it ships. For a fuller industry checklist, anchor reviews to the **OWASP API Security Top 10**, whose number-one risk (Broken Object Level Authorization) is precisely a failure of the per-object checks described under *Authorization* below.

### Transport & headers

29. **HTTPS everywhere — no exceptions.** Reject plain HTTP, require TLS 1.2+ (prefer 1.3), and send `Strict-Transport-Security` (HSTS). Apply this to internal service-to-service traffic too, wherever feasible.

30. **Set defensive response headers.** At minimum `X-Content-Type-Options: nosniff`, plus `Content-Security-Policy` and HSTS where relevant. Don't advertise implementation details via `Server` / `X-Powered-By`.

31. **Lock down CORS.** Configure an explicit origin allowlist; never reflect arbitrary `Origin` values, and never combine `Access-Control-Allow-Origin: *` with credentialed requests.

### Authentication

32. **Use a standard authentication scheme.** OAuth 2.0 / OpenID Connect bearer tokens for user- and service-facing APIs; scoped, revocable API keys for server-to-server integrations. Don't invent your own auth.

33. **Pass credentials only in the `Authorization` header — never in URLs or query strings.** Query-string credentials leak into access logs, browser history, and `Referer` headers.

34. **Keep tokens short-lived and verifiable.** Use short expiry plus a refresh mechanism, and validate signature, issuer, audience, and expiry on every request. Support revocation so a leaked token can be killed.

### Authorization

35. **Authentication is not authorization — enforce permissions on every request, server-side.** Knowing *who* the caller is doesn't tell you *what* they may do. Never rely on the client to enforce access.

36. **Check object-level authorization on every resource access (prevent BOLA / IDOR).** Verify the authenticated caller is actually allowed to act on *this specific* resource ID, not merely that they are logged in. This is the most common and most damaging API vulnerability. Non-enumerable IDs (rule 42) raise the bar but are **defense-in-depth, not a substitute** for the ownership check.

37. **Grant least privilege.** Scope tokens and API keys to the minimum operations and data they need — a read-only integration should never hold write or admin scope.

### Input handling

38. **Validate and sanitize all input** — body, query params, path params, and headers. Check type, range, length, format, and allowed values, and reject unexpected fields rather than silently ignoring them.

39. **Enforce size limits on everything** — maximum body size, array length, string length, and nesting depth — to prevent resource-exhaustion (DoS) attacks.

40. **Prevent injection** (SQL, NoSQL, OS command, LDAP, etc.) with parameterized queries and safe, well-maintained libraries. Never build queries or commands by string concatenation.

41. **Guard against mass assignment.** Explicitly allowlist which fields a request may set; never bind a request body directly onto your internal or ORM model. (See also rule 8 — model the domain, not the client.)

### Data protection & privacy

42. **Avoid enumerable IDs for sensitive resources.** Prefer UUIDs or opaque identifiers over sequential integers so resources can't be discovered by guessing. (Defense-in-depth for rule 36, not a replacement for it.)

43. **Don't leak data in errors.** Return generic messages to callers; keep stack traces, SQL, and internal hostnames in server-side logs only. (Reinforces rule 21.)

44. **Encrypt sensitive data in transit and at rest, and keep it out of URLs.** No tokens, PII, or secrets in query strings, path segments, or cached responses.

45. **Manage secrets properly.** Store credentials and keys in a secrets manager / vault — never in source code or committed config — and rotate them regularly and on suspected compromise.

46. **Never log secrets, tokens, passwords, or PII in plaintext.** Mask or omit them in both application and access logs.

### Abuse prevention & monitoring

47. **Rate-limit and throttle** to blunt brute-force, credential-stuffing, scraping, and denial-of-service attempts. Return `429` with a `Retry-After` header (see rule 20).

48. **Audit-log sensitive actions** — authentication, permission changes, data exports — capturing who did what, when, and from where, and protect those logs from tampering.

---

## 5. Performance

Efficient data access keeps an API fast and cheap to run at scale. The theme across this section: never make a client over-fetch, and never make it ask for more than it needs.

### Pagination, Filtering, Sorting & Searching

#### Pagination

49. **Paginate every collection — never return an unbounded list.** Always enforce a sensible default and a hard maximum page size (e.g. default 25, max 100) so a single request can't pull an entire table.

50. **Prefer cursor-based (keyset) pagination for large or fast-changing datasets.** It's stable under inserts and deletes and performs well at scale. Offset pagination (`?page=2&pageSize=50`) is fine for small, stable datasets and admin tools, but it degrades on deep pages and can skip or duplicate rows when the underlying data shifts between requests.

#### Filtering

51. **Filter with query parameters keyed by field name, and pick one operator syntax.** Choose a single convention for ranges and operators, then apply it everywhere:

```
GET /orders?status=PAID&createdAfter=2026-01-01
GET /products?price[gte]=10&price[lte]=100      # bracket style
GET /products?filter=price>=10,price<=100       # expression style — pick ONE
```

#### Sorting

52. **Support sorting through a `sort` parameter with an explicit direction convention.** A leading `-` for descending is a common, readable choice:

```
GET /users?sort=-createdAt,lastName    # createdAt descending, then lastName ascending
```

#### Field selection (sparse fieldsets)

53. **Let clients request only the fields they need.** Sparse fieldsets cut payload size and serialization cost for clients that don't need the full resource:

```
GET /users/42?fields=id,email,createdAt
```

#### Searching

54. **Provide a dedicated search parameter or endpoint for full-text or complex queries.** Keep it separate from exact-match filtering:

```
GET /products?q=wireless+headphones
GET /search?q=...&type=product
```

### Response efficiency

#### Compression

55. **Enable HTTP compression for text-based responses.** Negotiate it properly — honor the client's `Accept-Encoding` and set `Content-Encoding` on the response. Use `gzip` for broad compatibility and `br` (Brotli) where the client supports it; JSON typically compresses well (often 60–80% smaller), making this one of the cheapest latency wins available.
    - Apply a **minimum-size threshold** (e.g. ~1 KB) — compressing tiny responses costs more CPU than it saves.
    - **Don't recompress already-compressed content** (images, video, PDFs, zip archives); it burns CPU for no gain.
    - It's fine to terminate compression at the gateway, CDN, or reverse proxy rather than in the app — just ensure it's enabled and consistent.
    - **Security caveat:** compressing a response that mixes secret data with attacker-influenced input over TLS can enable BREACH-style attacks. For responses carrying sensitive tokens or secrets, disable compression or apply known mitigations rather than compressing blindly.

### Reducing round-trips

56. **Design against chatty access patterns.** When clients predictably need related data, let them fetch it in one request — via opt-in resource expansion (`GET /orders/1001?expand=customer,lineItems`) or a batch endpoint — rather than forcing an N+1 storm of follow-up calls. Balance this against payload size: expansion should be opt-in, never the default.

---

## 6. Idempotency

Idempotency lets a client safely retry a request without causing duplicate side effects — essential for payments, orders, and any state-changing call made over an unreliable network, where a response can be lost even though the server already acted.

57. **Know which methods are idempotent by definition.** `GET`, `PUT`, and `DELETE` are idempotent per the HTTP spec — repeating them yields the same end state. `POST` is not, and `PATCH` generally isn't. Design accordingly: a retried `PUT` is safe, but a retried `POST` can double-charge.

58. **Make unsafe creates retryable with an idempotency key.** Let the client send a unique, client-generated `Idempotency-Key` on `POST`. The server records the key alongside the result of the first successful request and, on any retry with the same key, returns that original result instead of performing the operation again. (This is the mechanism rule 20 refers to.)

```http
POST /payments
Idempotency-Key: 7f3a9b2c-1e4d-4a8f-9c2b-5e1f8d3a7b6c
Content-Type: application/json

{ "amount": 5000, "currency": "USD" }
```

59. **Specify the idempotency contract explicitly.** Document the **retention window** for stored keys, and define the edge cases: the same key sent with a *different* request body should be rejected (`409 Conflict` or `422`), and a retry that arrives while the original is still in flight must not start a second operation. Scope keys per endpoint and per caller so they can't collide.

---

## 7. Asynchronous & Long-Running Operations

When an operation can't reliably finish within a normal request window — roughly a few seconds, e.g. report generation, bulk imports, media processing — don't hold the connection open. Accept the work, process it in the background, and give the client a way to track it.

60. **Acknowledge the request with `202 Accepted` and a tracking handle.** Return immediately with a pointer to an operation resource (via the `Location` header) that the client can poll.

```http
POST /reports
→ 202 Accepted
  Location: /operations/op_8821
  { "operationId": "op_8821", "status": "PENDING" }
```

61. **Provide a status endpoint with a defined lifecycle.** Expose the operation's state through a stable set of statuses (e.g. `PENDING | RUNNING | SUCCEEDED | FAILED`, following the `UPPER_SNAKE_CASE` enum convention in rule 14), and include progress where you can. Tell clients how often to poll — or send `Retry-After` — so they don't hammer the endpoint.

```http
GET /operations/op_8821
→ 200 OK
  {
    "operationId": "op_8821",
    "status": "RUNNING",     // PENDING | RUNNING | SUCCEEDED | FAILED
    "progress": 0.6,
    "result": null
  }
```

62. **Expose the outcome on completion.** On success, return or link to the resource that was created; on failure, return the error using the standard Problem Details shape (rule 16), so async failures look like every other error in the system.

63. **Offer webhooks/push as an alternative to polling.** Where clients can receive callbacks, push completion events rather than making them poll. Secure webhooks by signing the payload so receivers can verify its origin, retrying delivery with backoff on failure, and making delivery idempotent on the receiver's side.

---

## 8. Know Your Consumers

Design every API from the consumer's point of view. Crucially, many of your consumers aren't people clicking through a UI — they're other systems that integrate once and then call you unattended, at scale, indefinitely. A machine can't "figure it out" at runtime the way a person can, so clarity, predictability, and stability matter even more than they would for a human-facing product.

64. **Make APIs self-explanatory.** Resource paths, query parameters, and request/response shapes should make an endpoint's behavior obvious on sight — a consumer should be able to guess what it does before opening the docs. Documentation augments good design; it never rescues bad design. If it takes a paragraph of prose to explain what `POST /x` does, the design is probably the problem.

65. **Right-size each API — neither overloaded nor underloaded.** Avoid the god-endpoint that tries to do everything (hard to learn, hard to evolve, easy to break), but equally avoid splintering one capability into dozens of near-identical endpoints. Aim for generic, reusable resources that compose, so a small, coherent set of endpoints covers many use cases.

66. **Consistency beats cleverness.** A predictable API that follows boring, familiar conventions beats a novel one every time. Settle on one pattern for naming, pagination, filtering, and errors, then apply it across every endpoint so a consumer learns it once and reuses that knowledge everywhere. When in doubt, do what the rest of our APIs already do.

67. **Treat backward compatibility as sacred.** Never break an existing consumer without a clear versioning and deprecation path — additive changes are safe; removals and semantic changes are not (see Section 3). A consumer you break silently is a consumer who stops trusting the platform, and that trust is far harder to win back than it was to lose.

68. **Stay stateless.** Each request must carry everything needed to process it; never rely on server-side session state carried implicitly between calls. Statelessness is what lets requests be routed, retried, load-balanced, and scaled freely — exactly the properties automated consumers depend on.

69. **Make the common case easy.** Provide sensible defaults so a consumer can make a correct, useful call with minimal required input, reaching for advanced parameters only when they actually need them. The simplest valid request should be short — don't force every caller to pay the cost of your most complex use case.

---

## 9. Documentation

An undocumented API is an unusable API — especially for the machine consumers described in Section 8, whose integrators rely entirely on the written contract.

70. **Maintain an OpenAPI (Swagger) specification as the single source of truth.** Either generate it from code, or treat the spec as a contract the code must satisfy (contract-first). Because it's machine-readable, one well-kept spec also yields generated client SDKs, mock servers, and contract tests — so it earns its keep well beyond human reading.

71. **Document every endpoint completely.** For each one: its purpose, every parameter, the request and response schemas, all possible status codes, the error codes it can return (the codes consumers branch on, per rule 18), and its authentication and authorization requirements.

72. **Provide realistic examples for requests and responses — including error responses.** Examples are what consumers actually copy. A worked example of each failure is as valuable as the success case, since handling errors is where most integrations break.

73. **Version docs alongside the API and update them in the same pull request as the code.** Documentation that drifts from behavior is worse than none, because it actively misleads. If a change ships without its doc update, the change isn't done.

74. **Publish a changelog.** Give consumers one place to track what changed between versions — new endpoints, additive fields, and especially anything deprecated or scheduled for sunset — so they can plan upgrades instead of discovering breakage in production.

---

## 10. Deprecation Policy

[ ] TODO

---

## 11. Observability

[ ] TODO
