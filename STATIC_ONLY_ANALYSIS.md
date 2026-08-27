# TKREC Static-Only Endpoint Analysis

> Generated: 2026-08-26
> 45 endpoints identified in static analysis, not confirmed dynamically

---

## 1. Summary

| Category | Count | Status |
|---|---|---|
| Records (CRUD) | 12 | NOT_TESTED |
| Followers Tracking | 12 | NOT_TESTED |
| User Profiles | 5 | NOT_TESTED |
| Followers Social | 4 | NOT_TESTED |
| Authentication | 2 | NOT_TESTED |
| Favorites | 2 | NOT_TESTED |
| Miscellaneous | 3 | NOT_TESTED |
| Legacy | 5 | NOT_TESTED |
| Dynamic but broken | 1 | BROKEN (404) |

---

## 2. Detailed Endpoint Analysis

### 2.1 Authentication

#### 4. POST /api/login-device
- **Path:** `/api/login-device`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Yes — standard device-based auth flow
- **Expected behavior:** Accepts `device_id` in body, returns JWT + session
- **Dependencies:** Valid device_id string, possibly tied to prior registration
- **Why NOT_TESTED:** Requires valid device_id mapped to existing account; cannot generate without physical device or prior registration
- **Risk:** MEDIUM — may be deprecated in favor of standard auth

#### 7. POST /api/reset-password/{token}
- **Path:** `/api/reset-password/{token}`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — password reset is a core auth flow
- **Expected behavior:** Accepts `{token}` in path + new password in body, returns success/failure
- **Dependencies:** Valid reset token (obtained via email/SMS flow at separate endpoint)
- **Why NOT_TESTED:** No reset token available; would need to trigger full password reset flow
- **Risk:** LOW — standard pattern, almost certainly functional

---

### 2.2 User Profiles

#### 17. POST /api/v2/user/profile/{id}
- **Path:** `/api/v2/user/profile/{id}`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — profile update is core functionality
- **Expected behavior:** Accepts profile fields (name, avatar, bio, etc.), returns updated profile
- **Dependencies:** Valid JWT, matching `id` to authenticated user
- **Why NOT_TESTED:** Requires authenticated session + valid profile ID
- **Risk:** LOW — standard CRUD update

#### 18. DELETE /api/v2/user/profile/{id}
- **Path:** `/api/v2/user/profile/{id}`
- **Method:** DELETE
- **Confidence:** HIGH
- **Likely functional:** Yes — profile deletion is standard
- **Expected behavior:** Deletes user profile, returns 200/204
- **Dependencies:** Valid JWT, matching `id` to authenticated user
- **Why NOT_TESTED:** Destructive — would delete real data
- **Risk:** LOW — standard CRUD delete, but destructive

#### 19. PUT /api/v2/user/profile/{id}/logout
- **Path:** `/api/v2/user/profile/{id}/logout`
- **Method:** PUT
- **Confidence:** MEDIUM
- **Likely functional:** Yes — logout is essential
- **Expected behavior:** Invalidates current session/token, returns 200
- **Dependencies:** Valid JWT, valid profile ID
- **Why NOT_TESTED:** Requires active session to test logout
- **Risk:** LOW — standard logout flow

#### 20. POST /api/v2/user/profile/{id}/mute
- **Path:** `/api/v2/user/profile/{id}/mute`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — mute is common in social apps
- **Expected body:** `{ muted: boolean }` or toggle behavior
- **Dependencies:** Valid JWT, valid profile ID, possibly target user ID
- **Why NOT_TESTED:** Requires authenticated session + target profile
- **Risk:** LOW — non-destructive toggle

#### 15. POST /api/v2/user/transfer-recordings/{accountId}/{targetId}
- **Path:** `/api/v2/user/transfer-recordings/{accountId}/{targetId}`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Possibly — transfer recordings between accounts is niche
- **Expected behavior:** Moves recordings from `accountId` to `targetId`
- **Dependencies:** Valid JWT, both accounts must exist, source must have recordings
- **Why NOT_TESTED:** Requires two valid accounts + recordings; destructive to source
- **Risk:** MEDIUM — niche feature, may be restricted or internal-only

---

### 2.3 Followers Tracking

#### 23. DELETE /api/v2/followers/{accountId}/{id}
- **Path:** `/api/v2/followers/{accountId}/{id}`
- **Method:** DELETE
- **Confidence:** HIGH
- **Likely functional:** Yes — remove a tracked profile
- **Expected behavior:** Deletes follower tracking entry, returns 200/204
- **Dependencies:** Valid JWT, `accountId` must have tracked profile `{id}`
- **Why NOT_TESTED:** Destructive operation on tracking data
- **Risk:** LOW — standard CRUD delete

