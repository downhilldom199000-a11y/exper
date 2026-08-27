# TKREC API Reference

> Generated: 2026-08-26
> Complete API reference for TKREC 1.3.1 (88 endpoints)

---

## Table of Contents

1. [Overview](#overview)
2. [Authentication](#authentication)
3. [Authentication Endpoints](#authentication-endpoints)
4. [User Management](#user-management)
5. [User Profiles](#user-profiles)
6. [Followers Tracking](#followers-tracking)
7. [Followers Social](#followers-social)
8. [Records](#records)
9. [Broadcasts / Streams](#broadcasts--streams)
10. [Favorites](#favorites)
11. [Purchases](#purchases)
12. [Public](#public)
13. [Miscellaneous](#miscellaneous)
14. [Legacy / Undocumented](#legacy--undocumented)
15. [Data Models](#data-models)
16. [Error Handling](#error-handling)
17. [Pagination](#pagination)
18. [Rate Limiting](#rate-limiting)

---

## Overview

TKREC is a livestream recording management service. The API manages user accounts, follower profiles, stream recording lifecycle, video storage, and in-app purchases.

### Base URLs

| Name | URL | Usage |
|------|-----|-------|
| **Main API** | `https://raid.arkzynco.com` | Primary API — auth, user management, records, purchases |
| **Public API** | `https://ws.arkzynco.com` | Public endpoints — history fetch, social discovery, legacy |
| **Analytics** | `https://intake.arkzynco.com` | Telemetry and analytics intake |

### Required Headers

All authenticated requests must include:

```
Authorization: Bearer <jwt_token>
X-App-Version: 1.3.0
X-Requested-With: XMLHttpRequest
Content-Type: application/json
```

The CSRF token is required for state-changing requests (POST, PUT, DELETE):

```
X-XSRF-TOKEN: <xsrf_token>
```

The CSRF token is obtained from the `XSRF-TOKEN` cookie returned by `GET /api/csrf`.

---

## Authentication

### Obtaining a Token

1. **Email/Password Login** — `POST /api/signin` with `{ email, password }`.
2. **Social Login** — `POST /api/login` with `{ provider, token }`.
3. **Device Login** — `POST /api/login-device` for auto-login from a trusted device.
4. **Registration** — `POST /api/register` with `{ name, email, password }`.

All auth endpoints return a JWT in the response body (field `token` or nested inside `user`).

### Using the Token

Include it in every request:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### CSRF Flow

```
GET /api/csrf
  → Sets XSRF-TOKEN cookie
  → Returns { token: "..." }

POST /api/v2/records/...
  Headers:
    X-XSRF-TOKEN: <token from cookie>
    Authorization: Bearer <jwt>
```

### OAuth Status Exchange

`POST /api/status` — Exchange an OAuth token (e.g. Apple/Google) for a TKREC session token. Used as a final step after the provider callback.

### FCM Registration

`POST /api/auth/fcm` — Register a Firebase Cloud Messaging token for push notifications.

### Logout

`POST /api/v2/logout` — Invalidate the current session. Should be called with a valid JWT and CSRF token.

---

## Authentication Endpoints

### POST `/api/signin`

Email and password login.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "s3cureP@ss"
}
```

**Response (200):**

```json
{
  "token": "eyJhbGci...",
  "user": { /* UserObject */ }
}
```

---

### POST `/api/login`

Social login via third-party OAuth provider.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "provider": "google",
  "token": "oauth_provider_token_here"
}
```

`provider` is one of: `google`, `apple`, `twitter`, `facebook`.

**Response (200):**

```json
{
  "token": "eyJhbGci...",
  "user": { /* UserObject */ }
}
```

---

### POST `/api/register`

Create a new account.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "name": "John",
  "email": "john@example.com",
  "password": "myPassword123"
}
```

**Response (201):**

```json
{
  "token": "eyJhbGci...",
  "user": { /* UserObject */ }
}
```

---

### POST `/api/login-device`

Device-based auto-login for returning users with a trusted device token.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "device_token": "trusted_device_identifier"
}
```

**Response (200):**

```json
{
  "token": "eyJhbGci...",
  "user": { /* UserObject */ }
}
```

---

### POST `/api/confirm-email`

Confirm a user's email address via a confirmation code/link.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "token": "email_confirmation_token"
}
```

**Response (200):**

```json
{
  "message": "Email confirmed successfully"
}
```

---

### POST `/api/forgot-password`

Request a password reset email.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "email": "user@example.com"
}
```

**Response (200):**

```json
{
  "message": "If the email exists, a reset link has been sent"
}
```

---

### POST `/api/reset-password/{token}`

Execute a password reset using the token from the email.

| | |
|---|---|
| **Auth Required** | No |
| **Path Parameters** | `token` — password reset token from email |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "password": "newPassword456",
  "password_confirmation": "newPassword456"
}
```

**Response (200):**

```json
{
  "message": "Password has been reset"
}
```

---

### GET `/api/csrf`

Fetch a CSRF token. Sets the `XSRF-TOKEN` cookie.

| | |
|---|---|
| **Auth Required** | No |

**Response (200):**

```json
{
  "token": "abc123csrf..."
}
```

---

### POST `/api/auth/fcm`

Register a Firebase Cloud Messaging token for push notifications.

| | |
|---|---|
| **Auth Required** | Yes |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "fcm_token": "dGhpc0lzQW5GQ01Ub2tlbg..."
}
```

**Response (200):**

```json
{
  "message": "FCM token registered"
}
```

---

### POST `/api/status`

OAuth token exchange — convert a provider token into a TKREC session.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "provider": "apple",
  "token": "apple_id_token",
  "name": "Jane Doe",
  "email": "jane@privaterelay.appleid.com"
}
```

**Response (200):**

```json
{
  "token": "eyJhbGci...",
  "user": { /* UserObject */ }
}
```

---

### POST `/api/v2/logout`

Invalidate the current session.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |

**Response (200):**

```json
{
  "message": "Logged out"
}
```

---

## User Management

### GET `/api/v2/user`

Get the currently authenticated user's full profile.

| | |
|---|---|
| **Auth Required** | Yes |

**Response (200):**

```json
{
  "user": { /* UserObject */ }
}
```

---

### PUT `/api/v2/user`

Update the current user's profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Content-Type** | application/json |

**Request Body (all fields optional):**

```json
{
  "name": "New Name",
  "email": "new@example.com",
  "settings": {
    "default_quality": "1080p",
    "instant_download": true,
    "notifications_enabled": false,
    "dark_theme": true,
    "locale": "fr"
  }
}
```

**Response (200):**

```json
{
  "user": { /* UserObject with updated fields */ }
}
```

---

### DELETE `/api/v2/user`

Delete the current user's account and all associated data.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |

**Response (200):**

```json
{
  "message": "Account deleted"
}
```

---

### POST `/api/v2/user/installed`

Mark the app as installed on this device (used for onboarding state).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |

**Response (200):**

```json
{
  "message": "Installation recorded"
}
```

---

### POST `/api/v2/user/transfer-recordings/{accountId}/{targetId}`

Transfer all recordings from one account to another.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — source account ID, `targetId` — destination account ID |

**Response (200):**

```json
{
  "transferred": 42,
  "message": "Recordings transferred"
}
```

---

### GET `/api/v2/user/profile`

Get the list of profiles for the current user.

| | |
|---|---|
| **Auth Required** | Yes |

**Response (200):**

```json
{
  "profiles": [ /* ProfileObject[] */ ]
}
```

---

## User Profiles

### POST `/api/v2/user/profile`

Create a new profile for the current user.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "username": "pro_gamer_42",
  "nickname": "Pro Gamer"
}
```

**Response (201):**

```json
{
  "profile": { /* ProfileObject */ }
}
```

---

### POST `/api/v2/user/profile/{id}`

Update an existing profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `id` — profile ID |
| **Content-Type** | application/json |

**Request Body (all fields optional):**

```json
{
  "nickname": "Updated Nickname",
  "pic": "https://cdn.example.com/pic.jpg"
}
```

**Response (200):**

```json
{
  "profile": { /* ProfileObject with updated fields */ }
}
```

---

### DELETE `/api/v2/user/profile/{id}`

Delete a profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `id` — profile ID |

**Response (200):**

```json
{
  "message": "Profile deleted"
}
```

---

### PUT `/api/v2/user/profile/{id}/logout`

Log out a specific profile (e.g. remote logout from another device).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `id` — profile ID |

**Response (200):**

```json
{
  "message": "Profile logged out"
}
```

---

### POST `/api/v2/user/profile/{id}/mute`

Mute or unmute a profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `id` — profile ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "muted": true
}
```

**Response (200):**

```json
{
  "muted": true,
  "message": "Profile muted"
}
```

---

## Followers Tracking

These endpoints manage the profiles you actively track/monitor for livestreams.

### GET `/api/v2/followers/{accountId}/list`

List all tracked profiles for an account.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "followers": [ /* ProfileObject[] */ ],
  "total": 150,
  "page": 1,
  "per_page": 20
}
```

---

### POST `/api/v2/followers/{accountId}`

Add a profile to the tracking list.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "uid": "target_user_uid",
  "username": "target_username"
}
```

**Response (201):**

```json
{
  "follower": { /* ProfileObject */ },
  "message": "Profile added to tracking"
}
```

---

### DELETE `/api/v2/followers/{accountId}/{id}`

Remove a profile from tracking.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `id` — follower ID |

**Response (200):**

```json
{
  "message": "Profile removed from tracking"
}
```

---

### PUT `/api/v2/followers/{accountId}/{id}`

Toggle the active/paused status of a tracked profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `id` — follower ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "status": "active"
}
```

`status` is one of: `active`, `paused`.

**Response (200):**

```json
{
  "follower": { /* ProfileObject */ }
}
```

---

### GET `/api/v2/followers/{accountId}/{pid}`

Get details for a single tracked profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "follower": { /* ProfileObject */ }
}
```

---

### GET `/api/v2/followers/{accountId}/{pid}/history`

Get the live history (recent streams) of a tracked profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "history": [ /* RecordObject[] */ ],
  "total": 85,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/{accountId}/{pid}/rm`

Get removal information for a tracked profile (e.g. whether removal is safe, what will be lost).

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "records_count": 23,
  "storage_used": "1.2 GB",
  "can_remove": true
}
```

---

### POST `/api/v2/followers/{accountId}/{pid}/fetch`

Force refresh profile data from the source platform.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "follower": { /* ProfileObject with fresh data */ },
  "message": "Profile refreshed"
}
```

---

### POST `/api/v2/followers/{accountId}/{pid}/validate`

Validate active streams for a tracked profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "validating": true,
  "validation_progress": 0,
  "message": "Validation started"
}
```

---

### POST `/api/v2/followers/{accountId}/{pid}/deep-check`

Perform a deep check on a tracked profile (extended validation beyond basic stream check).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "deep_check": {
    "status": "running",
    "progress": 0
  }
}
```

---

### POST `/api/v2/followers/{accountId}/{pid}/report`

Report a profile for policy violations.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "reason": "spam",
  "details": "This account posts inappropriate content"
}
```

**Response (200):**

```json
{
  "message": "Report submitted"
}
```

---

### POST `/api/v2/followers/{accountId}/{pid}/following/fetch`

Fetch the "following" list of a tracked profile from the source platform.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "fetching": true,
  "message": "Following list fetch started"
}
```

---

### GET `/api/v2/followers/{accountId}/{uid}/following`

Get the cached "following" list for a profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `uid` — user ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "following": [ /* ProfileObject[] */ ],
  "total": 340,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/{accountId}/{uid}/deep-check`

Get the result/status of a previously initiated deep check.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `uid` — user ID |

**Response (200):**

```json
{
  "deep_check": {
    "status": "complete",
    "result": { /* ... */ }
  }
}
```

---

## Followers Social

Discovery, search, and social interaction endpoints.

### GET `/api/v2/followers/discover/{accountId}`

Discover new profiles to track.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page`, `category` (optional) |

**Response (200):**

```json
{
  "profiles": [ /* ProfileObject[] */ ],
  "total": 500,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/discover/{accountId}/bubbles`

Discover profiles via a "bubbles" UI (visual discovery).

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |

**Response (200):**

```json
{
  "bubbles": [
    {
      "profile": { /* ProfileObject */ },
      "weight": 0.85
    }
  ]
}
```

---

### GET `/api/v2/followers/explore/lives/{accountId}`

Explore currently live streams.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page`, `category` (optional) |

**Response (200):**

```json
{
  "lives": [ /* ProfileObject[] (live_now: true) */ ],
  "total": 120,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/ranked/{accountId}`

Get ranked/popular profiles.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page`, `sort` (optional: `followers`, `views`, `live_count`) |

**Response (200):**

```json
{
  "ranked": [ /* ProfileObject[] */ ],
  "total": 200,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/search/{accountId}`

Search for profiles by username or keyword.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `q` (search query), `page`, `per_page` |

**Response (200):**

```json
{
  "results": [ /* ProfileObject[] */ ],
  "total": 15,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/similar/{accountId}/{pid}`

Get profiles similar to a given profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "similar": [ /* ProfileObject[] */ ],
  "total": 30,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/followers/suggest/{accountId}`

Get AI-recommended profiles to follow.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |

**Response (200):**

```json
{
  "suggestions": [ /* ProfileObject[] */ ]
}
```

---

### POST `/api/v2/followers/{accountId}/{pid}/like`

Like or unlike a profile (toggle).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "liked": true
}
```

**Response (200):**

```json
{
  "liked": true,
  "message": "Profile liked"
}
```

---

### GET `/api/v2/followers/{accountId}/likes`

Get all liked profiles.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "likes": [ /* ProfileObject[] */ ],
  "total": 25,
  "page": 1,
  "per_page": 20
}
```

---

### POST `/api/v2/followers/hide/{accountId}/{pid}`

Hide a profile from discovery/recommendations.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "message": "Profile hidden"
}
```

---

### POST `/api/v2/followers/{accountId}/report`

Report profiles with pagination (batch report interface).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "profiles": ["pid_1", "pid_2"],
  "reason": "spam",
  "details": "Multiple accounts posting the same content"
}
```

**Response (200):**

```json
{
  "reported": 2,
  "message": "Profiles reported"
}
```

---

## Records

Manage recorded livestream videos.

### GET `/api/v2/records/{accountId}`

List all recordings for an account.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page`, `sort` (optional: `created_at`, `size`, `duration`), `order` (`asc`/`desc`) |

**Response (200):**

```json
{
  "records": [ /* RecordObject[] */ ],
  "total": 340,
  "page": 1,
  "per_page": 20
}
```

---

### GET `/api/v2/records/{accountId}/{recordId}`

Get a single recording's details.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |

**Response (200):**

```json
{
  "record": { /* RecordObject */ }
}
```

---

### POST `/api/v2/records/new/{accountId}`

Create a new recording entry (start tracking a stream).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "uid": "streamer_uid",
  "username": "streamer_name",
  "stream_id": "platform_stream_id"
}
```

**Response (201):**

```json
{
  "record": { /* RecordObject */ },
  "message": "Recording started"
}
```

---

### DELETE `/api/v2/records/{accountId}/{recordId}`

Delete a single recording.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |

**Response (200):**

```json
{
  "message": "Record deleted",
  "storage_freed": "450 MB"
}
```

---

### DELETE `/api/v2/records/{accountId}/all`

Delete all recordings for an account.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |

**Response (200):**

```json
{
  "deleted": 340,
  "storage_freed": "12.5 GB",
  "message": "All records deleted"
}
```

---

### DELETE `/api/v2/records/follower/{accountId}/{pid}`

Delete all recordings for a specific follower/profile.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |

**Response (200):**

```json
{
  "deleted": 45,
  "storage_freed": "2.1 GB",
  "message": "Follower records deleted"
}
```

---

### POST `/api/v2/records/{accountId}/delete-selected`

Bulk delete selected recordings.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "record_ids": ["rec_1", "rec_2", "rec_3"]
}
```

**Response (200):**

```json
{
  "deleted": 3,
  "storage_freed": "1.8 GB",
  "message": "Selected records deleted"
}
```

---

### POST `/api/v2/records/{accountId}/{recordId}/attach`

Attach metadata or a file to a recording.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |
| **Content-Type** | application/json or multipart/form-data |

**Request Body:**

```json
{
  "cover": "https://cdn.example.com/cover.jpg",
  "thumb": "https://cdn.example.com/thumb.jpg"
}
```

**Response (200):**

```json
{
  "record": { /* RecordObject with updated fields */ }
}
```

---

### PUT `/api/v2/records/{accountId}/{recordId}/fix`

Rebuild/fix a video that failed processing.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |

**Response (200):**

```json
{
  "rebuild": {
    "status": "pending",
    "can_rebuild": true
  },
  "message": "Rebuild queued"
}
```

---

### POST `/api/v2/records/{accountId}/{recordId}/ups`

Get a presigned upload/download URL for a video.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "quality": "1080p"
}
```

**Response (200):**

```json
{
  "url": "https://storage.example.com/signed/...",
  "expires_at": "2026-08-26T12:00:00Z"
}
```

---

### POST `/api/v2/records/{accountId}/{recordId}/downloaded`

Mark a recording as downloaded.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |

**Response (200):**

```json
{
  "record": { /* RecordObject (downloaded: true) */ }
}
```

---

### GET `/api/v2/records/{accountId}/storage-usage`

Get storage usage statistics for an account.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |

**Response (200):**

```json
{
  "used": "15.3 GB",
  "total": "50 GB",
  "records_count": 340,
  "percent_used": 30.6
}
```

---

### GET `/api/v2/records/follower/{accountId}/{pid}`

Get all recordings for a specific follower.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `pid` — profile ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "records": [ /* RecordObject[] */ ],
  "total": 45,
  "page": 1,
  "per_page": 20
}
```

---

## Broadcasts / Streams

### POST `/api/v2/broadcasts/{accountId}`

Build broadcast data for a stream (prepare recording parameters).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "uid": "streamer_uid",
  "stream_id": "platform_stream_id"
}
```

**Response (200):**

```json
{
  "broadcast": {
    "stream_url": "...",
    "quality_options": ["720p", "1080p"],
    "status": "ready"
  }
}
```

---

### POST `/api/streams/record-now/{accountId}`

Start recording immediately (legacy endpoint).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "uid": "streamer_uid",
  "username": "streamer_name"
}
```

**Response (200):**

```json
{
  "record": { /* RecordObject */ },
  "message": "Recording started"
}
```

---

### POST `/api/streams/rebuild/{accountId}`

Rebuild a stream recording (legacy endpoint).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "record_id": "rec_to_rebuild",
  "stream_id": "original_stream_id"
}
```

**Response (200):**

```json
{
  "rebuild": {
    "status": "pending"
  },
  "message": "Rebuild started"
}
```

---

### POST `/api/v2/record-profile/{accountId}`

Record a profile's current stream (v2 endpoint).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "uid": "target_uid",
  "username": "target_username",
  "quality": "1080p"
}
```

**Response (201):**

```json
{
  "record": { /* RecordObject */ },
  "message": "Profile recording started"
}
```

---

### POST `/api/v2/record-profile/{username}`

Record a profile by username (native Java client endpoint).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `username` — target username |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "quality": "1080p"
}
```

**Response (201):**

```json
{
  "record": { /* RecordObject */ },
  "message": "Recording started for @username"
}
```

---

## Favorites

### GET `/api/v2/favorites/{accountId}/list`

List all favorited recordings.

| | |
|---|---|
| **Auth Required** | Yes |
| **Path Parameters** | `accountId` — account ID |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "favorites": [ /* RecordObject[] */ ],
  "total": 12,
  "page": 1,
  "per_page": 20
}
```

---

### POST `/api/v2/favorites/{accountId}/{recordId}`

Add a recording to favorites.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |

**Response (201):**

```json
{
  "message": "Added to favorites"
}
```

---

### DELETE `/api/v2/favorites/{accountId}/{recordId}`

Remove a recording from favorites.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Path Parameters** | `accountId` — account ID, `recordId` — record ID |

**Response (200):**

```json
{
  "message": "Removed from favorites"
}
```

---

## Purchases

### POST `/api/v2/purchase/storage`

Purchase additional storage.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "plan": "storage_50gb",
  "payment_method": "google_play",
  "receipt": "in_app_purchase_receipt_token"
}
```

**Response (200):**

```json
{
  "purchase": {
    "id": "purch_abc123",
    "plan": "storage_50gb",
    "amount": 4.99,
    "currency": "USD",
    "status": "completed"
  },
  "user": { /* UserObject with updated storage */ }
}
```

---

### POST `/api/v2/purchase/slots`

Purchase additional follower/recording slots.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "plan": "slots_25",
  "payment_method": "google_play",
  "receipt": "in_app_purchase_receipt_token"
}
```

