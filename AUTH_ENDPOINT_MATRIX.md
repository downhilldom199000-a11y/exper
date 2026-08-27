# TKREC Authentication Endpoint Matrix

> Generated: 2026-08-26
> 88 endpoints mapped with authentication requirements

---

## 1. Authentication Distribution Summary

| Auth Type | Count | Percentage | Description |
|-----------|-------|------------|-------------|
| **None** | 15 | 17.0% | Public endpoints, no credentials required |
| **JWT** | 73 | 83.0% | Bearer token in Authorization header |
| **CSRF** | 0 | 0.0% | XSRF-TOKEN cookie only (unused standalone) |
| **JWT+CSRF** | 0 | 0.0% | Both required (unused — all JWT endpoints rely on withCredentials) |
| **OAuth** | 0 | 0.0% | Social login token exchange (handled externally) |
| **Total** | **88** | **100%** | |

### Axios Instance Usage

| Instance | Base URL | Auth Method | withCredentials | withXSRFToken | Usage |
|----------|----------|-------------|-----------------|---------------|-------|
| `u` (main) | raid.arkzynco.com | Bearer token injection | true | true | All JWT endpoints |
| `Bg` (public) | ws.arkzynco.com | None | false | false | Public endpoints only |
| `xe` (secondary) | ws.arkzynco.com | Bearer token injection | — | — | Legacy/alternate JWT endpoints |

---

## 2. Complete Endpoint Matrix

### No Auth Required (15 endpoints)

| # | Method | Endpoint | Auth Type | CSRF | Cookies | Notes |
|---|--------|----------|-----------|------|---------|-------|
| 1 | POST | /api/signin | None | No | — | Authentication entry point |
| 2 | POST | /api/login | None | No | — | Alternative login |
| 3 | POST | /api/register | None | No | — | New account creation |
| 4 | POST | /api/login-device | None | No | — | Device-specific login |
| 5 | POST | /api/confirm-email | None | No | — | Email verification |
| 6 | POST | /api/forgot-password | None | No | — | Password reset request |
| 7 | POST | /api/reset-password/{token} | None | No | — | Password reset via token |
| 8 | GET | /api/csrf | None | No | XSRF-TOKEN | Sets XSRF-TOKEN cookie |
| 70 | POST | ws.arkzynco.com/api/public/history/fetch | None | No | — | Public history lookup |
| 71 | POST | /api/public/stream-count | None | No | — | Returns 404 (deprecated) |
| 79 | GET | /api/slides | None | No | — | Public slide content |
| 80 | GET | /api/pages/{pageUrl} | None | No | — | Public page content |
| 81 | GET | /api/pages/rewards-overview | None | No | — | Rewards overview page |
| 83 | POST | ws.arkzynco.com/api/profile/login | None | No | — | Legacy login endpoint |
| 84 | POST | ws.arkzynco.com/api/profile/code | None | No | — | Legacy code verification |
| 85 | GET | /api/pages/faq | None | No | — | FAQ page content |

### JWT Auth Required (73 endpoints)