#### 24. PUT /api/v2/followers/{accountId}/{id}
- **Path:** `/api/v2/followers/{accountId}/{id}`
- **Method:** PUT
- **Confidence:** HIGH
- **Likely functional:** Yes — toggle tracking status (pause/resume)
- **Expected body:** `{ active: boolean }` or similar toggle
- **Dependencies:** Valid JWT, follower entry must exist
- **Why NOT_TESTED:** Requires active tracking entry to toggle
- **Risk:** LOW — non-destructive toggle

#### 25. GET /api/v2/followers/{accountId}/{pid}
- **Path:** `/api/v2/followers/{accountId}/{pid}`
- **Method:** GET
- **Confidence:** HIGH
- **Likely functional:** Yes — get single tracked profile details
- **Expected behavior:** Returns follower object with status, last seen, etc.
- **Dependencies:** Valid JWT, follower must exist
- **Why NOT_TESTED:** Requires valid accountId + pid with existing tracking
- **Risk:** LOW — read-only

#### 26. GET /api/v2/followers/{accountId}/{pid}/history
- **Path:** `/api/v2/followers/{accountId}/{pid}/history`
- **Method:** GET
- **Confidence:** HIGH
- **Likely functional:** Yes — live history for tracked profile
- **Expected behavior:** Returns array of historical status changes (live/offline events)
- **Dependencies:** Valid JWT, follower must exist, historical data must exist
- **Why NOT_TESTED:** Requires active tracking with history
- **Risk:** LOW — read-only

#### 27. GET /api/v2/followers/{accountId}/{pid}/rm
- **Path:** `/api/v2/followers/{accountId}/{pid}/rm`
- **Method:** GET
- **Confidence:** LOW
- **Likely functional:** Possibly — "removal info" is ambiguous
- **Expected behavior:** May return metadata about removal or be an alias for DELETE
- **Dependencies:** Valid JWT, follower must exist
- **Why NOT_TESTED:** Ambiguous purpose; unclear if it returns info or performs action
- **Risk:** MEDIUM — unclear semantics, may be deprecated

#### 28. POST /api/v2/followers/{accountId}/{pid}/fetch
- **Path:** `/api/v2/followers/{accountId}/{pid}/fetch`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — force refresh a profile's data
- **Expected behavior:** Triggers immediate re-fetch of follower's public data; returns updated profile
- **Dependencies:** Valid JWT, follower must exist, rate limiting may apply
- **Why NOT_TESTED:** Requires valid tracking entry; may trigger external API calls
- **Risk:** LOW — force-refresh is standard in tracking apps

#### 29. POST /api/v2/followers/{accountId}/{pid}/validate
- **Path:** `/api/v2/followers/{accountId}/{pid}/validate`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Yes — validate if profile is still trackable/accessible
- **Expected behavior:** Checks if `{pid}` is valid and accessible; returns boolean or status
- **Dependencies:** Valid JWT, valid `pid`
- **Why NOT_TESTED:** Requires authenticated session; may hit external platform APIs
- **Risk:** LOW — validation check, read-only side effects

#### 30. POST /api/v2/followers/{accountId}/{pid}/deep-check
- **Path:** `/api/v2/followers/{accountId}/{pid}/deep-check`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Yes — more thorough validation than `/validate`
- **Expected behavior:** Full profile verification (private status, ban status, etc.); may be async
- **Dependencies:** Valid JWT, valid `pid`; may have rate limits or cost
- **Why NOT_TESTED:** Requires authenticated session; may be expensive server-side
- **Risk:** LOW — validation, but potentially heavier processing

#### 32. POST /api/v2/followers/{accountId}/{pid}/following/fetch
- **Path:** `/api/v2/followers/{accountId}/{pid}/following/fetch`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — fetch who a tracked profile follows
- **Expected behavior:** Returns list of accounts `{pid}` follows; may require scraping
- **Dependencies:** Valid JWT, target profile must be accessible (not private)
- **Why NOT_TESTED:** May require platform access or scraping; privacy concerns
- **Risk:** MEDIUM — may be restricted by platform ToS or rate limits

#### 33. GET /api/v2/followers/{accountId}/{uid}/following
- **Path:** `/api/v2/followers/{accountId}/{uid}/following`
- **Method:** GET
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — cached following list for a profile
- **Expected behavior:** Returns previously fetched following list
- **Dependencies:** Valid JWT, prior call to `/following/fetch` or cached data
- **Why NOT_TESTED:** Depends on data from `/following/fetch`; may return empty without it
- **Risk:** LOW — read-only, serves cached data

