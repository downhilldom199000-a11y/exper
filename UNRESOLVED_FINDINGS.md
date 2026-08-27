# TKREC Unresolved Findings & Contradictions

> Generated: 2026-08-26
> Cross-check results, contradictions, and unresolved issues from reverse engineering

---

## Executive Summary

This document records contradictions, unresolved issues, and cross-check findings from the TKREC 1.3.1 reverse engineering analysis. Of 18 identified findings:

- **1 CRITICAL** — base URL contradiction that invalidated prior endpoint documentation
- **3 HIGH** — unresolved endpoint behavior, legacy overlap, and security considerations
- **5 MEDIUM** — partially resolved issues requiring further investigation
- **5 LOW** — minor discrepancies or accepted risks
- **4 resolved** — confirmed and documented

The most significant finding is a **base URL contradiction** that split all previously documented endpoints across three separate axios instances with different authentication models. This has been corrected in API_MASTER.json.

---

## Findings

### 1. Base URL Contradiction

| Field | Value |
|---|---|
| **Severity** | CRITICAL |
| **Status** | Resolved (corrected in API_MASTER.json) |

**Static Analysis (old):** All endpoints listed under `ws.arkzynco.com`

**JS Source (corrected):** Three separate axios instances exist:

| Instance | Base URL | Auth | CSRF | withCredentials |
|---|---|---|---|---|
| `u` (main) | `https://raid.arkzynco.com/` | Yes | Yes | Yes |
| `Bg` (public) | `ws.arkzynco.com` | No | No | No |
| `xe` (secondary) | `ws.arkzynco.com` | Yes | No | — |

**Impact:** All old static analysis documents have wrong base URLs for most endpoints. The main authenticated API lives on `raid.arkzynco.com`, not `ws.arkzynco.com`.

**Resolution:** API_MASTER.json has correct URLs per endpoint.

---

### 2. Static Endpoint Count Discrepancy

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Resolved |

- **Original static analysis:** 76 unique endpoints
- **API_MASTER.json:** 88 total (83 confirmed static + 5 undocumented/legacy)

**Reason:** Original count missed some endpoints; API_MASTER added legacy/undocumented ones found in JS source.

**Resolution:** 88 is the accurate count.

---

### 3. Public Stream Count 404

| Field | Value |
|---|---|
| **Severity** | MEDIUM |
| **Status** | Unresolved |

- **Endpoint:** `POST /api/public/stream-count`
- **Expected:** Returns stream count for onboarding teaser
- **Observed:** Returns 404
- **Impact:** Onboarding flow incomplete — users see no stream count
- **Hypothesis:** Endpoint may have been deprecated, moved, or requires a specific request format

---

### 4. CSRF Endpoint Authentication Confusion

| Field | Value |
|---|---|
| **Severity** | MEDIUM |
| **Status** | Partially resolved |

- **Endpoint:** `GET /api/csrf`
- **Documented:** "no auth required"
- **Observed:** Returns 401 in some cases

**Analysis:** CSRF token is needed for state-changing requests, but the endpoint itself may require a session cookie. CSRF works after initial auth.

---

### 5. /api/status Authentication Model

| Field | Value |
|---|---|
| **Severity** | MEDIUM |
| **Status** | Resolved |

- **Endpoint:** `POST /api/status`
- **Observed:** Returns 401 without Bearer token
- **Also accepts:** OAuth token in Authorization header (different from JWT)
- **Function:** Exchanges social OAuth token for JWT — uses `pl()` helper function

**Resolution:** This is the OAuth token exchange endpoint.

---

### 6. Legacy vs Modern Endpoint Overlap

| Field | Value |
|---|---|
| **Severity** | HIGH |
| **Status** | Unresolved |

**Legacy endpoints (82-84, 87):**

| Legacy | Modern Equivalent |
|---|---|
| `POST ws.arkzynco.com/api/followers/{accountId}` | `POST /api/v2/followers/{accountId}` |
| `POST ws.arkzynco.com/api/profile/login` | `POST /api/signin` |
| `POST ws.arkzynco.com/api/profile/code` | `POST /api/confirm-email` |