| # | Method | Endpoint | Auth Type | CSRF | Cookies | Notes |
|---|--------|----------|-----------|------|---------|-------|
| 9 | POST | /api/auth/fcm | JWT | Yes | XSRF | Firebase Cloud Messaging registration |
| 10 | POST | /api/status | JWT | Yes | XSRF | Status check; uses pl() helper |
| 11 | GET | /api/v2/user | JWT | Yes | XSRF | Get current user |
| 12 | PUT | /api/v2/user | JWT | Yes | XSRF | Update current user |
| 13 | DELETE | /api/v2/user | JWT | Yes | XSRF | Delete account |
| 14 | POST | /api/v2/user/installed | JWT | Yes | XSRF | Record app installation |
| 15 | POST | /api/v2/user/transfer-recordings/{accountId}/{targetId} | JWT | Yes | XSRF | Transfer recordings between accounts |
| 16 | POST | /api/v2/user/profile | JWT | Yes | XSRF | Create user profile |
| 17 | POST | /api/v2/user/profile/{id} | JWT | Yes | XSRF | Update user profile |
| 18 | DELETE | /api/v2/user/profile/{id} | JWT | Yes | XSRF | Delete user profile |
| 19 | PUT | /api/v2/user/profile/{id}/logout | JWT | Yes | XSRF | Profile-level logout |
| 20 | POST | /api/v2/user/profile/{id}/mute | JWT | Yes | XSRF | Mute profile |
| 21 | GET | /api/v2/followers/{accountId}/list | JWT | Yes | XSRF | List all followers |
| 22 | POST | /api/v2/followers/{accountId} | JWT | Yes | XSRF | Add follower |
| 23 | DELETE | /api/v2/followers/{accountId}/{id} | JWT | Yes | XSRF | Remove follower |
| 24 | PUT | /api/v2/followers/{accountId}/{id} | JWT | Yes | XSRF | Update follower |
| 25 | GET | /api/v2/followers/{accountId}/{pid} | JWT | Yes | XSRF | Get follower detail |
| 26 | GET | /api/v2/followers/{accountId}/{pid}/history | JWT | Yes | XSRF | Follower stream history |
| 27 | GET | /api/v2/followers/{accountId}/{pid}/rm | JWT | Yes | XSRF | Remove follower (GET variant) |
| 28 | POST | /api/v2/followers/{accountId}/{pid}/fetch | JWT | Yes | XSRF | Fetch follower data |
| 29 | POST | /api/v2/followers/{accountId}/{pid}/validate | JWT | Yes | XSRF | Validate follower access |
| 30 | POST | /api/v2/followers/{accountId}/{pid}/deep-check | JWT | Yes | XSRF | Deep validation check |
| 31 | POST | /api/v2/followers/{accountId}/{pid}/report | JWT | Yes | XSRF | Report follower |
| 32 | POST | /api/v2/followers/{accountId}/{pid}/following/fetch | JWT | Yes | XSRF | Fetch follower's following list |
| 33 | GET | /api/v2/followers/{accountId}/{uid}/following | JWT | Yes | XSRF | Get following status |
| 34 | GET | /api/v2/followers/{accountId}/{uid}/deep-check | JWT | Yes | XSRF | Deep check (GET variant) |
| 35 | GET | /api/v2/followers/discover/{accountId} | JWT | Yes | XSRF | Discover new followers |
| 36 | GET | /api/v2/followers/discover/{accountId}/bubbles | JWT | Yes | XSRF | Discover bubbles |
| 37 | GET | /api/v2/followers/explore/lives/{accountId} | JWT | Yes | XSRF | Explore live streams |
| 38 | GET | /api/v2/followers/ranked/{accountId} | JWT | Yes | XSRF | Ranked followers |
| 39 | GET | /api/v2/followers/search/{accountId} | JWT | Yes | XSRF | Search followers |
| 40 | GET | /api/v2/followers/similar/{accountId}/{pid} | JWT | Yes | XSRF | Similar followers |
| 41 | GET | /api/v2/followers/suggest/{accountId} | JWT | Yes | XSRF | Suggested followers |
| 42 | POST | /api/v2/followers/{accountId}/{pid}/like | JWT | Yes | XSRF | Like follower |
| 43 | GET | /api/v2/followers/{accountId}/likes | JWT | Yes | XSRF | Get follower likes |
| 44 | POST | /api/v2/followers/hide/{accountId}/{pid} | JWT | Yes | XSRF | Hide follower |
| 45 | POST | /api/v2/followers/{accountId}/report | JWT | Yes | XSRF | Report follower (account-level) |
| 46 | GET | /api/v2/records/{accountId} | JWT | Yes | XSRF | List recordings |
| 47 | GET | /api/v2/records/{accountId}/{recordId} | JWT | Yes | XSRF | Get recording detail |
| 48 | POST | /api/v2/records/new/{accountId} | JWT | Yes | XSRF | Create new recording |
| 49 | DELETE | /api/v2/records/{accountId}/{recordId} | JWT | Yes | XSRF | Delete recording |
| 50 | DELETE | /api/v2/records/{accountId}/all | JWT | Yes | XSRF | Delete all recordings |
| 51 | DELETE | /api/v2/records/follower/{accountId}/{pid} | JWT | Yes | XSRF | Delete recordings by follower |
| 52 | POST | /api/v2/records/{accountId}/delete-selected | JWT | Yes | XSRF | Delete selected recordings |
| 53 | POST | /api/v2/records/{accountId}/{recordId}/attach | JWT | Yes | XSRF | Attach recording |
| 54 | PUT | /api/v2/records/{accountId}/{recordId}/fix | JWT | Yes | XSRF | Fix recording metadata |
| 55 | POST | /api/v2/records/{accountId}/{recordId}/ups | JWT | Yes | XSRF | Upscale recording |
| 56 | POST | /api/v2/records/{accountId}/{recordId}/downloaded | JWT | Yes | XSRF | Mark recording as downloaded |
| 57 | GET | /api/v2/records/{accountId}/storage-usage | JWT | Yes | XSRF | Get storage usage |
| 58 | GET | /api/v2/records/follower/{accountId}/{pid} | JWT | Yes | XSRF | Get recordings by follower |
| 59 | POST | /api/v2/broadcasts/{accountId} | JWT | Yes | XSRF | Start/manage broadcast |
| 60 | POST | /api/streams/record-now/{accountId} | JWT | Yes | XSRF | Start immediate recording |
| 61 | POST | /api/streams/rebuild/{accountId} | JWT | Yes | XSRF | Rebuild stream |
| 62 | POST | /api/v2/record-profile/{accountId} | JWT | Yes | XSRF | Record profile activity |
| 63 | POST | /api/v2/record-profile/{username} | JWT | Yes | XSRF | Record by username (native Java) |
| 64 | GET | /api/v2/favorites/{accountId}/list | JWT | Yes | XSRF | List favorites |
| 65 | POST | /api/v2/favorites/{accountId}/{recordId} | JWT | Yes | XSRF | Add to favorites |
| 66 | DELETE | /api/v2/favorites/{accountId}/{recordId} | JWT | Yes | XSRF | Remove from favorites |
| 67 | POST | /api/v2/purchase/storage | JWT | Yes | XSRF | Purchase storage |
| 68 | POST | /api/v2/purchase/slots | JWT | Yes | XSRF | Purchase slots |
| 69 | GET | /api/v2/purchase/history-list | JWT | Yes | XSRF | Purchase history |
| 72 | POST | ws.arkzynco.com/api/history/fetch | JWT | No | — | Authenticated history (via xe instance) |
| 73 | POST | /api/v2/logout | JWT | Yes | XSRF | Logout; excluded from auto-logout interceptor |
| 74 | POST | /api/v2/network-log | JWT | Yes | XSRF | Submit network diagnostic log |
| 75 | GET | /api/v2/news | JWT | Yes | XSRF | Get news feed |
| 76 | GET | /api/v2/promotions | JWT | Yes | XSRF | Get promotions |
| 77 | GET | /api/v2/flip-game/state | JWT | Yes | XSRF | Get flip game state |
| 78 | POST | /api/v2/flip-game/play | JWT | Yes | XSRF | Play flip game |
| 82 | POST | ws.arkzynco.com/api/followers/{accountId} | JWT | No | — | Legacy add follower (via xe instance) |
| 86 | GET | /api/v2/user/profile | JWT | Yes | XSRF | Get user profile |
| 87 | POST | ws.arkzynco.com/api/followers/{uid} | JWT | No | — | Legacy follower (via xe instance) |
| 88 | POST | /api/v2/followers/{uid}/{pid} | JWT | Yes | XSRF | Add follower (alternate route) |