#### 34. GET /api/v2/followers/{accountId}/{uid}/deep-check
- **Path:** `/api/v2/followers/{accountId}/{uid}/deep-check`
- **Method:** GET
- **Confidence:** MEDIUM
- **Likely functional:** Yes — GET variant of deep-check (idempotent)
- **Expected behavior:** Returns cached deep-check result or runs fresh check
- **Dependencies:** Valid JWT, valid `uid`
- **Why NOT_TESTED:** Requires authenticated session; GET vs POST variant unclear
- **Risk:** LOW — read-only variant

---

### 2.4 Followers Social

#### 36. GET /api/v2/followers/discover/{accountId}/bubbles
- **Path:** `/api/v2/followers/discover/{accountId}/bubbles`
- **Method:** GET
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — "bubbles" suggests UI feature (suggested profiles, activity bubbles)
- **Expected behavior:** Returns array of suggested/recommended profiles or activity indicators
- **Dependencies:** Valid JWT, algorithm-based (needs tracking history or social graph)
- **Why NOT_TESTED:** Feature-specific; may require specific app state or subscription
- **Risk:** LOW — discovery feature, likely active if UI references it

#### 40. GET /api/v2/followers/similar/{accountId}/{pid}
- **Path:** `/api/v2/followers/similar/{accountId}/{pid}`
- **Method:** GET
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — find profiles similar to a tracked one
- **Expected behavior:** Returns array of similar profile IDs/names
- **Dependencies:** Valid JWT, valid `pid`, server-side similarity engine
- **Why NOT_TESTED:** Requires valid tracked profile; similarity engine may need data
- **Risk:** LOW — social feature, read-only

#### 42. POST /api/v2/followers/{accountId}/{pid}/like
- **Path:** `/api/v2/followers/{accountId}/{pid}/like`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Yes — like/unlike a tracked profile
- **Expected body:** `{ liked: boolean }` or toggle behavior
- **Dependencies:** Valid JWT, follower must exist
- **Why NOT_TESTED:** Requires authenticated session + tracking entry
- **Risk:** LOW — simple toggle, non-destructive

#### 44. POST /api/v2/followers/hide/{accountId}/{pid}
- **Path:** `/api/v2/followers/hide/{accountId}/{pid}`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Yes — hide a profile from the list without removing
- **Expected body:** `{ hidden: boolean }` or toggle
- **Dependencies:** Valid JWT, follower must exist
- **Why NOT_TESTED:** Requires authenticated session + tracking entry
- **Risk:** LOW — UI organization feature, non-destructive

---

### 2.5 Records (CRUD)

#### 47. GET /api/v2/records/{accountId}/{recordId}
- **Path:** `/api/v2/records/{accountId}/{recordId}`
- **Method:** GET
- **Confidence:** HIGH
- **Likely functional:** Yes — fetch single recording metadata
- **Expected behavior:** Returns record object with URL, duration, timestamp, status
- **Dependencies:** Valid JWT, record must exist
- **Why NOT_TESTED:** Requires valid accountId + recordId
- **Risk:** LOW — read-only, core feature

#### 48. POST /api/v2/records/new/{accountId}
- **Path:** `/api/v2/records/new/{accountId}`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — create a new recording entry
- **Expected body:** Recording metadata (source, duration, timestamp, etc.)
- **Dependencies:** Valid JWT, possibly an active live session or stream URL
- **Why NOT_TESTED:** Requires valid accountId; may need stream context
- **Risk:** LOW — core CRUD create

#### 49. DELETE /api/v2/records/{accountId}/{recordId}
- **Path:** `/api/v2/records/{accountId}/{recordId}`
- **Method:** DELETE
- **Confidence:** HIGH
- **Likely functional:** Yes — delete a single recording
- **Expected behavior:** Removes record, returns 200/204
- **Dependencies:** Valid JWT, record must exist
- **Why NOT_TESTED:** Destructive — would delete real recording data
- **Risk:** LOW — standard CRUD delete

#### 50. DELETE /api/v2/records/{accountId}/all
- **Path:** `/api/v2/records/{accountId}/all`
- **Method:** DELETE
- **Confidence:** HIGH
- **Likely functional:** Yes — delete all recordings for an account
- **Expected behavior:** Mass deletion of all records, returns 200/204
- **Dependencies:** Valid JWT, account must have recordings
- **Why NOT_TESTED:** Mass destructive operation — cannot test safely
- **Risk:** HIGH — permanent mass data loss

#### 52. POST /api/v2/records/{accountId}/delete-selected
- **Path:** `/api/v2/records/{accountId}/delete-selected`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — bulk delete specific records
- **Expected body:** `{ recordIds: string[] }` or similar
- **Dependencies:** Valid JWT, all specified records must exist
- **Why NOT_TESTED:** Destructive bulk operation
- **Risk:** MEDIUM — bulk delete, but bounded by selection

