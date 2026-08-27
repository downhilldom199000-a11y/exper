# TKREC API Schemas (Complete)

> Generated: 2026-08-26
> Source: API_MASTER.json (88 endpoints) + JS source analysis
> Evidence markers: OBSERVED | CODE_REFERENCED | INFERRED

---

## Table of Contents

1. [API Infrastructure](#1-api-infrastructure)
2. [Common Error Responses](#2-common-error-responses)
3. [Authentication (IDs 1-10, 73)](#3-authentication)
4. [User Management (IDs 11-15)](#4-user-management)
5. [User Profiles (IDs 16-20, 86)](#5-user-profiles)
6. [Followers Tracking (IDs 21-34)](#6-followers-tracking)
7. [Followers Social (IDs 35-45)](#7-followers-social)
8. [Records / Recordings (IDs 46-58)](#8-records--recordings)
9. [Broadcasts / Streams (IDs 59-63)](#9-broadcasts--streams)
10. [Favorites (IDs 64-66)](#10-favorites)
11. [Purchases (IDs 67-69)](#11-purchases)
12. [Public Endpoints (IDs 70-72)](#12-public-endpoints)
13. [Miscellaneous (IDs 74-81, 85)](#13-miscellaneous)
14. [Legacy / Undocumented (IDs 82-84, 87-88)](#14-legacy--undocumented)
15. [FCM Event Types](#15-fcm-event-types)
16. [Pagination Schema](#16-pagination-schema)
17. [Video Quality Levels](#17-video-quality-levels)
18. [EDT Token Format](#18-edt-token-format)
19. [Download Flow](#19-download-flow)
20. [Ad Integration](#20-ad-integration)

---

## 1. API Infrastructure

### Axios Instances

| Instance | Base URL | Auth | CSRF | Usage |
|----------|----------|------|------|-------|
| `u` | `https://raid.arkzynco.com` | JWT Bearer | XSRF-TOKEN cookie | **Main client** — all authenticated API calls `[CODE_REFERENCED]` |
| `Bg` | `https://ws.arkzynco.com` | None | None | Public endpoints, follower discovery `[CODE_REFERENCED]` |
| `xe` | `https://ws.arkzynco.com` | JWT Bearer | None | Secondary auth client `[CODE_REFERENCED]` |

### Request Headers (applied to `u` instance)

| Header | Value | Evidence |
|--------|-------|----------|
| `Authorization` | `Bearer {jwt_token}` | `CODE_REFERENCED` |
| `X-XSRF-TOKEN` | `{cookie value}` | `CODE_REFERENCED` |
| `X-App-Version` | `1.3.0` | `OBSERVED` |
| `X-Requested-With` | `XMLHttpRequest` | `OBSERVED` |
| `Content-Type` | `application/json` | `CODE_REFERENCED` |

### Retry Configuration

| Field | Value | Evidence |
|-------|-------|----------|
| Retries | `1` | `CODE_REFERENCED` |
| Delay | Exponential backoff | `CODE_REFERENCED` |
| Retry Condition | Network error / 5xx | `INFERRED` |

### Response Interceptor

| Behavior | Detail | Evidence |
|----------|--------|----------|
| Auto-logout | On `401` response | `CODE_REFERENCED` |
| Exclusion | `/api/v2/logout` endpoint skipped | `CODE_REFERENCED` |
| Logout action | Clear token, redirect to login | `CODE_REFERENCED` |

### Backend

| Field | Value | Evidence |
|-------|-------|----------|
| Runtime | Express.js | `INFERRED` |
| CDN/Proxy | Cloudflare | `OBSERVED` |
| CORS Origin | `https://app.tkrec.com` | `OBSERVED` |

---

## 2. Common Error Responses

> All error objects share a single-field `{ message: string }` shape.

| HTTP Status | Message | Trigger | Evidence |
|-------------|---------|---------|----------|
| `401` | `"Invalid email or password."` | Bad credentials on `/api/signin` | `OBSERVED` |
| `401` | `"Unauthenticated."` | Missing/invalid JWT on protected route | `OBSERVED` |
| `401` | `"session_expired"` | JWT expiry or server-side session invalidation | `OBSERVED` |
| `401` | `"logged-out"` | Forced logout (concurrent session, admin action) | `OBSERVED` |
| `403` | `"sentry-block"` | Cloudflare/WAF bot detection | `OBSERVED` |
| `403` | `"challenge"` | Cloudflare JS challenge required | `OBSERVED` |
| `429` | `"Too Many Requests"` | Rate limit exceeded | `OBSERVED` |

---

## 3. Authentication

### ID 1 — Sign In

```
POST /api/signin
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `email` | `string` | body | Y | `OBSERVED` |
| `password` | `string` | body | Y | `OBSERVED` |
| `user` | `object` | response 200 | — | `OBSERVED` |
| `user.uid` | `string` | response 200 | — | `OBSERVED` |
| `user.name` | `string` | response 200 | — | `OBSERVED` |
| `user.email` | `string` | response 200 | — | `OBSERVED` |
| `token` | `string` | response 200 | — | `OBSERVED` |
| `message` | `string` | response 401 | — | `OBSERVED` |

**Instance:** `u` (raid.arkzynco.com) `[CODE_REFERENCED]`

---

### ID 2 — Social Login

```
POST /api/login
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `provider` | `string` | body | Y | `OBSERVED` |
| `token` | `string` | body | Y | `OBSERVED` |
| `email` | `string` | body | N | `OBSERVED` |
| `user` | `object` | response 200 | — | `OBSERVED` |
| `token` | `string` | response 200 | — | `OBSERVED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 3 — Register

```
POST /api/register
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `email` | `string` | body | Y | `OBSERVED` |
| `password` | `string` | body | Y | `OBSERVED` |
| `name` | `string` | body | N | `OBSERVED` |
| `user` | `object` | response 200 | — | `OBSERVED` |
| `token` | `string` | response 200 | — | `OBSERVED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 4 — Login Device

```
POST /api/login-device
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `device_id` | `string` | body | Y | `CODE_REFERENCED` |
| `user` | `object` | response 200 | — | `INFERRED` |
| `token` | `string` | response 200 | — | `INFERRED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 5 — Confirm Email

```
POST /api/confirm-email
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `token` | `string` | body | Y | `CODE_REFERENCED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 6 — Forgot Password

```
POST /api/forgot-password
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `email` | `string` | body | Y | `CODE_REFERENCED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 7 — Reset Password

```
POST /api/reset-password/{token}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `token` | `string` | URL param | Y | `CODE_REFERENCED` |
| `password` | `string` | body | Y | `CODE_REFERENCED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 8 — Get CSRF Token

```
GET /api/csrf
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(none)* | — | — | — | Sets `XSRF-TOKEN` cookie in response |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 9 — Register FCM Token

```
POST /api/auth/fcm
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `token` | `string` | body | Y | `CODE_REFERENCED` |
| `platform` | `string` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 10 — Device Status

```
POST /api/status
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `client` | `string` | body | Y | `CODE_REFERENCED` |
| `token` | `string` | body | Y | `CODE_REFERENCED` |
| `device_id` | `string` | body | N | `CODE_REFERENCED` |
| `os_version` | `string` | body | N | `CODE_REFERENCED` |
| `model` | `string` | body | N | `CODE_REFERENCED` |
| `platform` | `string` | body | N | `CODE_REFERENCED` |
| `app_version` | `string` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer + `pl()` helper `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 73 — Logout (v2)

```
POST /api/v2/logout
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(none)* | — | — | — | No body required |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`
**Note:** Excluded from the auto-logout interceptor on 401 `[CODE_REFERENCED]`

---

## 4. User Management

### ID 11 — Get Current User

```
GET /api/v2/user
```

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `data.uid` | `string` | `OBSERVED` |
| `data.name` | `string` | `OBSERVED` |
| `data.email` | `string` | `OBSERVED` |
| `data.subscription` | `object` | `OBSERVED` |
| `data.profiles` | `array<object>` | `OBSERVED` |
| `data.settings` | `object` | `OBSERVED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 12 — Update User

```
PUT /api/v2/user
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `name` | `string` | body | N | `CODE_REFERENCED` |
| `email` | `string` | body | N | `CODE_REFERENCED` |
| `settings` | `object` | body | N | `CODE_REFERENCED` |
| `settings.default_quality` | `string` | body | N | `CODE_REFERENCED` |
| `settings.instant_download` | `boolean` | body | N | `CODE_REFERENCED` |
| `settings.notifications_enabled` | `boolean` | body | N | `CODE_REFERENCED` |
| `settings.dark_theme` | `boolean` | body | N | `CODE_REFERENCED` |
| `settings.locale` | `string` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 13 — Delete User

```
DELETE /api/v2/user
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(none)* | — | — | — | No body required |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 14 — Record Installation

```
POST /api/v2/user/installed
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `referrer` | `string` | body | N | `CODE_REFERENCED` |
| `platform` | `string` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 15 — Transfer Recordings

```
POST /api/v2/user/transfer-recordings/{accountId}/{targetId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `targetId` | `string` | URL param | Y | `CODE_REFERENCED` |
| *(none)* | — | body | — | No body required |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 5. User Profiles

### Shared Object — Profile

```json
{
  "id":                    1,
  "uid":                   "7012345678",
  "username":              "charlidamelio",
  "nickname":              "Charli D'Amelio",
  "pic":                   "https://...",
  "video_count":           150,
  "storage":               5368709120,
  "deletable_count":       50,
  "validating":            false,
  "validation_progress":   0,
  "pending_validation":    0,
  "available_sync":        25,
  "available_streams":     25,
  "reactivated_at":        "2026-08-25T10:00:00Z",
  "last_live":             "2026-08-26T08:00:00Z",
  "live_now":              false,
  "real":                  true,
  "status":                "active",
  "follow_type":           "follower|following",
  "total_live":            120,
  "total_views":           125000,
  "total_followers":       150000000
}
```

> All fields `CODE_REFERENCED` from `index-1.3.0.js`

| Field | Type | Description | Evidence |
|-------|------|-------------|----------|
| `id` | `integer` | Internal profile ID | `CODE_REFERENCED` |
| `uid` | `string` | TikTok UID | `CODE_REFERENCED` |
| `username` | `string` | Current TikTok username | `CODE_REFERENCED` |
| `nickname` | `string` | Display name | `CODE_REFERENCED` |
| `pic` | `string` | Profile picture URL | `CODE_REFERENCED` |
| `video_count` | `integer` | Total videos on profile | `CODE_REFERENCED` |
| `storage` | `integer` | Storage used (bytes) | `CODE_REFERENCED` |
| `deletable_count` | `integer` | Videos eligible for deletion | `CODE_REFERENCED` |
| `validating` | `boolean` | Currently running validation | `CODE_REFERENCED` |
| `validation_progress` | `integer` | Validation progress 0-100 | `CODE_REFERENCED` |
| `pending_validation` | `integer` | Profiles queued for validation | `CODE_REFERENCED` |
| `available_sync` | `integer` | Remaining sync slots | `CODE_REFERENCED` |
| `available_streams` | `integer` | Remaining stream slots | `CODE_REFERENCED` |
| `reactivated_at` | `string (ISO 8601)` | Last reactivation timestamp | `CODE_REFERENCED` |
| `last_live` | `string (ISO 8601)` | Last live stream timestamp | `CODE_REFERENCED` |
| `live_now` | `boolean` | Currently live | `CODE_REFERENCED` |
| `real` | `boolean` | Real vs. test profile | `CODE_REFERENCED` |
| `status` | `string` | `"active"` / `"inactive"` / `"frozen"` | `CODE_REFERENCED` |
| `follow_type` | `string` | `"follower"` or `"following"` | `CODE_REFERENCED` |
| `total_live` | `integer` | Total recorded live streams | `CODE_REFERENCED` |
| `total_views` | `integer` | Total views across streams | `CODE_REFERENCED` |
| `total_followers` | `integer` | Current follower count | `CODE_REFERENCED` |

---

### ID 86 — Get Profile List (alias)

```
GET /api/v2/user/profile
```

Returns array of profile objects for the authenticated user.

**Response 200:** `{ data: [ProfileObject, ...] }`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 16 — Create Profile

```
POST /api/v2/user/profile
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `name` | `string` | body | Y | `CODE_REFERENCED` |
| `email` | `string` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 17 — Update Profile

```
POST /api/v2/user/profile/{id}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `id` | `integer` | URL param | Y | `CODE_REFERENCED` |
| `name` | `string` | body | N | `CODE_REFERENCED` |
| `email` | `string` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 18 — Delete Profile

```
DELETE /api/v2/user/profile/{id}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `id` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 19 — Logout Profile

```
PUT /api/v2/user/profile/{id}/logout
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `id` | `integer` | URL param | Y | `CODE_REFERENCED` |
| *(spread)* | `ProfileObject` | body | Y | `CODE_REFERENCED` — full profile object is spread into body |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 20 — Toggle Mute Profile

```
POST /api/v2/user/profile/{id}/mute
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `id` | `integer` | URL param | Y | `CODE_REFERENCED` |
| *(none)* | — | body | — | Toggle operation, no body required |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 6. Followers Tracking

### ID 21 — List Followers

```
GET /api/v2/followers/{accountId}/list
```

**Query Parameters:**

| Param | Type | Required | Description | Evidence |
|-------|------|----------|-------------|----------|
| `page` | `integer` | N | Page number (default: 1) | `CODE_REFERENCED` |
| `limit` | `integer` | N | Items per page | `CODE_REFERENCED` |
| `sort` | `string` | N | Sort field | `CODE_REFERENCED` |
| `order` | `string` | N | `"asc"` / `"desc"` | `CODE_REFERENCED` |
| `q` | `string` | N | Search query | `CODE_REFERENCED` |
| `a` | `string` | N | Filter (active/inactive/frozen) | `CODE_REFERENCED` |

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `data` | `array<ProfileObject>` | `CODE_REFERENCED` |
| `totals.active` | `integer` | `CODE_REFERENCED` |
| `totals.inactive` | `integer` | `CODE_REFERENCED` |
| `totals.frozen` | `integer` | `CODE_REFERENCED` |
| `meta.current_page` | `integer` | `CODE_REFERENCED` |
| `meta.last_page` | `integer` | `CODE_REFERENCED` |
| `meta.per_page` | `integer` | `CODE_REFERENCED` |
| `meta.total` | `integer` | `CODE_REFERENCED` |
| `rc` | `integer` | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 22 — Add Follower

```
POST /api/v2/followers/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | body | Y | `OBSERVED` |
| `fetch` | `boolean` | body | N | `CODE_REFERENCED` — triggers immediate data fetch |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 23 — Remove Follower

```
DELETE /api/v2/followers/{accountId}/{id}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `id` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 24 — Toggle Follower Status

```
PUT /api/v2/followers/{accountId}/{id}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `id` | `integer` | URL param | Y | `CODE_REFERENCED` |
| *(none)* | — | body | — | Toggle, no body required |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 25 — Get Single Follower Detail

```
GET /api/v2/followers/{accountId}/{pid}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Response 200:** `ProfileObject`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 26 — Follower Username History

```
GET /api/v2/followers/{accountId}/{pid}/history
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Response 200:** Array of past usernames `[CODE_REFERENCED]`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 27 — Follower Removal Info

```
GET /api/v2/followers/{accountId}/{pid}/rm
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Response 200:** Removal/recommendation info `[INFERRED]`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 28 — Force Refresh Follower

```
POST /api/v2/followers/{accountId}/{pid}/fetch
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 29 — Validate Follower

```
POST /api/v2/followers/{accountId}/{pid}/validate
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `confirm_after_check` | `boolean` | body | N | `CODE_REFERENCED` |
| `rooms_to_validate` | `array<integer>` | body | N | `CODE_REFERENCED` |
| `force_revalidate` | `boolean` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 30 — Deep Check (POST)

```
POST /api/v2/followers/{accountId}/{pid}/deep-check
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 31 — Report Follower (individual)

```
POST /api/v2/followers/{accountId}/{pid}/report
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 32 — Fetch Following (POST)

```
POST /api/v2/followers/{accountId}/{pid}/following/fetch
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `refetch` | `boolean` | body | N | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 33 — Get Following (paginated)

```
GET /api/v2/followers/{accountId}/{uid}/following
```

**Query Parameters:** Standard pagination (page, limit, sort, order) `[CODE_REFERENCED]`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 34 — Deep Check (GET)

```
GET /api/v2/followers/{accountId}/{uid}/deep-check
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `uid` | `string` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 7. Followers Social

### ID 35 — Discover Followers

```
GET /api/v2/followers/discover/{accountId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 36 — Discover Bubbles

```
GET /api/v2/followers/discover/{accountId}/bubbles
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 37 — Explore Lives

```
GET /api/v2/followers/explore/lives/{accountId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 38 — Ranked Followers

```
GET /api/v2/followers/ranked/{accountId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 39 — Search Followers

```
GET /api/v2/followers/search/{accountId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 40 — Similar Profiles

```
GET /api/v2/followers/similar/{accountId}/{pid}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 41 — Suggested Followers

```
GET /api/v2/followers/suggest/{accountId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 42 — Like/Unlike Follower

```
POST /api/v2/followers/{accountId}/{pid}/like
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `status` | `integer (0\|1)` | body | Y | `CODE_REFERENCED` — `0` = unlike, `1` = like |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 43 — Get Liked Followers

```
GET /api/v2/followers/{accountId}/likes
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 44 — Hide Follower

```
POST /api/v2/followers/hide/{accountId}/{pid}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `accountId` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 45 — Report Follower (batch)

```
POST /api/v2/followers/{accountId}/report
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `page` | `string` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 8. Records / Recordings

### Shared Object — Record

```json
{
  "id":           12345,
  "status":       "done|pending|recording|processing",
  "thumb":        "https://p16-common-sign.tiktokcdn-us.com/...",
  "pic":          "https://...",
  "username":     "charlidamelio",
  "uid":          "7012345678",
  "nickname":     "Charli D'Amelio",
  "stream_id":    "abc123",
  "videos":       [
    { "q": "216p", "size": 5242880,  "quality": "SD" },
    { "q": "360p", "size": 10485760, "quality": "SD" },
    { "q": "480p", "size": 20971520, "quality": "SD" },
    { "q": "720p", "size": 52428800, "quality": "HD" }
  ],
  "favorites":    5,
  "views":        120,
  "downloaded":   null,
  "cover":        "https://...",
  "rebuild":      {
    "status":      "none|pending|processing|completed|failed",
    "can_rebuild": false,
    "rebuilt_at":  null
  },
  "created_at":   1693000000,
  "updated_at":   1693000000,
  "duration":     3600,
  "size":         52428800,
  "real":         true
}
```

> All fields `CODE_REFERENCED` from `index-1.3.0.js`

| Field | Type | Description | Evidence |
|-------|------|-------------|----------|
| `id` | `integer` | Internal record ID | `CODE_REFERENCED` |
| `status` | `string` | `"done"` / `"pending"` / `"recording"` / `"processing"` | `CODE_REFERENCED` |
| `thumb` | `string` | Thumbnail URL (TikTok CDN) | `CODE_REFERENCED` |
| `pic` | `string` | Cover image URL | `CODE_REFERENCED` |
| `username` | `string` | TikTok username at recording time | `CODE_REFERENCED` |
| `uid` | `string` | TikTok UID | `CODE_REFERENCED` |
| `nickname` | `string` | Display name | `CODE_REFERENCED` |
| `stream_id` | `string` | Unique stream identifier | `CODE_REFERENCED` |
| `videos` | `array<object>` | Available quality variants | `CODE_REFERENCED` |
| `videos[].q` | `string` | Resolution label (`"216p"`, `"360p"`, `"480p"`, `"720p"`) | `CODE_REFERENCED` |
| `videos[].size` | `integer` | File size in bytes | `CODE_REFERENCED` |
| `videos[].quality` | `string` | `"SD"` or `"HD"` | `CODE_REFERENCED` |
| `favorites` | `integer` | Number of favorites | `CODE_REFERENCED` |
| `views` | `integer` | View count | `CODE_REFERENCED` |
| `downloaded` | `null\|boolean` | Download status (`null` = not downloaded) | `CODE_REFERENCED` |
| `cover` | `string` | Cover image URL | `CODE_REFERENCED` |
| `rebuild` | `object` | Video rebuild status | `CODE_REFERENCED` |
| `rebuild.status` | `string` | `"none"` / `"pending"` / `"processing"` / `"completed"` / `"failed"` | `CODE_REFERENCED` |
| `rebuild.can_rebuild` | `boolean` | Whether rebuild is available | `CODE_REFERENCED` |
| `rebuild.rebuilt_at` | `null\|string` | ISO 8601 rebuild timestamp | `CODE_REFERENCED` |
| `created_at` | `integer` | Unix timestamp (seconds) | `CODE_REFERENCED` |
| `updated_at` | `integer` | Unix timestamp (seconds) | `CODE_REFERENCED` |
| `duration` | `integer` | Duration in seconds | `CODE_REFERENCED` |
| `size` | `integer` | Total size in bytes | `CODE_REFERENCED` |
| `real` | `boolean` | Real vs. test record | `CODE_REFERENCED` |

---

### ID 46 — List Records

```
GET /api/v2/records/{accountId}
```

**Query Parameters:**

| Param | Type | Required | Description | Evidence |
|-------|------|----------|-------------|----------|
| `page` | `integer` | N | Page number | `CODE_REFERENCED` |
| `limit` | `integer` | N | Items per page | `CODE_REFERENCED` |
| `sort` | `string` | N | Sort field | `CODE_REFERENCED` |
| `order` | `string` | N | `"asc"` / `"desc"` | `CODE_REFERENCED` |
| `dateRange` | `string` | N | Date range filter | `CODE_REFERENCED` |
| `status` | `string` | N | Status filter | `CODE_REFERENCED` |

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `data` | `array<RecordObject>` | `CODE_REFERENCED` |
| `meta` | `PaginationMeta` | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 47 — Get Single Record

```
GET /api/v2/records/{accountId}/{recordId}
```

**Response 200:** `RecordObject`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 48 — Create Record

```
POST /api/v2/records/new/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `data` | `object` | body | Y | `CODE_REFERENCED` — record config (username, quality, etc.) |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 49 — Delete Single Record

```
DELETE /api/v2/records/{accountId}/{recordId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 50 — Delete All Records

```
DELETE /api/v2/records/{accountId}/all
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 51 — Delete Records by Follower

```
DELETE /api/v2/records/follower/{accountId}/{pid}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 52 — Delete Selected Records

```
POST /api/v2/records/{accountId}/delete-selected
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `ids` | `array<string>` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 53 — Attach Record

```
POST /api/v2/records/{accountId}/{recordId}/attach
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 54 — Fix / Rebuild Record

```
PUT /api/v2/records/{accountId}/{recordId}/fix
```

**Side Effect:** Triggers `video_rebuilt` FCM notification on completion `[CODE_REFERENCED]`

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 55 — Get Download URL (UPS)

```
POST /api/v2/records/{accountId}/{recordId}/ups
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `edt` | `string (base64)` | body | Y | `CODE_REFERENCED` — see [EDT Token Format](#18-edt-token-format) |
| `quality` | `string` | body | Y | `CODE_REFERENCED` |

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `video` | `string` | `CODE_REFERENCED` — CDN URL for download |
| `record` | `RecordObject` | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 56 — Mark Downloaded

```
POST /api/v2/records/{accountId}/{recordId}/downloaded
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `status` | `boolean` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 57 — Storage Usage

```
GET /api/v2/records/{accountId}/storage-usage
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 58 — Records by Follower

```
GET /api/v2/records/follower/{accountId}/{pid}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 9. Broadcasts / Streams

### ID 59 — Create Broadcast

```
POST /api/v2/broadcasts/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `data` | `object` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 60 — Record Now (Legacy v1)

```
POST /api/streams/record-now/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 61 — Rebuild Stream (Legacy v1)

```
POST /api/streams/rebuild/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `uuid` | `string` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 62 — Record Profile (v2)

```
POST /api/v2/record-profile/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | body | Y | `CODE_REFERENCED` |
| *(additional params)* | `object` | body | N | `CODE_REFERENCED` — quality, duration, etc. |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 63 — Record Profile (Native Java)

```
POST /api/v2/record-profile/{username}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | URL param | Y | `CODE_REFERENCED` |

**Source:** `ShareReceiverActivity.java` (Android native) `[CODE_REFERENCED]`
**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 10. Favorites

### ID 64 — List Favorites

```
GET /api/v2/favorites/{accountId}/list
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 65 — Add Favorite

```
POST /api/v2/favorites/{accountId}/{recordId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `recordId` | `integer` | URL param | Y | `CODE_REFERENCED` |
| *(none)* | — | body | — | No body required |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 66 — Remove Favorite

```
DELETE /api/v2/favorites/{accountId}/{recordId}
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 11. Purchases

### ID 67 — Purchase Storage

```
POST /api/v2/purchase/storage
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(receipt)* | `string` | body | Y | `CODE_REFERENCED` — RevenueCat IAP receipt |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 68 — Purchase Slots

```
POST /api/v2/purchase/slots
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(receipt)* | `string` | body | Y | `CODE_REFERENCED` — RevenueCat IAP receipt |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 69 — Purchase History

```
GET /api/v2/purchase/history-list
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 12. Public Endpoints

> These endpoints use the `Bg` instance (ws.arkzynco.com, no auth).

### ID 70 — Public History Fetch

```
POST ws.arkzynco.com/api/public/history/fetch
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | body | Y | `CODE_REFERENCED` |

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `status` | `string` | `CODE_REFERENCED` — `"success"` |
| `data` | `array<object>` | `CODE_REFERENCED` |
| `data[].thumb` | `string` | `CODE_REFERENCED` |
| `data[].count` | `integer` | `CODE_REFERENCED` |
| `data[].username` | `string` | `CODE_REFERENCED` |
| `data[].uid` | `string` | `CODE_REFERENCED` |
| `data[].nickname` | `string` | `CODE_REFERENCED` |

**Instance:** `Bg` `[CODE_REFERENCED]`

---

### ID 71 — Stream Count (unimplemented)

```
POST /api/public/stream-count
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | body | Y | `CODE_REFERENCED` |

**Response:** `404` (not implemented) `[OBSERVED]`

**Instance:** `Bg` `[CODE_REFERENCED]`

---

### ID 72 — Auth History Fetch

```
POST ws.arkzynco.com/api/history/fetch
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `username` | `string` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `xe` `[CODE_REFERENCED]`

---

## 13. Miscellaneous

### ID 74 — Network Log

```
POST /api/v2/network-log
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(error details)* | `object` | body | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 75 — News

```
GET /api/v2/news
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 76 — Promotions

```
GET /api/v2/promotions
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 77 — Flip Game State

```
GET /api/v2/flip-game/state
```

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `canFlip` | `boolean` | `CODE_REFERENCED` |
| `flipBalance` | `integer` | `CODE_REFERENCED` |
| `dailyBonusFlip` | `boolean` | `CODE_REFERENCED` |
| `isJackpotReady` | `boolean` | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 78 — Flip Game Play

```
POST /api/v2/flip-game/play
```

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 79 — Slides

```
GET /api/slides
```

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `data` | `array<object>` | `CODE_REFERENCED` |
| `data[].id` | `integer` | `CODE_REFERENCED` |
| `data[].img` | `string` | `CODE_REFERENCED` |
| `data[].url` | `string` | `CODE_REFERENCED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 80 — Page Content

```
GET /api/pages/{pageUrl}
```

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `data.content` | `string` | `CODE_REFERENCED` — raw HTML |
| `data.title` | `string` | `CODE_REFERENCED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 81 — Rewards Overview

```
GET /api/pages/rewards-overview
```

**Response 200:**

| Field | Type | Evidence |
|-------|------|----------|
| `data.content` | `string` | `CODE_REFERENCED` — raw HTML |
| `data.title` | `string` | `CODE_REFERENCED` |

**Instance:** `u` `[CODE_REFERENCED]`

---

### ID 85 — FAQ (alias)

```
GET /api/pages/faq
```

Alias for page content endpoint. Returns FAQ HTML content. `[CODE_REFERENCED]`

**Instance:** `u` `[CODE_REFERENCED]`

---

## 14. Legacy / Undocumented

> These endpoints exist in traffic or JS source but are not part of the primary API surface.

### ID 82 — Legacy Follower Add

```
POST ws.arkzynco.com/api/followers/{accountId}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(follower data)* | `object` | body | Y | `INFERRED` |

**Instance:** `Bg` `[CODE_REFERENCED]`

---

### ID 83 — Legacy Profile Login

```
POST ws.arkzynco.com/api/profile/login
```

**Conditional Route:** `/api/profile/login/{id}` — appends profile ID when targeting specific profile `[CODE_REFERENCED]`

**Instance:** `Bg` `[CODE_REFERENCED]`

---

### ID 84 — Legacy Verification Code

```
POST ws.arkzynco.com/api/profile/code
```

**Conditional Route:** `/api/profile/code/{id}` — appends profile ID when targeting specific profile `[CODE_REFERENCED]`

**Instance:** `Bg` `[CODE_REFERENCED]`

---

### ID 87 — Undocumented Follower Add (duplicate)

```
POST ws.arkzynco.com/api/followers/{uid}
```

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| *(follower data)* | `object` | body | Y | `INFERRED` |

**Note:** Duplicate of ID 82 with `uid` instead of `accountId` `[CODE_REFERENCED]`

**Instance:** `Bg` `[CODE_REFERENCED]`

---

### ID 88 — Unlock History

```
POST /api/v2/followers/{uid}/{pid}
```

**Conditional Route:** Appends `/free` when `isFree=true` `[CODE_REFERENCED]`

| Field | Type | Location | Required | Evidence |
|-------|------|----------|----------|----------|
| `uid` | `string` | URL param | Y | `CODE_REFERENCED` |
| `pid` | `integer` | URL param | Y | `CODE_REFERENCED` |

**Auth:** JWT Bearer `[CODE_REFERENCED]`
**Instance:** `u` `[CODE_REFERENCED]`

---

## 15. FCM Event Types

> All event types `CODE_REFERENCED` from `index-1.3.0.js`

### Recording Events

| Event | Trigger | Evidence |
|-------|---------|----------|
| `recording_start` | Stream recording initiated | `CODE_REFERENCED` |
| `recording_finished` | Stream recording completed successfully | `CODE_REFERENCED` |
| `ext_recording_start` | External/extended recording initiated | `CODE_REFERENCED` |
| `recording_failed` | Stream recording failed | `CODE_REFERENCED` |

### Video Events

| Event | Trigger | Evidence |
|-------|---------|----------|
| `video_ready` | Processed video available for playback | `CODE_REFERENCED` |
| `video_rebuilt` | Video rebuild/fix completed (triggered by ID 54) | `CODE_REFERENCED` |
| `video_rebuild_failed` | Video rebuild failed | `CODE_REFERENCED` |
| `video_fix_completed` | Video fix completed | `CODE_REFERENCED` |
| `reencoding_finished` | Video re-encoding completed | `CODE_REFERENCED` |

### Session Events

| Event | Trigger | Evidence |
|-------|---------|----------|
| `session_expired` | JWT/session expired server-side | `CODE_REFERENCED` |
| `challenge` | Cloudflare challenge required | `CODE_REFERENCED` |
| `logged-out` | Forced logout (concurrent session, admin) | `CODE_REFERENCED` |

### Subscription Events

| Event | Trigger | Evidence |
|-------|---------|----------|
| `slot_expired` | Recording slot expired | `CODE_REFERENCED` |
| `pro_expired` | Pro subscription expired | `CODE_REFERENCED` |
| `storage_expired` | Storage quota expired | `CODE_REFERENCED` |
| `storage_warning` | Storage approaching limit | `CODE_REFERENCED` |

### Game Events

| Event | Trigger | Evidence |
|-------|---------|----------|
| `flip_daily_ready` | Daily bonus flip available | `CODE_REFERENCED` |
| `flip_bonus_expiring` | Bonus flip about to expire | `CODE_REFERENCED` |
| `flip_jackpot_winner` | Jackpot win | `CODE_REFERENCED` |

### Profile / Marketing Events

| Event | Trigger | Evidence |
|-------|---------|----------|
| `updateProfile` | Profile data update notification | `CODE_REFERENCED` |
| `updateRecord` | Record data update notification | `CODE_REFERENCED` |
| `reengagement` | User re-engagement prompt | `CODE_REFERENCED` |

---

## 16. Pagination Schema

### Query Parameters

| Param | Type | Default | Evidence |
|-------|------|---------|----------|
| `page` | `integer` | `1` | `CODE_REFERENCED` |
| `limit` | `integer` | varies | `CODE_REFERENCED` |
| `sort` | `string` | field-dependent | `CODE_REFERENCED` |
| `order` | `string` | `"desc"` | `CODE_REFERENCED` |
| `q` | `string` | — | `CODE_REFERENCED` — search filter |
| `a` | `string` | — | `CODE_REFERENCED` — category filter (active/inactive/frozen) |
| `dateRange` | `string` | — | `CODE_REFERENCED` — date range filter |
| `status` | `string` | — | `CODE_REFERENCED` — status filter |

### Response Envelope

```json
{
  "data": [ ... ],
  "meta": {
    "current_page": 1,
    "last_page": 5,
    "per_page": 20,
    "total": 100
  }
}
```

> All fields `CODE_REFERENCED`

| Field | Type | Description | Evidence |
|-------|------|-------------|----------|
| `data` | `array` | Current page items | `CODE_REFERENCED` |
| `meta.current_page` | `integer` | Active page number | `CODE_REFERENCED` |
| `meta.last_page` | `integer` | Total pages | `CODE_REFERENCED` |
| `meta.per_page` | `integer` | Items per page | `CODE_REFERENCED` |
| `meta.total` | `integer` | Total items | `CODE_REFERENCED` |

---

## 17. Video Quality Levels

> All data `CODE_REFERENCED` from `index-1.3.0.js`

| Quality | Resolution | Codec | Typical Size (1hr) | Tier | Evidence |
|---------|------------|-------|---------------------|------|----------|
| `216p` | 384×216 | H.264 | ~5 MB | SD | `CODE_REFERENCED` |
| `360p` | 640×360 | H.264 | ~10 MB | SD | `CODE_REFERENCED` |
| `480p` | 854×480 | H.264 | ~20 MB | SD | `CODE_REFERENCED` |
| `720p` | 1280×720 | H.264 | ~50 MB | HD | `CODE_REFERENCED` |

> **Note:** Sizes are approximate and depend on bitrate/content. Higher quality = larger file size. Non-subscribers are limited to lower quality tiers; subscribers access all tiers including `720p`. `[INFERRED]`

---

## 18. EDT Token Format

> All data `CODE_REFERENCED` from `index-1.3.0.js`

The EDT (Encrypted Download Token) is a base64-encoded JSON payload used for download URL requests (ID 55).

### Structure

```json
{
  "e": "<event_type>",
  "t": "<unix_timestamp_seconds>"
}
```

### Generation

```javascript
const timestamp = Math.round(new Date().getTime() / 1000);
const edt = window.btoa(JSON.stringify({ e: eventType, t: timestamp }));
```

### Fields

| Field | Type | Description | Evidence |
|-------|------|-------------|----------|
| `e` | `string` | Event type identifier | `CODE_REFERENCED` |
| `t` | `integer` | Unix timestamp in seconds | `CODE_REFERENCED` |

### Encoding

1. Build JSON object: `{ "e": eventType, "t": unixSeconds }`
2. `JSON.stringify()` the object
3. `window.btoa()` to base64-encode
4. Send as `edt` field in POST body `[CODE_REFERENCED]`

---

## 19. Download Flow

> All steps `CODE_REFERENCED` from `index-1.3.0.js`

### Step 1 — Determine Quality

```
if (user.subscription && user.settings.instant_download) {
  quality = highest_available;           // 720p for subscribers
} else {
  quality = show_quality_picker();       // UI picker for non-subscribers
}
```

| Condition | Behavior | Evidence |
|-----------|----------|----------|
| Subscribed + instant download enabled | Auto-select highest quality | `CODE_REFERENCED` |
| Subscribed, instant download disabled | Show quality picker | `CODE_REFERENCED` |
| Not subscribed | Show quality picker → reward ad → unlock | `CODE_REFERENCED` |

### Step 2 — Ad Gate (non-subscribers)

| Step | Detail | Evidence |
|------|--------|----------|
| 2a | Display rewarded ad (AdMob) | `CODE_REFERENCED` |
| 2b | User completes ad watch | `CODE_REFERENCED` |
| 2c | Grant download permission | `CODE_REFERENCED` |

### Step 3 — Fetch CDN URL

```
POST /api/v2/records/{accountId}/{recordId}/ups
Body: { edt: base64(JSON({ e, t })), quality: "720p" }
Response: { video: "https://cdn...", record: RecordObject }
```

| Field | Type | Evidence |
|-------|------|----------|
| Request `edt` | `string (base64)` | `CODE_REFERENCED` |
| Request `quality` | `string` | `CODE_REFERENCED` |
| Response `video` | `string` | `CODE_REFERENCED` — CDN URL |
| Response `record` | `RecordObject` | `CODE_REFERENCED` |

### Step 4 — Save to Device

```javascript
Capacitor.Plugins.Storage.saveVideo({
  url: cdnUrl,
  album: "TKREC",
  filename: `${username}_${streamId}.mp4`,
  extension: ".mp4"
});
```

| Field | Value | Evidence |
|-------|-------|----------|
| Plugin | `Capacitor.Plugins.Storage` | `CODE_REFERENCED` |
| Album | `"TKREC"` | `CODE_REFERENCED` |
| Format | `.mp4` | `CODE_REFERENCED` |

---

## 20. Ad Integration

> All data `CODE_REFERENCED` from `index-1.3.0.js`

### Google AdMob

| Field | Value | Evidence |
|-------|-------|----------|
| SDK | Google AdMob | `CODE_REFERENCED` |
| Ad Unit ID | `ca-app-pub-7501332842730537/7935013498` | `CODE_REFERENCED` |
| Format | Rewarded video | `CODE_REFERENCED` |

### Ad Display Rules

| User State | Ad Behavior | Evidence |
|------------|-------------|----------|
| Subscriber (`pro`) | No ads, direct playback/download | `CODE_REFERENCED` |
| Non-subscriber | Rewarded ad before video playback | `CODE_REFERENCED` |
| Non-subscriber | Rewarded ad before download | `CODE_REFERENCED` |
| Ad completion | Grants temporary access to content | `CODE_REFERENCED` |

### Ad Trigger Points

| Trigger | Location | Evidence |
|---------|----------|----------|
| Before video playback | Record detail view | `CODE_REFERENCED` |
| Before download | Download button press | `CODE_REFERENCED` |
| Daily bonus claim | Flip game bonus | `INFERRED` |

---

## Appendix A — Endpoint ID Reference

| ID | Method | Endpoint | Category |
|----|--------|----------|----------|
| 1 | POST | `/api/signin` | Auth |
| 2 | POST | `/api/login` | Auth |
| 3 | POST | `/api/register` | Auth |
| 4 | POST | `/api/login-device` | Auth |
| 5 | POST | `/api/confirm-email` | Auth |
| 6 | POST | `/api/forgot-password` | Auth |
| 7 | POST | `/api/reset-password/{token}` | Auth |
| 8 | GET | `/api/csrf` | Auth |
| 9 | POST | `/api/auth/fcm` | Auth |
| 10 | POST | `/api/status` | Auth |
| 11 | GET | `/api/v2/user` | User |
| 12 | PUT | `/api/v2/user` | User |
| 13 | DELETE | `/api/v2/user` | User |
| 14 | POST | `/api/v2/user/installed` | User |
| 15 | POST | `/api/v2/user/transfer-recordings/{accountId}/{targetId}` | User |
| 16 | POST | `/api/v2/user/profile` | Profiles |
| 17 | POST | `/api/v2/user/profile/{id}` | Profiles |
| 18 | DELETE | `/api/v2/user/profile/{id}` | Profiles |
| 19 | PUT | `/api/v2/user/profile/{id}/logout` | Profiles |
| 20 | POST | `/api/v2/user/profile/{id}/mute` | Profiles |
| 21 | GET | `/api/v2/followers/{accountId}/list` | Followers |
| 22 | POST | `/api/v2/followers/{accountId}` | Followers |
| 23 | DELETE | `/api/v2/followers/{accountId}/{id}` | Followers |
| 24 | PUT | `/api/v2/followers/{accountId}/{id}` | Followers |
| 25 | GET | `/api/v2/followers/{accountId}/{pid}` | Followers |
| 26 | GET | `/api/v2/followers/{accountId}/{pid}/history` | Followers |
| 27 | GET | `/api/v2/followers/{accountId}/{pid}/rm` | Followers |
| 28 | POST | `/api/v2/followers/{accountId}/{pid}/fetch` | Followers |
| 29 | POST | `/api/v2/followers/{accountId}/{pid}/validate` | Followers |
| 30 | POST | `/api/v2/followers/{accountId}/{pid}/deep-check` | Followers |
| 31 | POST | `/api/v2/followers/{accountId}/{pid}/report` | Followers |
| 32 | POST | `/api/v2/followers/{accountId}/{pid}/following/fetch` | Followers |
| 33 | GET | `/api/v2/followers/{accountId}/{uid}/following` | Followers |
| 34 | GET | `/api/v2/followers/{accountId}/{uid}/deep-check` | Followers |
| 35 | GET | `/api/v2/followers/discover/{accountId}` | Social |
| 36 | GET | `/api/v2/followers/discover/{accountId}/bubbles` | Social |
| 37 | GET | `/api/v2/followers/explore/lives/{accountId}` | Social |
| 38 | GET | `/api/v2/followers/ranked/{accountId}` | Social |
| 39 | GET | `/api/v2/followers/search/{accountId}` | Social |
| 40 | GET | `/api/v2/followers/similar/{accountId}/{pid}` | Social |
| 41 | GET | `/api/v2/followers/suggest/{accountId}` | Social |
| 42 | POST | `/api/v2/followers/{accountId}/{pid}/like` | Social |
| 43 | GET | `/api/v2/followers/{accountId}/likes` | Social |
| 44 | POST | `/api/v2/followers/hide/{accountId}/{pid}` | Social |
| 45 | POST | `/api/v2/followers/{accountId}/report` | Social |
| 46 | GET | `/api/v2/records/{accountId}` | Records |
| 47 | GET | `/api/v2/records/{accountId}/{recordId}` | Records |
| 48 | POST | `/api/v2/records/new/{accountId}` | Records |
| 49 | DELETE | `/api/v2/records/{accountId}/{recordId}` | Records |
| 50 | DELETE | `/api/v2/records/{accountId}/all` | Records |
| 51 | DELETE | `/api/v2/records/follower/{accountId}/{pid}` | Records |
| 52 | POST | `/api/v2/records/{accountId}/delete-selected` | Records |
| 53 | POST | `/api/v2/records/{accountId}/{recordId}/attach` | Records |
| 54 | PUT | `/api/v2/records/{accountId}/{recordId}/fix` | Records |
| 55 | POST | `/api/v2/records/{accountId}/{recordId}/ups` | Records |
| 56 | POST | `/api/v2/records/{accountId}/{recordId}/downloaded` | Records |
| 57 | GET | `/api/v2/records/{accountId}/storage-usage` | Records |
| 58 | GET | `/api/v2/records/follower/{accountId}/{pid}` | Records |
| 59 | POST | `/api/v2/broadcasts/{accountId}` | Streams |
| 60 | POST | `/api/streams/record-now/{accountId}` | Streams |
| 61 | POST | `/api/streams/rebuild/{accountId}` | Streams |
| 62 | POST | `/api/v2/record-profile/{accountId}` | Streams |
| 63 | POST | `/api/v2/record-profile/{username}` | Streams |
| 64 | GET | `/api/v2/favorites/{accountId}/list` | Favorites |
| 65 | POST | `/api/v2/favorites/{accountId}/{recordId}` | Favorites |
| 66 | DELETE | `/api/v2/favorites/{accountId}/{recordId}` | Favorites |
| 67 | POST | `/api/v2/purchase/storage` | Purchases |
| 68 | POST | `/api/v2/purchase/slots` | Purchases |
| 69 | GET | `/api/v2/purchase/history-list` | Purchases |
| 70 | POST | `ws.arkzynco.com/api/public/history/fetch` | Public |
| 71 | POST | `/api/public/stream-count` | Public |
| 72 | POST | `ws.arkzynco.com/api/history/fetch` | Public |
| 73 | POST | `/api/v2/logout` | Auth |
| 74 | POST | `/api/v2/network-log` | Misc |
| 75 | GET | `/api/v2/news` | Misc |
| 76 | GET | `/api/v2/promotions` | Misc |
| 77 | GET | `/api/v2/flip-game/state` | Misc |
| 78 | POST | `/api/v2/flip-game/play` | Misc |
| 79 | GET | `/api/slides` | Misc |
| 80 | GET | `/api/pages/{pageUrl}` | Misc |
| 81 | GET | `/api/pages/rewards-overview` | Misc |
| 82 | POST | `ws.arkzynco.com/api/followers/{accountId}` | Legacy |
| 83 | POST | `ws.arkzynco.com/api/profile/login` | Legacy |
| 84 | POST | `ws.arkzynco.com/api/profile/code` | Legacy |
| 85 | GET | `/api/pages/faq` | Misc |
| 86 | GET | `/api/v2/user/profile` | Profiles |
| 87 | POST | `ws.arkzynco.com/api/followers/{uid}` | Legacy |
| 88 | POST | `/api/v2/followers/{uid}/{pid}` | Legacy |