---

## 3. Authentication Flow Diagrams

### 3.1 Login Flow

```
Client                          Server (raid.arkzynco.com)
  │                                    │
  │  GET /api/csrf                     │
  │───────────────────────────────────>│
  │  200 OK                            │
  │  Set-Cookie: XSRF-TOKEN=xxx       │
  │<───────────────────────────────────│
  │                                    │
  │  POST /api/login                   │
  │  Body: { email, password }         │
  │  Cookie: XSRF-TOKEN=xxx           │
  │───────────────────────────────────>│
  │  200 OK                            │
  │  { token, user, ... }              │
  │  Set-Cookie: laravel_session=xxx   │
  │<───────────────────────────────────│
  │                                    │
  │  [token stored in localStorage]    │
  │  [session cookie stored]           │
```

### 3.2 Authenticated Request Flow

```
Client                          Server (raid.arkzynco.com)
  │                                    │
  │  POST /api/v2/user                 │
  │  Authorization: Bearer <jwt>       │
  │  X-XSRF-TOKEN: <xsrf-token>       │
  │  Cookie: XSRF-TOKEN=xxx           │
  │  Cookie: laravel_session=xxx      │
  │───────────────────────────────────>│
  │                                    │
  │  [server validates JWT]            │
  │  [server validates CSRF token]     │
  │                                    │
  │  200 OK                            │
  │<───────────────────────────────────│
```