#### 53. POST /api/v2/records/{accountId}/{recordId}/attach
- **Path:** `/api/v2/records/{accountId}/{recordId}/attach`
- **Method:** POST
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — attach record to a profile or event
- **Expected body:** Attachment metadata (e.g., `{ profileId }`, `{ eventId }`, or file reference)
- **Dependencies:** Valid JWT, record must exist, target must exist
- **Why NOT_TESTED:** Ambiguous purpose; may require specific app state
- **Risk:** LOW — metadata operation, non-destructive

#### 54. PUT /api/v2/records/{accountId}/{recordId}/fix
- **Path:** `/api/v2/records/{accountId}/{recordId}/fix`
- **Method:** PUT
- **Confidence:** MEDIUM
- **Likely functional:** Yes — rebuild/repair a corrupted or incomplete recording
- **Expected behavior:** Server-side rebuild of recording metadata or file; returns 200 or 202 (async)
- **Dependencies:** Valid JWT, record must exist, source data must be available
- **Why NOT_TESTED:** Requires corrupted/incomplete record to test meaningfully
- **Risk:** LOW — repair operation, non-destructive

#### 55. POST /api/v2/records/{accountId}/{recordId}/ups
- **Path:** `/api/v2/records/{accountId}/{recordId}/ups`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Possibly — "ups" likely refers to upload/presigned URL service
- **Expected behavior:** Returns a presigned upload URL or CDN path for the recording file
- **Dependencies:** Valid JWT, record must exist, cloud storage configuration
- **Why NOT_TESTED:** Ambiguous naming; may be internal or deprecated
- **Risk:** LOW — URL generation, non-destructive

#### 56. POST /api/v2/records/{accountId}/{recordId}/downloaded
- **Path:** `/api/v2/records/{accountId}/{recordId}/downloaded`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — mark a recording as downloaded
- **Expected body:** `{ downloaded: true }` or empty body (toggle)
- **Dependencies:** Valid JWT, record must exist
- **Why NOT_TESTED:** Requires valid record to mark
- **Risk:** LOW — metadata flag, non-destructive

#### 58. GET /api/v2/records/follower/{accountId}/{pid}
- **Path:** `/api/v2/records/follower/{accountId}/{pid}`
- **Method:** GET
- **Confidence:** HIGH
- **Likely functional:** Yes — list recordings for a specific tracked profile
- **Expected behavior:** Returns array of record objects filtered by follower `pid`
- **Dependencies:** Valid JWT, follower must have recordings
- **Why NOT_TESTED:** Requires valid accountId + pid with recordings
- **Risk:** LOW — read-only, core feature

---

### 2.6 Favorites

#### 65. POST /api/v2/favorites/{accountId}/{recordId}
- **Path:** `/api/v2/favorites/{accountId}/{recordId}`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — add a record to favorites
- **Expected behavior:** Creates favorite entry, returns 200/201
- **Dependencies:** Valid JWT, record must exist
- **Why NOT_TESTED:** Requires valid accountId + recordId
- **Risk:** LOW — non-destructive, easily reversible

#### 66. DELETE /api/v2/favorites/{accountId}/{recordId}
- **Path:** `/api/v2/favorites/{accountId}/{recordId}`
- **Method:** DELETE
- **Confidence:** HIGH
- **Likely functional:** Yes — remove a record from favorites
- **Expected behavior:** Removes favorite entry, returns 200/204
- **Dependencies:** Valid JWT, favorite must exist
- **Why NOT_TESTED:** Requires prior favorite to exist
- **Risk:** LOW — non-destructive, easily re-added

---

### 2.7 Miscellaneous

#### 9. POST /api/auth/fcm
- **Path:** `/api/auth/fcm`
- **Method:** POST
- **Confidence:** HIGH
- **Likely functional:** Yes — register Firebase Cloud Messaging token
- **Expected body:** `{ token: string, platform?: "ios"|"android" }`
- **Dependencies:** Valid JWT, valid FCM token from Firebase SDK
- **Why NOT_TESTED:** Requires a real FCM token from a device/app
- **Risk:** LOW — standard push notification registration

#### 79. GET /api/slides
- **Path:** `/api/slides`
- **Method:** GET
- **Confidence:** HIGH
- **Likely functional:** Yes — onboarding tutorial slides
- **Expected behavior:** Returns array of slide objects (image URL, text, display order)
- **Dependencies:** None (likely public endpoint, no auth required)
- **Why NOT_TESTED:** Should be testable without auth; may be region-gated
- **Risk:** LOW — static content delivery