**Response (200):**

```json
{
  "purchase": {
    "id": "purch_def456",
    "plan": "slots_25",
    "amount": 2.99,
    "currency": "USD",
    "status": "completed"
  },
  "user": { /* UserObject with updated slot count */ }
}
```

---

### GET `/api/v2/purchase/history-list`

Get purchase history.

| | |
|---|---|
| **Auth Required** | Yes |
| **Query Parameters** | `page`, `per_page` (optional) |

**Response (200):**

```json
{
  "purchases": [
    {
      "id": "purch_abc123",
      "plan": "storage_50gb",
      "amount": 4.99,
      "currency": "USD",
      "status": "completed",
      "platform": "google_play",
      "created_at": "2026-07-15T10:30:00Z"
    }
  ],
  "total": 5,
  "page": 1,
  "per_page": 20
}
```

---

## Public

Endpoints that may not require authentication or use the public base URL.

### POST `ws.arkzynco.com/api/public/history/fetch`

Fetch the username history (name changes) of a public profile.

| | |
|---|---|
| **Auth Required** | No |
| **Base URL** | `https://ws.arkzynco.com` |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "username": "target_username"
}
```

**Response (200):**

```json
{
  "history": [
    {
      "username": "old_name",
      "changed_at": "2026-01-10T00:00:00Z"
    },
    {
      "username": "current_name",
      "changed_at": "2026-06-20T00:00:00Z"
    }
  ]
}
```

---

### POST `/api/public/stream-count`

Get the stream count for a profile.

| | |
|---|---|
| **Auth Required** | No |
| **Content-Type** | application/json |

> **Note:** This endpoint currently returns 404. It may be deprecated or not yet deployed.

**Request Body:**

```json
{
  "uid": "target_uid"
}
```

**Expected Response (when available):**

```json
{
  "stream_count": 142
}
```

---

### POST `ws.arkzynco.com/api/history/fetch`

Fetch stream history with authentication (uses the public WS base URL but requires a JWT).

| | |
|---|---|
| **Auth Required** | Yes |
| **Base URL** | `https://ws.arkzynco.com` |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "uid": "target_uid",
  "username": "target_username"
}
```

**Response (200):**

```json
{
  "history": [ /* RecordObject[] */ ],
  "total": 85
}
```

---

## Miscellaneous

### POST `/api/v2/network-log`

Report a network error from the client for diagnostics.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |
| **Content-Type** | application/json |

**Request Body:**

```json
{
  "error": "ETIMEDOUT",
  "endpoint": "/api/v2/records/123",
  "timestamp": "2026-08-26T08:15:00Z",
  "retry_count": 3
}
```

**Response (200):**

```json
{
  "message": "Log recorded"
}
```

---

### GET `/api/v2/news`

Get news and announcements.

| | |
|---|---|
| **Auth Required** | Yes |

**Response (200):**

```json
{
  "news": [
    {
      "id": "news_1",
      "title": "TKREC 1.3.1 Released",
      "body": "Performance improvements and bug fixes.",
      "image": "https://cdn.example.com/news/1.3.1.png",
      "published_at": "2026-08-20T00:00:00Z",
      "url": "https://example.com/blog/1.3.1"
    }
  ]
}
```

---

### GET `/api/v2/promotions`

Get active promotional offers.

| | |
|---|---|
| **Auth Required** | Yes |

**Response (200):**

```json
{
  "promotions": [
    {
      "id": "promo_summer26",
      "title": "Summer Sale",
      "description": "50% off storage upgrades",
      "discount_percent": 50,
      "valid_until": "2026-09-01T00:00:00Z",
      "target_plans": ["storage_50gb", "storage_100gb"]
    }
  ]
}
```

---

### GET `/api/v2/flip-game/state`

Get the current state of the in-app flip game.

| | |
|---|---|
| **Auth Required** | Yes |

**Response (200):**

```json
{
  "plays_remaining": 3,
  "daily_plays": 5,
  "rewards": [
    { "type": "storage", "amount": "100 MB", "chance": 0.3 },
    { "type": "slots", "amount": 5, "chance": 0.1 },
    { "type": "nothing", "amount": 0, "chance": 0.6 }
  ]
}
```

---

### POST `/api/v2/flip-game/play`

Play the flip game.

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Yes |

**Response (200):**

```json
{
  "result": {
    "type": "storage",
    "amount": "100 MB"
  },
  "plays_remaining": 2,
  "message": "You won 100 MB of storage!"
}
```

---

### GET `/api/slides`

Get onboarding slides (first-launch tutorial).

| | |
|---|---|
| **Auth Required** | No |

**Response (200):**

```json
{
  "slides": [
    {
      "id": 1,
      "title": "Welcome to TKREC",
      "description": "Record your favorite livestreams automatically.",
      "image": "https://cdn.example.com/slides/welcome.png"
    },
    {
      "id": 2,
      "title": "Track Profiles",
      "description": "Add streamers and we'll record them for you.",
      "image": "https://cdn.example.com/slides/track.png"
    }
  ]
}
```

---

### GET `/api/pages/{pageUrl}`

Get CMS page content by URL slug.

| | |
|---|---|
| **Auth Required** | No |
| **Path Parameters** | `pageUrl` — URL slug (e.g. `terms`, `privacy`, `about`) |

**Response (200):**

```json
{
  "page": {
    "title": "Terms of Service",
    "content": "<h1>Terms of Service</h1><p>...</p>",
    "updated_at": "2026-08-01T00:00:00Z"
  }
}
```

---

### GET `/api/pages/rewards-overview`

Get the rewards overview page content.

| | |
|---|---|
| **Auth Required** | No |

**Response (200):**

```json
{
  "page": {
    "title": "Rewards Overview",
    "content": "<h1>Earn Rewards</h1><p>...</p>",
    "tiers": [
      { "name": "Bronze", "min_plays": 0, "bonus_multiplier": 1.0 },
      { "name": "Silver", "min_plays": 50, "bonus_multiplier": 1.5 },
      { "name": "Gold", "min_plays": 200, "bonus_multiplier": 2.0 }
    ]
  }
}
```

---

### GET `/api/pages/faq`

Get the FAQ page content.

| | |
|---|---|
| **Auth Required** | No |

**Response (200):**

```json
{
  "page": {
    "title": "Frequently Asked Questions",
    "sections": [
      {
        "question": "How does recording work?",
        "answer": "TKREC monitors streamers you follow and automatically records their live streams."
      },
      {
        "question": "How much storage do I get?",
        "answer": "Free accounts get 10 GB. Premium plans offer 50 GB and 100 GB."
      }
    ]
  }
}
```

---

## Legacy / Undocumented

These endpoints exist in the codebase but are either deprecated or undocumented. They may be removed in future versions.

### POST `ws.arkzynco.com/api/followers/{accountId}`

Legacy follower addition endpoint.

| | |
|---|---|
| **Auth Required** | Yes |
| **Base URL** | `https://ws.arkzynco.com` |
| **Path Parameters** | `accountId` — account ID |
| **Status** | Legacy — prefer `POST /api/v2/followers/{accountId}` |