### 3.3 Public Endpoint Flow (via Bg instance)

```
Client                          Server (ws.arkzynco.com)
  │                                    │
  │  GET /api/pages/faq                │
  │  Content-Type: application/json    │
  │  [no auth headers]                 │
  │───────────────────────────────────>│
  │  200 OK                            │
  │<───────────────────────────────────│
```

### 3.4 Error Response Flow

```
Client                          Server
  │                                    │
  │  Request with bad/expired token    │
  │───────────────────────────────────>│
  │                                    │
  │  401 { "message": "Unauthenticated." }
  │<───────────────────────────────────│
  │                                    │
  │  [interceptor: redirect to login]  │
  │  [clear localStorage token]        │
  │  [navigate to /login]              │
```

---

## 4. Token Lifecycle

### 4.1 Issuance

```
POST /api/login  or  POST /api/signin
        │
        ▼
  ┌─────────────────────┐
  │  Server validates    │
  │  credentials         │
  │  (email + password)  │
  └─────────┬───────────┘
            │
            ▼
  ┌─────────────────────┐
  │  JWT created with    │
  │  payload: {          │
  │    sub: userId,      │
  │    exp: timestamp,   │
  │    iat: timestamp,   │
  │    ...               │
  │  }                   │
  └─────────┬───────────┘
            │
            ▼
  ┌─────────────────────┐
  │  Response:           │
  │  { token: "eyJ...",  │
  │    user: {...},      │
  │    ... }             │
  └─────────┬───────────┘
            │
            ▼
  [Client stores token in localStorage]
```

### 4.2 Storage

| Storage Location | Type | Persistence | Accessible By |
|------------------|------|-------------|---------------|
| `localStorage` | JWT string | Until explicit deletion | JavaScript on same origin |
| `XSRF-TOKEN` cookie | CSRF token | Session/transient | Browser auto-sends |
| `laravel_session` cookie | Session ID | Session | Browser auto-sends |

### 4.3 Injection (per request)

```
Axios interceptor on instance "u":
  │
  ├── if (localStorage.token)
  │     config.headers.Authorization = `Bearer ${token}`
  │
  └── config.xsrfCookieName = "XSRF-TOKEN"
      config.xsrfHeaderName = "X-XSRF-TOKEN"
      → Axios reads cookie, injects header automatically
```

### 4.4 Validation (server-side)

```
Request arrives
  │
  ├── 1. Extract Bearer token from Authorization header
  ├── 2. Verify JWT signature against secret
  ├── 3. Check exp claim against current time
  ├── 4. Verify CSRF token (X-XSRF-TOKEN header matches XSRF-TOKEN cookie)
  ├── 5. (Optional) pl() helper validates additional status checks
  │
  └── Result:
        200 → authorized
        401 → "Unauthenticated." or "session_expired"
        403 → "sentry-block" or "challenge"
        429 → "Too Many Requests"
```

### 4.5 Expiry Handling

```
Token expires (exp < now)
        │
        ▼
  Server returns 401
        │
        ▼
  Response interceptor fires:
  ├── message === "Unauthenticated."
  │     → redirect to login
  ├── message === "session_expired"
  │     → auto-logout, redirect to login
  ├── message === "logged-out"
  │     → auto-logout, redirect to login
  └── message === "sentry-block"
        → display block message (no redirect)
```

### 4.6 Logout

```
POST /api/v2/logout  (endpoint #73)
  │
  ├── [excluded from auto-logout interceptor]
  │     → prevents infinite loop
  │
  ├── Client sends request with current token
  │
  └── Server:
        ├── Invalidates token/session
        ├── Clears server-side session
        └── Returns 200

  Client:
    ├── Remove token from localStorage
    ├── Clear XSRF-TOKEN cookie
    ├── Clear laravel_session cookie
    └── Redirect to /login
```