#### 81. GET /api/pages/rewards-overview
- **Path:** `/api/pages/rewards-overview`
- **Method:** GET
- **Confidence:** MEDIUM
- **Likely functional:** Possibly — rewards/earnings page content
- **Expected behavior:** Returns page config (reward tiers, current balance, referral info)
- **Dependencies:** May require authentication for personalized data; may be public for general info
- **Why NOT_TESTED:** May require subscription or specific account tier
- **Risk:** LOW — content endpoint

---

### 2.8 Legacy Endpoints

#### 82. POST ws.arkzynco.com/api/followers/{accountId}
- **Path:** `ws.arkzynco.com/api/followers/{accountId}`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Deprecated — different domain (`arkzynco.com` vs current API domain)
- **Expected behavior:** Legacy follower creation/update; may redirect, 404, or 410 Gone
- **Dependencies:** Legacy auth flow, possibly different JWT signing key
- **Why NOT_TESTED:** Different domain, likely requires legacy credentials
- **Risk:** HIGH — almost certainly deprecated

#### 83. POST ws.arkzynco.com/api/profile/login
- **Path:** `ws.arkzynco.com/api/profile/login`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Deprecated — legacy login on old domain
- **Expected behavior:** May still accept credentials for legacy accounts or return 410
- **Dependencies:** Legacy auth, may use different token format
- **Why NOT_TESTED:** Different domain, legacy flow
- **Risk:** HIGH — deprecated

#### 84. POST ws.arkzynco.com/api/profile/code
- **Path:** `ws.arkzynco.com/api/profile/code`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Deprecated — legacy verification code endpoint
- **Expected behavior:** May still accept verification codes or return 410
- **Dependencies:** Legacy auth flow
- **Why NOT_TESTED:** Different domain, legacy flow
- **Risk:** HIGH — deprecated

#### 87. POST ws.arkzynco.com/api/followers/{uid}
- **Path:** `ws.arkzynco.com/api/followers/{uid}`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Deprecated — undocumented legacy follower endpoint
- **Expected behavior:** Unknown; may be identical to #82 or a variant
- **Dependencies:** Legacy auth
- **Why NOT_TESTED:** Undocumented + legacy domain
- **Risk:** HIGH — likely dead

#### 88. POST /api/v2/followers/{uid}/{pid}
- **Path:** `/api/v2/followers/{uid}/{pid}`
- **Method:** POST
- **Confidence:** LOW
- **Likely functional:** Deprecated — "unlock history" suggests removed paywall feature
- **Expected behavior:** May return 404 or 410; history access may now be handled by subscription system
- **Dependencies:** Valid JWT, valid `uid` and `pid`, possibly premium tier
- **Why NOT_TESTED:** Ambiguous naming, may be restructured
- **Risk:** HIGH — likely deprecated or restructured

---

### 2.9 Dynamic but Broken

#### 71. POST /api/public/stream-count
- **Path:** `/api/public/stream-count`
- **Method:** POST
- **Status:** BROKEN (confirmed 404 via dynamic testing)
- **Confidence:** HIGH that it is broken
- **Likely functional:** No — returns 404, indicating route removed or restructured
- **Expected behavior:** Was likely a public endpoint returning live stream viewer counts
- **Dependencies:** None (public endpoint)
- **Why BROKEN:** Route no longer exists on the server; may have been moved or removed
- **Risk:** N/A — confirmed non-functional

---

## 3. Risk Assessment

### High Risk (Likely Deprecated or Broken)
| # | Endpoint | Risk | Reason |
|---|---|---|---|
| 82 | POST ws.arkzynco.com/api/followers/{accountId} | HIGH | Legacy domain, likely deprecated |
| 83 | POST ws.arkzynco.com/api/profile/login | HIGH | Legacy domain, likely deprecated |
| 84 | POST ws.arkzynco.com/api/profile/code | HIGH | Legacy domain, likely deprecated |
| 87 | POST ws.arkzynco.com/api/followers/{uid} | HIGH | Undocumented legacy, likely dead |
| 88 | POST /api/v2/followers/{uid}/{pid} | HIGH | "Unlock history" likely restructured |
| 71 | POST /api/public/stream-count | HIGH | Confirmed 404 |
| 50 | DELETE /api/v2/records/{accountId}/all | HIGH | Mass destructive, too risky to test |

### Medium Risk (Probably Functional but Uncertain)
| # | Endpoint | Risk | Reason |
|---|---|---|---|
| 15 | POST /api/v2/user/transfer-recordings/{accountId}/{targetId} | MEDIUM | Niche feature, may require special permissions |
| 27 | GET /api/v2/followers/{accountId}/{pid}/rm | MEDIUM | Ambiguous purpose, may be deprecated |
| 32 | POST /api/v2/followers/{accountId}/{pid}/following/fetch | MEDIUM | May require platform access/ToS compliance |
| 33 | GET /api/v2/followers/{accountId}/{uid}/following | MEDIUM | Depends on `/following/fetch` data |
| 55 | POST /api/v2/records/{accountId}/{recordId}/ups | MEDIUM | Ambiguous naming, may be internal |

