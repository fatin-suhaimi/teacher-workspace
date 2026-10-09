# 04 API and Interface Catalog

Revision: `main` @ `5ff58a7`. Started in batch 3 (2026-10-09). Rows marked "partial" or "not traced" are filled in by later batches.

## Teacher Workspace server (Go)

All routes except `/static/` pass through RequestID, RequestLog and the Session middleware (`main.go:107-111`, `handler.go:93-105`), so every response from them carries `Set-Cookie: tw_session=...` and `Cache-Control: no-store`, and a session store failure turns any of them into a 500.

| Route | Handler | Auth required | Input | Success | Failure | Side effects | Tests | Flow doc |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `GET /auth/edupass` | `Handler.authEdupass` (`auth.go:38-64`) | no | query `return_to` (optional, sanitised by `safeReturnTo`) | 302 to Edupass authorize URL | 500 plain if no session in context | session data: return_to, state, nonce, code_verifier (replaces any pending login) | `auth_test.go:26-377` | `workflows/edupass-sign-in.md` |
| `GET /auth/edupass/callback` | `Handler.authEdupassCallback` (`auth.go:90-253`) | no (needs the pending login in the session) | query `code`, `state`, or `error` / `error_description` | 302 to stored `return_to` (default `/`) | 302 `/login?error=oauth2_callback_failed&return_to=...`; 500 plain if no session | token request to Edupass; JWKS fetch; `SetUser` rotates the session ID; old session entry dropped | `auth_test.go:378-1395`, `TestSafeReturnTo` | `workflows/edupass-sign-in.md` |
| `/api/{posts,student-insights}/*` (any method) | `Handler.proxy()` (`proxy.go:25-78`) | **no** (Q21) | path after `/api/<name>`; body passed through | backend's response | 404 JSON `{message}` for unknown or unregistered remote; 500 JSON if JWT signing fails | HS256 JWT (`iss=tw`, `aud=pg` or `si`, `iat`, `exp` = now + `TW_REMOTE_SIGNED_TOKEN_TTL`) attached by the backend proxy (header not yet traced) | `proxy_test.go` (not read) | partial, batch 5 |
| `/` (catch-all, any method) | `Handler.index()` (`index.go`) | not traced | not traced | HTML page with preloaded state (`csrfToken`, `remotes`) | not traced | mints a CSRF token (`index.go:44`) | `index_test.go` (not read) | batch 4 |
| `/static/*` | `Handler.static` | no session | path | asset (prod) or dev-server proxy (dev) | 404 / 502 in dev (`handler.go:107-130`) | none | not read | batch 4 |

Not present (Verified absent from the route table): health or readiness endpoint, logout, user-info or "who am I" endpoint, CSRF-protected mutation endpoints.

## mock-edupass (dev and test only)

| Route | Handler | Notes | Tests |
| --- | --- | --- | --- |
| `GET /health` | inline (`app.ts:60-62`) | 200 | not listed |
| `GET /.well-known/openid-configuration` | `oidc-provider` | issuer = `MOCK_EDUPASS_URL`; query response mode; S256 PKCE | `api.test.ts:20-120` |
| `GET /jwks` | `oidc-provider` | new RSA-2048 key per boot | `api.test.ts:122-188` |
| `GET /authorize` | `oidc-provider` (route renamed, `provider.ts:124-126`) | PKCE required; extra param `account` selects the fixture | `api.test.ts:189-332` |
| `GET /interaction/:uid` | `createApp` router (`app.ts:15-56`) | auto login (account param or staff-1) and auto consent (`openid`) | via `/authorize` tests |
| `POST /token` | `oidc-provider` | `client_secret_post` or `private_key_jwt` (PS256 plus `x5t#S256` check) | `api.test.ts:333-1400` |
| other `oidc-provider` defaults (userinfo, etc.) | `oidc-provider` | not used by TW | not reviewed |

## Coverage map

| Interface | Traced | Sequence diagram | Error paths |
| --- | --- | --- | --- |
| `GET /auth/edupass` | yes | yes (`workflows/edupass-sign-in.md` section 3) | yes |
| `GET /auth/edupass/callback` | yes | yes (sections 3 and 5) | yes |
| `/api/*` | partial | no | partial |
| `/` | no | no | no |
| `/static/*` | partial (dev proxy only) | no | partial |