---

## 5. CSRF Protection Analysis

### 5.1 CSRF Token Flow

```
1. GET /api/csrf (endpoint #8)
   │
   ├── No auth required
   ├── Server sets XSRF-TOKEN cookie
   └── Returns 200 with empty body (or token in response)
       │
       ▼
2. Subsequent requests:
   │
   ├── Axios (instance "u") with withXSRFToken: true
   │   ├── Reads XSRF-TOKEN from cookie
   │   ├── Sends as X-XSRF-TOKEN header
   │   └── Server compares header value == cookie value
   │
   └── Validation:
       ├── Match → request proceeds
       └── Mismatch → 419 (token mismatch) or 403
```

### 5.2 CSRF Applicability

| Endpoint Group | CSRF Required | Reason |
|----------------|---------------|--------|
| All JWT endpoints on raid.arkzynco.com | Yes | Same-origin cookie-based protection |
| Legacy ws.arkzynco.com endpoints (#72, #82, #87) | No | Cross-origin, Bearer token only |
| Public endpoints | No | No state-changing auth context |

### 5.3 Key Observations

- CSRF protection is **implicit** via Laravel's framework default, not explicitly enforced per-endpoint in the client
- The `withXSRFToken: true` flag on the `u` instance ensures all main-origin requests carry CSRF headers
- The `Bg` and `xe` instances do **not** use withXSRFToken, relying solely on Bearer token or no auth
- No endpoint uses CSRF-only authentication — CSRF is always paired with JWT on the main instance

---

## 6. OAuth Provider Details

### 6.1 Supported Providers

| Provider | Client ID Field | Auth Flow | Token Endpoint | Notes |
|----------|-----------------|-----------|----------------|-------|
| **Google** | Google Client ID | OAuth 2.0 Authorization Code | Google token endpoint | Social login via redirect |
| **Apple** | Apple Client ID | Sign in with Apple | Apple token endpoint | JWT-based identity |
| **Twitter** | Twitter Client ID | OAuth 1.0a / OAuth 2.0 | Twitter token endpoint | API key based |
| **Facebook** | Facebook Client ID | Facebook Login | Facebook Graph API | Access token exchange |

### 6.2 OAuth Flow

```
Client                    TKREC Server              OAuth Provider
  │                            │                         │
  │  Click "Login with Google"  │                         │
  │───────────────────────────>│                         │
  │                            │                         │
  │  Redirect to Google OAuth   │                         │
  │<───────────────────────────│                         │
  │                            │                         │
  │  User authenticates         │                         │
  │─────────────────────────────────────────────────────>│
  │                            │                         │
  │  Authorization code returned│                         │
  │<─────────────────────────────────────────────────────│
  │                            │                         │
  │  POST /api/login            │                         │
  │  { provider, code, ... }    │                         │
  │───────────────────────────>│                         │
  │                            │  Exchange code for token │
  │                            │────────────────────────>│
  │                            │  Access token returned   │
  │                            │<────────────────────────│
  │                            │                         │
  │                            │  Verify user identity    │
  │                            │  Create/link account     │
  │                            │                         │
  │  200 OK                     │                         │
  │  { token: "eyJ...", ... }   │                         │
  │<───────────────────────────│                         │
```

### 6.3 Client ID Locations

OAuth client IDs are embedded in the frontend bundle, typically found in:
- API request payloads to `/api/login` or `/api/signin` with `provider` field
- Frontend OAuth initialization scripts
- Environment configuration bundled into the production build

---

## 7. Security Observations

### 7.1 Token Storage — localStorage (HIGH RISK)

```
RISK: JWT stored in localStorage
  │
  ├── XSS Attack Vector:
  │   Any injected JavaScript can read:
  │   localStorage.getItem('token')
  │   → Full account compromise
  │
  ├── No HttpOnly Flag:
  │   localStorage has no server-controlled access restrictions
  │
  ├── No SameSite Protection:
  │   Accessible to any script on the same origin
  │
  └── Mitigation Already Present:
      ├── Bearer token + CSRF cookie = double-submit protection
      └── But CSRF protection does NOT prevent XSS token theft
```

**Recommendation:** Migrate JWT to HttpOnly, Secure, SameSite=Lax cookies to prevent XSS token exfiltration.

### 7.2 Auto-Logout Behavior

```
Response Interceptor Logic:
  │
  ├── 401 (non-logout endpoints):
  │   ├── "Unauthenticated." → redirect to /login
  │   ├── "session_expired"  → auto-logout → redirect to /login
  │   └── "logged-out"       → auto-logout → redirect to /login
  │
  ├── 403:
  │   ├── "sentry-block"     → display block message (no redirect)
  │   └── "challenge"        → display CAPTCHA (challenge flow)
  │
  ├── 429:
  │   └── "Too Many Requests" → rate limit message (no redirect)
  │
  └── Endpoint #73 (/api/v2/logout):
      └── EXCLUDED from auto-logout interceptor
          → prevents infinite redirect loop
```

### 7.3 Rate Limiting

| Endpoint Group | Rate Limit Evidence | Behavior |
|----------------|---------------------|----------|
| Login endpoints (#1-4) | 429 response | "Too Many Requests" message |
| Password reset (#6) | Likely rate-limited | Standard Laravel throttle |
| All endpoints | Possible per-IP or per-user | Framework-level throttling |

### 7.4 CSRF vs XSS Trade-off Summary

| Protection | CSRF Attack | XSS Attack | Status |
|------------|-------------|------------|--------|
| JWT in localStorage | Partially mitigated by CSRF | Vulnerable | **Weak** |
| XSRF-TOKEN cookie | Mitigated | No protection | **Partial** |
| Bearer + CSRF double-submit | Mitigated | Vulnerable | **Partial** |
| HttpOnly cookie (recommended) | Mitigated | Mitigated | **Strong** |

### 7.5 Cross-Origin Exposure

| Instance | Origin | Auth | Risk |
|----------|--------|------|------|
| `u` (main) | raid.arkzynco.com | JWT + CSRF | Standard — double-submit |
| `Bg` (public) | ws.arkzynco.com | None | Low — read-only public data |
| `xe` (secondary) | ws.arkzynco.com | JWT only | **Medium** — no CSRF on cross-origin |

The `xe` instance sends Bearer tokens to a different origin (ws.arkzynco.com) without CSRF protection. While this prevents traditional CSRF (different origin), it exposes the JWT to a secondary domain.

### 7.6 Endpoint Security Classification

| Classification | Count | Description |
|----------------|-------|-------------|
| **Public / Unauthenticated** | 15 | No credentials required |
| **Authenticated (JWT + CSRF)** | 69 | Full protection on main origin |
| **Authenticated (JWT only)** | 4 | Legacy/cross-origin, no CSRF |
| **Error-state special** | 0 | (counted in authenticated) |

### 7.7 Known Vulnerabilities

1. **JWT in localStorage** — XSS can exfiltrate tokens
2. **Legacy endpoints without CSRF** — ws.arkzynco.com endpoints (#72, #82, #87) lack CSRF protection
3. **GET-based state changes** — #27 (`/rm`) uses GET for a destructive operation (follower removal)
4. **Password in POST body** — No evidence of HTTPS enforcement or CSP headers
5. **session_expired ambiguity** — Same 401 code as "Unauthenticated." — requires message string matching

---

## 8. Appendix: Endpoint Groups by Function

| Group | Endpoints | Count | Auth |
|-------|-----------|-------|------|
| **Authentication** | #1-8, #83-85 | 11 | None |
| **User Management** | #9-20, #86 | 13 | JWT |
| **Follower Operations** | #21-45, #82, #87-88 | 27 | JWT |
| **Recording Operations** | #46-58 | 13 | JWT |
| **Streaming** | #59-61 | 3 | JWT |
| **Record Profile** | #62-63 | 2 | JWT |
| **Favorites** | #64-66 | 3 | JWT |
| **Purchases** | #67-69 | 3 | JWT |
| **Public Content** | #70-71, #79-81, #85 | 5 | None |
| **Status & Logging** | #10, #74-76 | 4 | JWT |
| **Flip Game** | #77-78 | 2 | JWT |
| **Logout** | #73 | 1 | JWT |
| **History** | #72 | 1 | JWT |
| **Status Check** | #10 | 1 | JWT |