### Low Risk (Likely Functional)
| # | Endpoint | Risk | Reason |
|---|---|---|---|
| 4 | POST /api/login-device | LOW | Standard device auth pattern |
| 7 | POST /api/reset-password/{token} | LOW | Core auth flow |
| 17 | POST /api/v2/user/profile/{id} | LOW | Standard CRUD update |
| 18 | DELETE /api/v2/user/profile/{id} | LOW | Standard CRUD delete |
| 19 | PUT /api/v2/user/profile/{id}/logout | LOW | Standard logout |
| 20 | POST /api/v2/user/profile/{id}/mute | LOW | Standard social feature |
| 23 | DELETE /api/v2/followers/{accountId}/{id} | LOW | Standard CRUD delete |
| 24 | PUT /api/v2/followers/{accountId}/{id} | LOW | Standard toggle |
| 25 | GET /api/v2/followers/{accountId}/{pid} | LOW | Standard read |
| 26 | GET /api/v2/followers/{accountId}/{pid}/history | LOW | Standard read |
| 28 | POST /api/v2/followers/{accountId}/{pid}/fetch | LOW | Standard refresh |
| 29 | POST /api/v2/followers/{accountId}/{pid}/validate | LOW | Validation check |
| 30 | POST /api/v2/followers/{accountId}/{pid}/deep-check | LOW | Validation check |
| 34 | GET /api/v2/followers/{accountId}/{uid}/deep-check | LOW | Read-only variant |
| 36 | GET /api/v2/followers/discover/{accountId}/bubbles | LOW | Discovery feature |
| 40 | GET /api/v2/followers/similar/{accountId}/{pid} | LOW | Social feature |
| 42 | POST /api/v2/followers/{accountId}/{pid}/like | LOW | Standard toggle |
| 44 | POST /api/v2/followers/hide/{accountId}/{pid} | LOW | UI feature |
| 47 | GET /api/v2/records/{accountId}/{recordId} | LOW | Standard read |
| 48 | POST /api/v2/records/new/{accountId} | LOW | Standard create |
| 49 | DELETE /api/v2/records/{accountId}/{recordId} | LOW | Standard delete |
| 52 | POST /api/v2/records/{accountId}/delete-selected | LOW | Bulk delete |
| 53 | POST /api/v2/records/{accountId}/{recordId}/attach | LOW | Metadata attach |
| 54 | PUT /api/v2/records/{accountId}/{recordId}/fix | LOW | Repair operation |
| 56 | POST /api/v2/records/{accountId}/{recordId}/downloaded | LOW | Flag update |
| 58 | GET /api/v2/records/follower/{accountId}/{pid} | LOW | Standard read |
| 65 | POST /api/v2/favorites/{accountId}/{recordId} | LOW | Standard add |
| 66 | DELETE /api/v2/favorites/{accountId}/{recordId} | LOW | Standard remove |
| 9 | POST /api/auth/fcm | LOW | Standard FCM registration |
| 79 | GET /api/slides | LOW | Static content |
| 81 | GET /api/pages/rewards-overview | LOW | Content endpoint |

---

## 4. Dependencies and Prerequisites

### Authentication Chain
```
/api/login-device (4) ──→ JWT Token ──→ All /api/v2/* endpoints
         │
         └── Requires valid device_id (from device registration or platform)

POST /api/reset-password/{token} (7) ──→ Requires reset token from email/SMS flow
         │
         └── Token lifecycle: Request → Email → Click → Token → POST here

POST /api/auth/fcm (9) ──→ Requires JWT + valid FCM token from Firebase SDK
```

### Records CRUD Chain
```
POST /api/v2/records/new/{accountId} (48)
    │
    ├──→ GET /api/v2/records/{accountId}/{recordId} (47)
    │        │
    │        ├──→ POST /api/v2/records/{accountId}/{recordId}/ups (55)
    │        ├──→ POST /api/v2/records/{accountId}/{recordId}/downloaded (56)
    │        ├──→ POST /api/v2/records/{accountId}/{recordId}/attach (53)
    │        ├──→ PUT /api/v2/records/{accountId}/{recordId}/fix (54)
    │        └──→ DELETE /api/v2/records/{accountId}/{recordId} (49)
    │
    ├──→ POST /api/v2/favorites/{accountId}/{recordId} (65)
    │        │
    │        └──→ DELETE /api/v2/favorites/{accountId}/{recordId} (66)
    │
    └──→ POST /api/v2/records/{accountId}/delete-selected (52)
    └──→ DELETE /api/v2/records/{accountId}/all (50)
    └──→ GET /api/v2/records/follower/{accountId}/{pid} (58)
```