**Question:** Are legacy endpoints still functional or deprecated?

**Status:** No traffic observed for legacy endpoints. Unresolved.

---

### 7. /api/v2/user/profile Ambiguity

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Resolved |

- `POST /api/v2/user/profile` — create profile (ID 16)
- `GET /api/v2/user/profile` — list profiles (ID 86, undocumented)

Both use the same path, different methods. Standard REST convention (POST=create, GET=list).

---

### 8. Conditional Path Logic

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Resolved |

- **Endpoint:** `POST /api/v2/followers/{uid}/{pid}`
- **Behavior:** Appends `/free` to path when `s.isFree=true`
- **Result:** `/api/v2/followers/{uid}/{pid}` or `/api/v2/followers/{uid}/{pid}/free`

Documented in API_MASTER.json.

---

### 9. Dynamic Path Parameters

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Resolved |

- `POST /api/profile/login/{id}` — path includes profile ID if profile exists
- `POST /api/profile/code/{id}` — same pattern

Conditional path based on profile existence. Documented in API_MASTER.json.

---

### 10. WebView Debugging Limitation

| Field | Value |
|---|---|
| **Severity** | MEDIUM |
| **Status** | Limitation — no resolution without root/Frida |

- **Issue:** Cannot inspect WebView network traffic or JS console
- **Impact:** Cannot observe actual API calls made by the app
- **Mitigation:** Logcat shows some network activity, API probing confirmed endpoints

---

### 11. HTTPS MITM Not Possible

| Field | Value |
|---|---|
| **Severity** | MEDIUM |
| **Status** | Limitation — requires rooted device or Frida |

- **Issue:** Cannot intercept HTTPS traffic without root
- **Impact:** Cannot see full request/response bodies
- **Mitigation:** JS source analysis provides request/response schemas

---

### 12. Video URL Security (EDT Token)

| Field | Value |
|---|---|
| **Severity** | HIGH |
| **Status** | Unresolved |

- **Endpoint:** `POST /api/v2/records/{accountId}/{recordId}/ups`
- **Body:** EDT token (base64-encoded event+timestamp)
- **Question:** How does server validate EDT? Is it time-based? Replay protection?
- **Status:** EDT algorithm partially understood from JS source

---

### 13. Rate Limiting Behavior

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Partially resolved |

- **Observed:** 429 "Too Many Requests" responses
- **Pattern:** Exponential backoff (90s base, max 600s)
- **Scope:** Per-profile polling state tracking
- **Status:** Client-side rate limiting understood; server-side limits unknown

---

### 14. Flip Game Endpoints

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Partially resolved |

- `GET /api/v2/flip-game/state` — returns `{canFlip, flipBalance, dailyBonusFlip, isJackpotReady}`
- `POST /api/v2/flip-game/play` — no body required
- **Question:** What is the flip game? Gamification reward system?
- **Status:** Gamification feature confirmed; details unknown

---

### 15. Sentry DSN Exposure

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Accepted risk |

- **DSN:** `https://b9ed440e5457ca44ecdc3d8f20b3f109@o234722.ingest.us.sentry.io/4509434825867264`
- **Risk:** Sentry DSN is public by design, but exposes project ID
- **Status:** Low risk — DSN is meant to be client-side

---

### 16. OAuth Client IDs Exposed

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Accepted risk |

- **Google:** `139825549831-2i033uihh1rf7ne69p31qq5a0f7hf0er.apps.googleusercontent.com`
- **Apple:** `com.tkrec.app`
- **Risk:** Client IDs are public by design (needed for OAuth flows)
- **Status:** Normal — not a security issue

---

### 17. Token Storage in localStorage

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Low risk in practice |

- **Issue:** JWT token stored in localStorage (accessible to XSS)
- **Impact:** If XSS vulnerability exists, token can be stolen
- **Mitigation:** WebView-based app, no external JS execution
- **Status:** Not best practice, but low risk in current architecture