---

### POST `ws.arkzynco.com/api/profile/login`

Legacy profile login endpoint.

| | |
|---|---|
| **Auth Required** | Yes |
| **Base URL** | `https://ws.arkzynco.com` |
| **Status** | Legacy — replaced by v2 auth flow |

---

### POST `ws.arkzynco.com/api/profile/code`

Legacy verification code endpoint.

| | |
|---|---|
| **Auth Required** | Yes |
| **Base URL** | `https://ws.arkzynco.com` |
| **Status** | Legacy — replaced by v2 auth flow |

---

### POST `ws.arkzynco.com/api/followers/{uid}`

Undocumented follower endpoint.

| | |
|---|---|
| **Auth Required** | Likely Yes |
| **Base URL** | `https://ws.arkzynco.com` |
| **Path Parameters** | `uid` — user ID |
| **Status** | Undocumented — use with caution |

---

### POST `/api/v2/followers/{uid}/{pid}`

Unlock the history of a profile (possibly behind a paywall or restriction).

| | |
|---|---|
| **Auth Required** | Yes |
| **CSRF Required** | Likely Yes |
| **Path Parameters** | `uid` — user ID, `pid` — profile ID |
| **Status** | Undocumented — behavior unclear |

---

## Data Models

### UserObject

The core user entity returned by most authenticated endpoints.