### Followers Tracking Chain
```
POST follower creation (dynamic, confirmed)
    │
    ├──→ GET /api/v2/followers/{accountId}/{pid} (25)
    │        │
    │        ├──→ GET /api/v2/followers/{accountId}/{pid}/history (26)
    │        ├──→ GET /api/v2/followers/{accountId}/{pid}/rm (27)
    │        ├──→ POST /api/v2/followers/{accountId}/{pid}/fetch (28)
    │        ├──→ POST /api/v2/followers/{accountId}/{pid}/validate (29)
    │        ├──→ POST /api/v2/followers/{accountId}/{pid}/deep-check (30)
    │        ├──→ POST /api/v2/followers/{accountId}/{pid}/following/fetch (32)
    │        │        │
    │        │        └──→ GET /api/v2/followers/{accountId}/{uid}/following (33)
    │        │
    │        ├──→ GET /api/v2/followers/{accountId}/{uid}/deep-check (34)
    │        ├──→ POST /api/v2/followers/{accountId}/{pid}/like (42)
    │        └──→ POST /api/v2/followers/hide/{accountId}/{pid} (44)
    │
    ├──→ PUT /api/v2/followers/{accountId}/{id} (24)  [toggle status]
    └──→ DELETE /api/v2/followers/{accountId}/{id} (23) [remove]

Social Discovery (independent):
    GET /api/v2/followers/discover/{accountId}/bubbles (36)
    GET /api/v2/followers/similar/{accountId}/{pid} (40)
```

### User Profile Chain
```
POST /api/v2/user/profile/{id} (17) ──→ Update profile
    │
    ├──→ PUT /api/v2/user/profile/{id}/logout (19) ──→ End session
    ├──→ POST /api/v2/user/profile/{id}/mute (20) ──→ Mute toggle
    ├──→ DELETE /api/v2/user/profile/{id} (18) ──→ Delete account [DESTRUCTIVE]
    └──→ POST /api/v2/user/transfer-recordings/{accountId}/{targetId} (15) ──→ Transfer data
```

---

## 5. Recommendations for Dynamic Testing Priorities

### Priority 1: Safe Read-Only Endpoints (test first)
These require only a valid JWT and existing data, with no side effects:

| # | Endpoint | Method | Why |
|---|---|---|---|
| 79 | GET /api/slides | GET | No auth required, immediate validation |
| 81 | GET /api/pages/rewards-overview | GET | May be public, quick check |
| 47 | GET /api/v2/records/{accountId}/{recordId} | GET | Core read, validates record access |
| 58 | GET /api/v2/records/follower/{accountId}/{pid} | GET | Validates follower-record relationship |
| 25 | GET /api/v2/followers/{accountId}/{pid} | GET | Core read, validates follower access |
| 26 | GET /api/v2/followers/{accountId}/{pid}/history | GET | Validates history data exists |
| 34 | GET /api/v2/followers/{accountId}/{uid}/deep-check | GET | Read-only deep check |
| 36 | GET /api/v2/followers/discover/{accountId}/bubbles | GET | Discovery feature validation |
| 40 | GET /api/v2/followers/similar/{accountId}/{pid} | GET | Similar profiles validation |

### Priority 2: Safe Write Endpoints (test with test account)
These are non-destructive or easily reversible:

| # | Endpoint | Method | Why |
|---|---|---|---|
| 9 | POST /api/auth/fcm | POST | Standard FCM registration |
| 42 | POST /api/v2/followers/{accountId}/{pid}/like | POST | Toggle, easily reversed |
| 44 | POST /api/v2/followers/hide/{accountId}/{pid} | POST | Toggle, easily reversed |
| 24 | PUT /api/v2/followers/{accountId}/{id} | PUT | Toggle status, easily reversed |
| 28 | POST /api/v2/followers/{accountId}/{pid}/fetch | POST | Refresh, non-destructive |
| 29 | POST /api/v2/followers/{accountId}/{pid}/validate | POST | Validation, no data change |
| 56 | POST /api/v2/records/{accountId}/{recordId}/downloaded | POST | Flag update, easily reversed |
| 65 | POST /api/v2/favorites/{accountId}/{recordId} | POST | Add favorite, easily removed |

### Priority 3: Auth Endpoints (test with caution)
| # | Endpoint | Method | Why |
|---|---|---|---|
| 4 | POST /api/login-device | POST | Requires valid device_id |
| 7 | POST /api/reset-password/{token} | POST | Requires reset token from email flow |
| 19 | PUT /api/v2/user/profile/{id}/logout | PUT | Ends session, may need re-login |

