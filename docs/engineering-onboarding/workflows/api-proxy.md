# Workflow: API Proxy to Remote App Backends

Revision: `main` @ `abe5457` (synced from `5ff58a7` on 2026-10-16; covers #212). Batch 5 (2026-10-09).

Files in scope (read in full): `server/internal/handler/proxy.go`, `server/pkg/require/require.go`. Tests: `server/internal/handler/proxy_test.go` case names reviewed and the JWT test bodies read (`proxy_test.go:354-533`).

## 1. Purpose

Remote apps (MFEs) run inside the host shell on the TW origin, so their browser requests carry the TW session cookie. They call their own backends through TW at `/api/<app>/...`. TW requires a signed-in session user with a role, strips cookies, attaches a short-lived HS256 JWT signed with a per-app shared key and carrying the user's base role and attributes, and reverse-proxies to the app's backend. This is hop 3 to 4 of ADR-0001.

## 2. Request flow

```mermaid
sequenceDiagram
    autonumber
    participant MFE as Remote app (in browser)
    participant TW as TW server
    participant P as proxy() handler
    participant RP as ReverseProxy (per app)
    participant BE as Remote backend

    MFE->>TW: GET /api/posts/v1/announcements?page=2 (Cookie tw_session)
    Note over TW: RequestID, RequestLog, Session (load and later re-save session)
    TW->>P: request
    P->>P: name = first segment after /api/ ("posts")
    alt unknown or unregistered app, or no slash after the name
        P-->>MFE: 404 JSON {"message":"Not Found"}
    else no session, no signed-in user, or user without role or attributes
        P-->>MFE: 500 JSON {"message":"Internal Server Error"} (logged)
    else registered and signed in
        P->>P: sign HS256 JWT {iss: tw, aud: [pg], iat, exp = iat + TTL, role, effective_role, attributes}
        P->>RP: StripPrefix("/api/posts") so path is /v1/announcements
        RP->>RP: SetURL(base URL): scheme, host, base path + /v1/announcements, query kept
        RP->>RP: delete Cookie, set Authorization Bearer JWT
        RP->>BE: GET {base}/v1/announcements?page=2
        alt backend unreachable or transport error
            RP-->>MFE: 502 JSON {"message":"Bad Gateway"} (logged without query string)
        else client disconnected
            RP-->>MFE: nothing (not logged)
        else response
            BE-->>RP: status, headers, body
            RP->>RP: delete Set-Cookie
            RP-->>MFE: backend status and body, plus TW's own session Set-Cookie and Cache-Control no-store
        end
    end
```

| Step | Status | Evidence |
| --- | --- | --- |
| Session layer runs for `/api/` and re-saves the session; since #212 the proxy reads the session user | Verified | `handler.go:97, 102`, `proxy.go:60-86` |
| App name from the first path segment; `/api/posts` (no trailing slash) is 404. Routing runs **before** the session check, so an unknown app is 404 even when signed out | Verified | `proxy.go:51-58`; test "missing path after app" `proxy_test.go:151` |
| No session, no user, empty `Role` or nil `Attributes`: 500 JSON, logged | Verified | `proxy.go:60-86`; tests `proxy_test.go:194-352` |
| JWT claims `iss`, `aud`, `iat`, `exp` plus `role`, `effective_role` (copy of `role`), `attributes` | Verified | `proxy.go:88-107`; tests `proxy_test.go:354-445` (registered claims), `446-533` (role and attributes) |
| Prefix stripped, base URL path prepended, escaped segments preserved | Verified | `proxy.go:118, 125`; tests `proxy_test.go:41-118, 559` |
| Cookie removed, `Authorization: Bearer` set (replacing any client-sent value) | Verified | `proxy.go:127-130`; tests `proxy_test.go:575, 593` |
| Backend `Set-Cookie` removed | Verified | `proxy.go:132-135`; test `proxy_test.go:612` |
| 502 JSON on transport error, query string kept out of logs; silent on client cancel | Verified | `proxy.go:136-158`; tests `proxy_test.go:635-751` |

## 3. Path mapping

| Registered app | Browser path prefix | JWT `aud` | Backend base URL setting | MF remote name |
| --- | --- | --- | --- | --- |
| Posts (Parents Gateway) | `/api/posts/` | `pg` | `TW_REMOTE_POSTS_BACKEND_BASE_URL` | `pg` (exposes `Posts`, `Groups`) |
| Student Insights | `/api/student-insights/` | `si` | `TW_REMOTE_STUDENT_INSIGHTS_BACKEND_BASE_URL` | `si` |

Examples, with base URL `https://pg-api.internal/base`:

| Browser request                   | Forwarded to                                        |
| --------------------------------- | --------------------------------------------------- |
| `/api/posts/`                     | `https://pg-api.internal/base/`                     |
| `/api/posts/v1/groups/P5%2F3?x=1` | `https://pg-api.internal/base/v1/groups/P5%2F3?x=1` |
| `/api/posts`                      | 404 at TW                                           |
| `/api/unknown/x`                  | 404 at TW                                           |

The API prefix (`posts`, `student-insights`), the audience (`pg`, `si`) and the remote name (`pg`, `si`) are three different identifiers for the same app, all hard-coded (`proxy.go:28-41`, `index.go:27-31`). Groups API calls presumably also go under `/api/posts/` since the `pg` remote owns both (Inferred).

## 4. Contract for remote backend teams

What a remote backend receives (Verified from code and tests):

| Item | Value |
| --- | --- |
| Method, path suffix, query, body | as sent by the browser, minus the `/api/<app>` prefix |
| `Authorization` | `Bearer <JWT>`, always set by TW |
| `Cookie` | never (removed) |
| JWT header | `alg: HS256`, `typ: JWT` |
| JWT registered claims | `iss: "tw"`, `aud: ["pg"]` or `["si"]`, `iat`, `exp` = `iat` + `TW_REMOTE_SIGNED_TOKEN_TTL` (default 1 minute) |
| JWT custom claims | `role`: the user's one Edupass base role, e.g. `"ROLE_TEACHER"`; `effective_role`: same value, kept for PG until it reads `role` (comment `proxy.go:93`); `attributes`: array of attribute codes, e.g. `["ATTR_PG_ADMIN"]`, `[]` when none |
| Signing key | the app's `TW_REMOTE_..._BACKEND_SIGNING_KEY` (at least 32 bytes, shared secret) |
| User identity | role and attributes only: **no** `sub`, Edupass ID, name, email or school (location is stripped at sign-in, `edupass-sign-in.md` section 7.1) |
| Client IP / forwarding headers | not set by TW (`SetXForwarded` is not called). With a `Rewrite` hook, the stdlib also drops inbound `X-Forwarded-*` (Inferred, `net/http/httputil` behaviour) |
| Request ID | TW's own ID is not forwarded (no header set in `Rewrite`); a client-supplied `X-Request-ID` passes through unchanged (Verified in batch 6, `subsystems/observability.md` section 5) |

Example decoded payload for mock fixture `staff-1` calling Posts (shape Verified from `proxy.go:89-105`; values from the fixture):

```json
{
  "iss": "tw",
  "aud": ["pg"],
  "iat": 1760600000,
  "exp": 1760600060,
  "role": "ROLE_TEACHER",
  "effective_role": "ROLE_TEACHER",
  "attributes": ["ATTR_PG_ADMIN"]
}
```

What a backend should verify (recommendation, mirroring the test at `proxy_test.go:410-440`): signature with the shared key, algorithm restricted to HS256, `iss == "tw"`, `aud` contains its own code, and `exp` not passed (allowing small clock skew). Authorise by `role` and `attributes`.

What the backend can send back: any status, headers and body; TW passes them through except `Set-Cookie` (removed) and hop-by-hop headers. TW adds its own session `Set-Cookie` and `Cache-Control: no-store` to every `/api/` response, overriding any caching header the backend sets (`middleware/session.go:102`).

## 5. Failure and timeout behaviour

| Situation | Result | Evidence |
| --- | --- | --- |
| Unknown app, unregistered app, missing path | 404 JSON | `proxy.go:53-58` |
| No session in context, signed-out session, user without `Role` or with nil `Attributes` | 500 JSON, logged ("no session found in context", "no user found in session", "no role or attributes found in user") | `proxy.go:60-86` |
| JWT signing error | 500 JSON, logged with app name | `proxy.go:108-115` |
| Backend unreachable, connection reset, TLS error | 502 JSON, logged with method, path (no query) and backend URL (no query) | `proxy.go:136-158` |
| Client disconnects | nothing written, nothing logged | `proxy.go:137-141` |
| Backend slow | no per-backend timeout: the proxy uses the default transport, so the server's `WriteTimeout` (30s default) cuts the response | Verified absence (`proxy.go:122-160` sets no `Transport`); cut-off behaviour Inferred from `net/http` |
| Large or slow uploads | the server's `ReadTimeout` (15s default) applies to the request body | Inferred from `net/http` and `config.go:52-54` |
| Session store failure | 500 before the proxy runs (session layer) | `middleware/session.go:65-80` |
| Backend 4xx/5xx | passed through unchanged | Verified (no `ModifyResponse` status handling) |

Two consequences worth knowing (Inferred from the code above, not covered by a dedicated test):

- A signed-out caller gets **500**, not 401, so the host cannot tell "please sign in" from a server fault by status.
- A session saved before #199 holds `User{Email}` only; it decodes with an empty `Role`, so `/api/` returns 500 for that user until they sign in again or the session idles out (30m `AuthenticatedTTL`, sliding, no absolute timeout; `subsystems/sessions.md`).

## 6. Gap against ADR-0001 (Q21)

ADR-0001 hop 3: the host backend "authenticates the session, strips the cookie, and signs a short-lived JWT scoped to that one app".

| ADR element | Implemented? |
| --- | --- |
| Strips the cookie | yes (`proxy.go:127`) |
| Short-lived JWT scoped to one app | yes (`aud`, 1m default TTL) |
| Authenticates the session | **yes since #212**: no JWT is issued without a signed-in user with a role (`proxy.go:60-86`); the refusal is a 500, not a 401 |
| Carries who the user is | **partly**: role and attributes, but no subject (`sub`) or other identity |
| CSRF on state-changing calls | **no** (Q8) |

Impact: anonymous callers can no longer obtain a signed token, and backends can authorise by role. Backends still cannot tell two users with the same role apart, which matters for per-user data (an outstanding follow-up to add a subject claim).

## 7. `pkg/require`

Test assertion helpers only (`Equal`, `NotEqual`, `True`, `False`, `NoError`, `HasError`, all `t.Fatalf` on failure), used by e.g. `pkg/dotenv/dotenv_test.go`. It imports `testing` and is not used by the server binary. It sits in `pkg/` (exported) although it is test-only (`require.go:1-63`).

## 8. Where to change things

| Change | Where |
| --- | --- |
| Return 401 instead of 500 for signed-out callers | the "no user found in session" branch `proxy.go:69-77`; update the 500 cases in `proxy_test.go:194-352` |
| Add a subject (`sub`) or more identity to the JWT | the anonymous claims struct `proxy.go:89-105`; data comes from `session.User` (`session.go:59-65`); extend `proxy_test.go:446-533` |
| Drop `effective_role` once PG reads `role` | `proxy.go:93-94, 103` and its test assertion `proxy_test.go:524` |
| Enforce CSRF on unsafe methods | before signing, using `sess.VerifyCSRFToken(r.Header.Get(...))`; host must send `preloadedState.csrfToken` |
| Forward request ID or client IP | `Rewrite` in `newRemoteBackendProxy` (`proxy.go:124-131`): `pr.SetXForwarded()`, set `X-Request-Id` |
| Per-backend timeout | give the `ReverseProxy` a `Transport` with `ResponseHeaderTimeout` |
| New remote app | config, `proxy.go:28-41`, `index.go:27-31`, host routes (see Q20) |

Breakpoints: `proxy.go:51` (routing), `proxy.go:69` (session user), `proxy.go:108` (signing), `proxy.go:127` (outbound headers), `proxy.go:148` (proxy errors).