```json
{
  "id": "usr_abc123",
  "email": "user@example.com",
  "name": "John Doe",
  "storage": {
    "used": "15.3 GB",
    "total": "50 GB",
    "percent": 30.6
  },
  "subscription": {
    "status": "active",
    "plan": "premium",
    "expires_at": "2026-12-31T23:59:59Z",
    "platform": "google_play"
  },
  "settings": {
    "default_quality": "1080p",
    "instant_download": true,
    "notifications_enabled": true,
    "dark_theme": false,
    "locale": "en"
  },
  "profiles": [
    { /* ProfileObject */ }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique user identifier |
| `email` | string | User email address |
| `name` | string | Display name |
| `storage` | object | Storage usage stats |
| `subscription` | object | Subscription details |
| `subscription.status` | string | `active`, `expired`, `cancelled`, `none` |
| `subscription.plan` | string | `free`, `basic`, `premium` |
| `subscription.expires_at` | string (ISO 8601) | Expiration date |
| `subscription.platform` | string | `google_play`, `apple`, `stripe`, `none` |
| `settings` | object | User preferences |
| `settings.default_quality` | string | Default recording quality |
| `settings.instant_download` | boolean | Auto-download after recording |
| `settings.notifications_enabled` | boolean | Push notifications on/off |
| `settings.dark_theme` | boolean | Dark theme preference |
| `settings.locale` | string | Language code (e.g. `en`, `fr`, `ar`) |
| `profiles` | ProfileObject[] | Associated profiles |

---

### ProfileObject

Represents a tracked streamer profile.

```json
{
  "id": "prf_xyz789",
  "uid": "platform_uid_123",
  "username": "pro_gamer_42",
  "nickname": "Pro Gamer",
  "pic": "https://cdn.example.com/profiles/pro_gamer_42.jpg",
  "video_count": 142,
  "storage": "3.2 GB",
  "deletable_count": 28,
  "validating": false,
  "validation_progress": null,
  "pending_validation": 0,
  "available_sync": true,
  "available_streams": 2,
  "reactivated_at": null,
  "last_live": "2026-08-25T22:30:00Z",
  "live_now": false,
  "real": true,
  "status": "active",
  "follow_type": "manual",
  "total_live": 340,
  "total_views": 125000,
  "total_followers": 45200
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Internal profile ID |
| `uid` | string | Platform-specific user ID |
| `username` | string | Streamer username/handle |
| `nickname` | string | Display name |
| `pic` | string | Profile picture URL |
| `video_count` | number | Number of recorded videos |
| `storage` | string | Total storage used by recordings |
| `deletable_count` | number | Number of records eligible for deletion |
| `validating` | boolean | Whether validation is in progress |
| `validation_progress` | number \| null | Validation progress (0-100) or null |
| `pending_validation` | number | Number of streams awaiting validation |
| `available_sync` | boolean | Whether sync is available |
| `available_streams` | number | Currently available live streams |
| `reactivated_at` | string \| null | When the profile was reactivated |
| `last_live` | string (ISO 8601) \| null | Last time the streamer went live |
| `live_now` | boolean | Whether the streamer is currently live |
| `real` | boolean | Whether this is a verified/real profile |
| `status` | string | `active`, `paused`, `removed`, `banned` |
| `follow_type` | string | `manual`, `auto`, `suggested`, `imported` |
| `total_live` | number | Total number of past livestreams |
| `total_views` | number | Total historical views |
| `total_followers` | number | Follower count on the platform |

---

### RecordObject

Represents a single recorded livestream.

```json
{
  "id": "rec_abc123def",
  "status": "completed",
  "thumb": "https://cdn.example.com/thumbs/rec_abc123.jpg",
  "pic": "https://cdn.example.com/covers/rec_abc123.jpg",
  "username": "pro_gamer_42",
  "uid": "platform_uid_123",
  "nickname": "Pro Gamer",
  "stream_id": "platform_stream_987",
  "videos": [
    {
      "q": "1080p",
      "size": "1.2 GB",
      "quality": "1920x1080"
    },
    {
      "q": "720p",
      "size": "650 MB",
      "quality": "1280x720"
    },
    {
      "q": "480p",
      "size": "320 MB",
      "quality": "854x480"
    }
  ],
  "favorites": 3,
  "views": 156,
  "downloaded": true,
  "cover": "https://cdn.example.com/covers/rec_abc123_full.jpg",
  "rebuild": {
    "status": "completed",
    "can_rebuild": true,
    "rebuilt_at": "2026-08-26T02:15:00Z"
  },
  "created_at": "2026-08-25T22:30:00Z",
  "updated_at": "2026-08-26T02:30:00Z",
  "duration": 7200,
  "size": "2.17 GB",
  "real": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique record identifier |
| `status` | string | `recording`, `processing`, `completed`, `failed`, `deleted` |
| `thumb` | string | Thumbnail URL |
| `pic` | string | Cover image URL |
| `username` | string | Streamer username |
| `uid` | string | Streamer platform UID |
| `nickname` | string | Streamer display name |
| `stream_id` | string | Original stream identifier |
| `videos` | array | Available video quality variants |
| `videos[].q` | string | Quality label (`1080p`, `720p`, `480p`) |
| `videos[].size` | string | File size (human-readable) |
| `videos[].quality` | string | Resolution (`1920x1080`) |
| `favorites` | number | Times favorited |
| `views` | number | View count |
| `downloaded` | boolean | Whether the user has downloaded this |
| `cover` | string | Full-size cover image URL |
| `rebuild` | object | Video rebuild/reprocessing info |
| `rebuild.status` | string | `pending`, `processing`, `completed`, `failed` |
| `rebuild.can_rebuild` | boolean | Whether a rebuild is possible |
| `rebuild.rebuilt_at` | string (ISO 8601) \| null | When last rebuild completed |
| `created_at` | string (ISO 8601) | Recording start time |
| `updated_at` | string (ISO 8601) | Last update time |
| `duration` | number | Duration in seconds |
| `size` | string | Total size across all qualities |
| `real` | boolean | Whether this is a verified recording |

---

## Error Handling

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| `200` | Success |
| `201` | Created |
| `400` | Bad Request — invalid parameters or malformed JSON |
| `401` | Unauthorized — missing or invalid JWT token |
| `403` | Forbidden — valid token but insufficient permissions / CSRF failure |
| `404` | Not Found — resource does not exist or endpoint is deprecated |
| `422` | Unprocessable Entity — validation errors in request body |
| `429` | Too Many Requests — rate limit exceeded (see [Rate Limiting](#rate-limiting)) |
| `500` | Internal Server Error — server-side failure |

### Error Response Format

All errors follow a consistent structure:

```json
{
  "error": "error_code",
  "message": "Human-readable description of the error",
  "details": {
    "field_name": ["This field is required"]
  }
}
```

### Common Error Codes

| Error Code | Description |
|------------|-------------|
| `unauthorized` | No valid token provided |
| `token_expired` | JWT has expired — re-authenticate |
| `invalid_credentials` | Wrong email/password |
| `email_not_confirmed` | Email confirmation required |
| `rate_limited` | Too many requests — back off |
| `validation_error` | Request body failed validation |
| `not_found` | Requested resource does not exist |
| `storage_full` | Account storage limit reached |
| `subscription_expired` | Feature requires active subscription |
| `csrf_mismatch` | CSRF token missing or invalid |

### Handling 401 Errors

If you receive a `401`, the token is invalid or expired. Re-authenticate via `/api/signin` or `/api/login-device`.

### Handling 403 Errors

If you receive a `403` on a state-changing request:
1. Ensure the `X-XSRF-TOKEN` header is set.
2. Fetch a fresh CSRF token via `GET /api/csrf`.
3. Read the `XSRF-TOKEN` cookie from the response.
4. Include it in the `X-XSRF-TOKEN` header on retry.

### Handling 429 Errors

Implement exponential backoff. Start at 1 second, double on each retry, max 60 seconds. Check the `Retry-After` header if present.

---

## Pagination

Most list endpoints support cursor-based pagination via `page` and `per_page` query parameters.

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `page` | `1` | Page number (1-indexed) |
| `per_page` | `20` | Items per page (max varies by endpoint, typically 50-100) |

### Response Metadata

Paginated responses include:

```json
{
  "data": [ /* ... */ ],
  "total": 340,
  "page": 1,
  "per_page": 20
}
```

| Field | Type | Description |
|-------|------|-------------|
| `total` | number | Total number of items across all pages |
| `page` | number | Current page number |
| `per_page` | number | Items per page requested |

### Example Pagination Flow

```
# Page 1
GET /api/v2/records/usr_abc123?page=1&per_page=20
  → { records: [...20 items...], total: 340, page: 1, per_page: 20 }

# Page 2
GET /api/v2/records/usr_abc123?page=2&per_page=20
  → { records: [...20 items...], total: 340, page: 2, per_page: 20 }

# Last page (page 17, only 0 items left)
GET /api/v2/records/usr_abc123?page=17&per_page=20
  → { records: [], total: 340, page: 17, per_page: 20 }
```

---

## Rate Limiting

The API enforces rate limits to maintain service stability.

### Limits

| Tier | Limit | Window |
|------|-------|--------|
| **Unauthenticated** | 60 requests | per minute |
| **Authenticated (free)** | 300 requests | per minute |
| **Authenticated (premium)** | 600 requests | per minute |

### Headers

Rate limit information is included in response headers:

```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 287
X-RateLimit-Reset: 1693036800
```

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests allowed in the window |
| `X-RateLimit-Remaining` | Requests remaining in the current window |
| `X-RateLimit-Reset` | Unix timestamp when the window resets |

### Best Practices

1. **Cache responses** where possible — profile data, news, slides, and CMS pages change infrequently.
2. **Use bulk endpoints** — `POST /api/v2/records/{accountId}/delete-selected` instead of individual deletes.
3. **Implement backoff** — on `429` responses, wait for the `Retry-After` header value before retrying.
4. **Batch operations** — when manipulating multiple records, use bulk endpoints rather than sequential individual calls.
5. **Poll judiciously** — avoid tight polling loops for live status; use webhooks or FCM push notifications when available.