### Priority 4: Destructive Endpoints (test last, with extreme caution)
| # | Endpoint | Method | Why |
|---|---|---|---|
| 49 | DELETE /api/v2/records/{accountId}/{recordId} | DELETE | Deletes recording |
| 23 | DELETE /api/v2/followers/{accountId}/{id} | DELETE | Removes follower |
| 52 | POST /api/v2/records/{accountId}/delete-selected | POST | Bulk delete |
| 66 | DELETE /api/v2/favorites/{accountId}/{recordId} | DELETE | Removes favorite |
| 18 | DELETE /api/v2/user/profile/{id} | DELETE | Deletes entire account |
| 50 | DELETE /api/v2/records/{accountId}/all | DELETE | Mass delete all recordings |

### Priority 5: Skip (legacy/deprecated)
| # | Endpoint | Method | Why |
|---|---|---|---|
| 82 | POST ws.arkzynco.com/api/followers/{accountId} | POST | Legacy domain |
| 83 | POST ws.arkzynco.com/api/profile/login | POST | Legacy domain |
| 84 | POST ws.arkzynco.com/api/profile/code | POST | Legacy domain |
| 87 | POST ws.arkzynco.com/api/followers/{uid} | POST | Undocumented legacy |
| 88 | POST /api/v2/followers/{uid}/{pid} | POST | Likely restructured |

---

## 6. Legacy Endpoint Analysis

### Domain: `ws.arkzynco.com`

The legacy endpoints (#82, #83, #84, #87) all reside on a different domain (`ws.arkzynco.com`) compared to the current API. This indicates:

1. **Separate infrastructure** — likely the original backend before migration to the current domain
2. **Different auth system** — may use different JWT signing keys, token formats, or session management
3. **Potential deprecation** — the existence of v2 endpoints on the current domain suggests these were superseded

#### Endpoint 82: POST ws.arkzynco.com/api/followers/{accountId}
- **Purpose:** Legacy follower management (create/update)
- **Status:** Likely deprecated — replaced by v2 followers endpoints
- **Evidence:** v2 equivalents exist (#23-34) on current domain

#### Endpoint 83: POST ws.arkzynco.com/api/profile/login
- **Purpose:** Legacy authentication
- **Status:** Likely deprecated — replaced by `/api/login-device` and standard auth
- **Evidence:** Current auth flow uses different endpoint structure

#### Endpoint 84: POST ws.arkzynco.com/api/profile/code
- **Purpose:** Legacy verification code (possibly 2FA or email verification)
- **Status:** Likely deprecated — verification flow may now be handled differently
- **Evidence:** No v2 equivalent found, suggesting feature was removed or restructured

#### Endpoint 87: POST ws.arkzynco.com/api/followers/{uid}
- **Purpose:** Undocumented legacy follower endpoint
- **Status:** Unknown — may be identical to #82 or a variant
- **Evidence:** No documentation, no v2 equivalent with same structure
- **Note:** Uses `{uid}` parameter which differs from `{accountId}` pattern in v2

#### Endpoint 88: POST /api/v2/followers/{uid}/{pid}
- **Purpose:** "Unlock history" — likely a premium/paywall feature
- **Status:** Likely deprecated — "unlock" implies monetization that may have been restructured
- **Evidence:** History access may now be part of subscription tiers rather than per-profile unlocks
- **Note:** Despite being on v2 domain, uses `{uid}` instead of `{accountId}`, suggesting older origin

### Recommendation for Legacy Endpoints
- **Do not prioritize testing** unless legacy credentials are available
- If testing is desired, attempt with `curl` to check for:
  - `404 Not Found` — route completely removed
  - `410 Gone` — explicitly deprecated
  - `301/302 Redirect` — migrated to new endpoint
  - `401/403` — auth system changed
- The legacy domain may still be operational for existing users who haven't migrated
- Consider monitoring for any traffic to these endpoints as an indicator of legacy client usage

---

## 7. Conclusion

Of the 45 static-only endpoints:

- **31 are likely functional** (LOW risk) — standard CRUD and social features
- **5 are medium risk** — niche features or ambiguous naming
- **7 are high risk** — legacy/deprecated or confirmed broken
- **2 are authentication-dependent** — require specific tokens or credentials

The primary barrier to dynamic testing is **authentication** — most endpoints require a valid JWT token, which requires either a real account or the ability to create test accounts. The secondary barrier is **data dependencies** — many endpoints need specific accountId, recordId, and profileId values that only exist after using the app.

**Recommended testing approach:**
1. Obtain or create a test account with valid credentials
2. Start with Priority 1 (read-only) endpoints to validate basic access
3. Progress to Priority 2 (safe writes) to test non-destructive operations
4. Avoid Priority 5 (legacy) unless specifically investigating migration status
5. Document all response codes and payloads for future reference