---

### 18. Native Java vs WebView Consistency

| Field | Value |
|---|---|
| **Severity** | LOW |
| **Status** | Resolved |

- **Java (ShareReceiverActivity.java):** `POST /api/v2/record-profile/{username}` (username in path)
- **JS:** `POST /api/v2/record-profile/{accountId}` (accountId in path)

Different path params (username vs accountId), both valid and serve different purposes.

---

## Summary Table

| # | Finding | Severity | Status |
|---|---|---|---|
| 1 | Base URL Contradiction | CRITICAL | Resolved |
| 2 | Endpoint Count Discrepancy | LOW | Resolved |
| 3 | Public Stream Count 404 | MEDIUM | Unresolved |
| 4 | CSRF Auth Confusion | MEDIUM | Partially resolved |
| 5 | /api/status Auth Model | MEDIUM | Resolved |
| 6 | Legacy Endpoint Overlap | HIGH | Unresolved |
| 7 | /api/v2/user/profile Ambiguity | LOW | Resolved |
| 8 | Conditional Path Logic | LOW | Resolved |
| 9 | Dynamic Path Parameters | LOW | Resolved |
| 10 | WebView Debugging Limitation | MEDIUM | Limitation |
| 11 | HTTPS MITM Limitation | MEDIUM | Limitation |
| 12 | Video URL EDT Security | HIGH | Unresolved |
| 13 | Rate Limiting Behavior | LOW | Partially resolved |
| 14 | Flip Game Endpoints | LOW | Partially resolved |
| 15 | Sentry DSN Exposure | LOW | Accepted risk |
| 16 | OAuth Client IDs | LOW | Accepted risk |
| 17 | Token in localStorage | LOW | Low risk |
| 18 | Java vs WebView Consistency | LOW | Resolved |

---

## Recommendations for Resolution

1. **Prioritize finding #6 (Legacy Endpoints):** Determine if legacy endpoints are still active by sending test requests and observing responses. If deprecated, remove from API_MASTER.json and mark as historical.

2. **Investigate finding #3 (Stream Count 404):** Test the endpoint with different request bodies, headers, and content types. Check if it requires authentication or a specific `Content-Type`.

3. **Clarify finding #12 (EDT Token):** Continue JS source analysis to reverse the EDT generation algorithm. Look for time-based validation, replay protection, or server-side state checks.

4. **Resolve finding #4 (CSRF):** Map the exact conditions under which `GET /api/csrf` returns 401 vs 200. Determine if it requires a pre-existing session cookie.

5. **Resolve finding #13 (Rate Limits):** Perform controlled rate-limit testing to map server-side limits independently of client-side backoff.

---

## Limitations of the Analysis

- **No root access:** Cannot perform MITM on HTTPS traffic, limiting request/response body inspection to JS source analysis.
- **No WebView debugging:** Network traffic and JS console inside the app's WebView are unobservable without Frida or root.
- **Static JS analysis only:** All API schemas inferred from obfuscated JavaScript source; no runtime confirmation of edge cases.
- **No test account:** Analysis conducted without a live authenticated session for most endpoints.
- **Single version:** Findings apply to TKREC 1.3.1 only; behavior may differ across versions.

---

## Cross-Check Results

| Analysis Type | Finding |
|---|---|
| Static (Java) vs JS Source | Endpoint count: 76 (static) vs 88 (JS) — resolved |
| Static (Java) vs JS Source | Path params: username vs accountId (#18) — resolved, both valid |
| Static vs API_MASTER | Base URLs: all wrong in static, corrected in API_MASTER |
| Dynamic (API probing) vs JS Source | Stream count: JS claims exists, API returns 404 — unresolved |
| JS Source vs API probing | CSRF: JS claims no auth needed, API sometimes returns 401 — partially resolved |
| JS Source vs API probing | /api/status: JS shows OAuth exchange, API confirms — resolved |
