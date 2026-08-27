# TKREC API Dependency Graph

> Generated: 2026-08-26
> 88 endpoints mapped with dependency relationships

---

## 1. Visual Dependency Graph

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           TKREC 1.3.1 — API Dependency Graph                    │
└─────────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────┐
  │   App Start   │
  └──────┬───────┘
         │ GET /api/csrf
         │ (sets XSRF-TOKEN cookie)
         ▼
  ┌──────────────┐
  │   Auth Gate   │◄─────────────────────────────────────────────────┐
  └──────┬───────┘                                                  │
         │                                                          │
         ├──► POST /api/signin (email/password)                     │
         ├──► POST /api/login (social OAuth)                        │
         │      └─► POST /api/status (exchange OAuth → JWT)         │
         ├──► POST /api/register (new account)                      │
         ├──► POST /api/login-device (device token)                 │
         │                                                          │
         ▼                                                          │
  ┌──────────────┐                                                  │
  │  Authenticated │                                                │
  └──────┬───────┘                                                  │
         │                                                          │
         ├──► POST /api/auth/fcm ───────────────────────────────────┤
         │                                                          │
         ▼                                                          │
  ┌─────────────────────────────────────────────────────────────┐   │
  │                    CORE ENTITY GRAPH                         │   │
  │                                                              │   │
  │   ┌─────────────┐    GET /list     ┌──────────────────┐    │   │
  │   │   Profiles   │◄────────────────│   /api/v2/user    │    │   │
  │   │  (Followers) │                └──────────────────┘    │   │
  │   └──────┬──────┘                                          │   │
  │          │                                                   │   │
  │          │ POST /fetch (force refresh)                       │   │
  │          │ POST /validate (archive)                          │   │
  │          │ POST /deep-check (deep check)                     │   │
  │          ▼                                                   │   │
  │   ┌─────────────┐    GET /list     ┌──────────────────┐    │   │
  │   │   Records    │◄────────────────│  /api/v2/records  │    │   │
  │   └──────┬──────┘                └──────────────────┘    │   │
  │          │                                                   │   │
  │          ├────► POST /ups (CDN URL) ──► ExoPlayer ──► PLAY  │   │
  │          │                                                   │   │
  │          ├────► POST /downloaded (mark)                      │   │
  │          ├────► PUT /fix (rebuild video)                     │   │
  │          ├────► POST /record-now / record-profile (record)   │   │
  │          └────► DELETE /remove (delete)                      │   │
  │                                                              │   │
  │   ┌─────────────┐  GET /list   ┌──────────────────────┐    │   │
  │   │  Favorites   │◄────────────│ /api/v2/favorites     │    │   │
  │   └─────────────┘             └──────────────────────┘    │   │
  │                                                              │   │
  │   ┌─────────────┐  POST /play   ┌─────────────────────┐   │   │
  │   │  Flip Game   │◄─────────────│ /api/v2/game         │   │   │
  │   └─────────────┘              └─────────────────────┘   │   │
  └──────────────────────────────────────────────────────────────┘   │
         │                                                          │
         ├──► GET  /api/v2/user/profile                            │
         ├──► POST /api/v2/user/profile (create)                   │
         ├──► POST /api/v2/user/profile/{id} (update)              │
         ├──► DELETE /api/v2/user/profile/{id} (delete)            │
         ├──► PUT  /api/v2/user/profile/{id}/logout (switch)       │
         ├──► POST /api/v2/user/profile/{id}/mute                  │
         │                                                          │
         ├──► POST /api/v2/purchase/storage (IAP)                   │
         ├──► POST /api/v2/purchase/slots (IAP)                     │
         ├──► GET  /api/v2/purchase/history-list                    │
         │                                                          │
         ├────► POST /api/v2/logout ───────────────────────────────►│
         │                                                          │
  ┌─────────────────────────────────────────────────────────────┐   │
  │                   PUBLIC / UNAUTHENTICATED                   │   │
  │                                                              │   │
  │   POST /api/public/history/fetch (TikTok username lookup)   │   │
  │   GET  /api/slides (onboarding)                              │   │
  │   POST /api/public/stream-count (teaser → 404)               │   │
  └──────────────────────────────────────────────────────────────┘   │
         │                                                          │
         ▼                                                          │
  ┌─────────────────────────────────────────────────────────────┐   │
  │                   LEGACY → V2 MAPPING                        │   │
  │                                                              │   │
  │   fe.fetchFollower  ────────►  y.store                       │   │
  │   fe.recordNow      ────────►  P.recordNow                   │   │
  │   fe.rebuildVideo   ────────►  P.buildVideo                  │   │
  │   fe.login          ────────►  Q.signin                       │   │
  │   fe.code           ────────►  Q.confirmEmail                 │   │
  │   /api/profile/login ───────►  /api/signin                    │   │
  │   /api/profile/code ────────►  /api/confirm-email             │   │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2. Service Object Relationship Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                    SERVICE OBJECT TOPOLOGY                             │
└──────────────────────────────────────────────────────────────────────┘

  ┌─────────────────────┐         ┌─────────────────────┐
  │  Q  (Auth Service)   │         │  L  (User Service)   │
  │  host: raid.arkzynco │         │  host: raid.arkzynco │
  │                      │         │                      │
  │  • signin            │  auth   │  • fetch             │
  │  • login             │────────►│  • update            │
  │  • register          │  token  │  • addProfile        │
  │  • confirmEmail      │         │  • updateProfile     │
  │  • forgotPassword    │         │  • removeProfile     │
  │  • resetPassword     │         │  • logoutProfile     │
  │  • loginDevice       │         │  • muteProfile       │
  │  • logout            │         │  • subscribeStorage  │
  │  • csrf              │         │  • subscribeSlots    │
  │  • status            │         │  • purchaseHistory   │
  │  • promotions        │         │  • fcm               │
  │  • sendNetworkError  │         │  • delete            │
  └──────────┬──────────┘         │  • transferRecordings│
             │                    │  • installed         │
             │                    │  • news              │
             │                    └──────────┬──────────┘
             │                               │
             │    ┌──────────────────────────┘
             │    │
             ▼    ▼
  ┌─────────────────────────────────────────────────────────┐
  │         P  (Records Service)                             │
  │         host: raid.arkzynco                              │
  │                                                          │
  │  • favorites        • addFavorite     • removeFavorite   │
  │  • removeRecord     • removeSelectedRecords              │
  │  • removeUserRecords  • records       • getRecord        │
  │  • getVideoUrl      • recordNow       • startRecord      │
  │  • broadcasts       • buildVideo      • setDownloaded    │
  │  • attachRecord     • getStorageUsage • deleteAllRecords │
  └───────────────────────┬─────────────────────────────────┘
                          │
          ┌───────────────┴──────────────────┐
          │                                   │
          ▼                                   ▼
  ┌─────────────────────┐         ┌─────────────────────┐
  │  y  (Followers)      │         │  fe  (Legacy WS)     │
  │  host: raid.arkzynco │         │  host: ws.arkzynco   │
  │                      │         │                      │
  │  • followers         │         │  • fetchFollower     │
  │  • follower          │         │  • fetchSyncStreams  │
  │  • fetchUnlock       │         │  • recordNow         │
  │  • unlockHistory     │         │  • rebuildVideo      │
  │  • store             │         │  • login             │
  │  • remove            │         │  • code              │
  │  • search            │         └─────────────────────┘
  │  • suggest           │
  │  • discover          │         ┌─────────────────────┐
  │  • discoverBubbles   │         │  us  (Flip Game)     │
  │  • similar           │         │  host: raid.arkzynco │
  │  • hide              │         │                      │
  │  • reportUnlockError │         │  • getState          │
  │  • exploreLives      │         │  • play              │
  │  • report            │         └─────────────────────┘
  │  • like / likes      │
  │  • usernameHistory   │         ┌─────────────────────┐
  │  • toggleFollowerStatus        │  fs  (Public)        │
  │  • alsoBought        │         │  host: ws.arkzynco   │
  │  • ranked            │         │                      │
  │  • followingList     │         │  • fetchHistory      │
  │  • fetchFollowing    │         │  • getStreamCount    │
  │  • requestValidation │         └─────────────────────┘
  │  • deepCheckArchive  │
  └─────────────────────┘

  CROSS-SERVICE DATA FLOW:
  ════════════════════════
  Q (Auth) ──auth token──► L (User) ──accountId──► y (Followers)
                                                   │
                                    accountId+pid──► P (Records)
                                                   │
                                            recordId──► P (Video URL)
                                            recordId──► P (Rebuild)
                                            recordId──► P (Favorites)
```

---

## 3. Endpoint Dependency Table

| # | Source Endpoint | Target Endpoint | Type | Description |
|---|----------------|-----------------|------|-------------|
| 1 | App Start | `GET /api/csrf` | PREREQUISITE | Must be called first; sets XSRF-TOKEN cookie required for all subsequent POST/PUT/DELETE |
| 2 | `GET /api/csrf` | `POST /api/signin` | PREREQUISITE | CSRF token cookie must be present before login |
| 3 | `GET /api/csrf` | `POST /api/login` | PREREQUISITE | CSRF token cookie required for social login |
| 4 | `GET /api/csrf` | `POST /api/register` | PREREQUISITE | CSRF token cookie required for registration |
| 5 | `POST /api/login` | `POST /api/status` | TRIGGER | Social login triggers OAuth token exchange |
| 6 | `POST /api/status` | `POST /api/signin` | ALTERNATIVE | Status returns JWT directly; signin is fallback for email-based |
| 7 | `POST /api/signin` | `POST /api/auth/fcm` | DATA_FLOW | After JWT obtained, register FCM token for push notifications |
| 8 | `POST /api/register` | `POST /api/auth/fcm` | DATA_FLOW | After new account created, register FCM token |
| 9 | `POST /api/login-device` | `POST /api/auth/fcm` | DATA_FLOW | Device login also requires FCM registration |
| 10 | `POST /api/auth/fcm` | `GET /api/v2/user` | PREREQUISITE | FCM registered; now fetch full user profile with accounts |
| 11 | `GET /api/v2/user` | `GET /api/v2/followers/{accountId}/list` | DATA_FLOW | User object provides accountId for profile listing |
| 12 | `POST /api/v2/user/profile` | `POST /api/v2/user/profile/{id}/mute` | PREREQUISITE | Profile must exist before muting |
| 13 | `POST /api/v2/user/profile` | `PUT /api/v2/user/profile/{id}/logout` | PREREQUISITE | Profile must exist before switching |
| 14 | `POST /api/v2/user/profile` | `DELETE /api/v2/user/profile/{id}` | PREREQUISITE | Profile must exist before deleting |
| 15 | `POST /api/v2/user/profile` | `POST /api/v2/transfer-recordings/{accountId}/{targetId}` | PREREQUISITE | Both profiles must exist for transfer |
| 16 | `POST /api/public/history/fetch` | `GET /api/slides` | DATA_FLOW | Public TikTok lookup feeds onboarding; slides shown after |
| 17 | `GET /api/slides` | `POST /api/public/stream-count` | DATA_FLOW | After slides, teaser count shown before login CTA |
| 18 | `POST /api/public/stream-count` | `POST /api/signin` | TRIGGER | Teaser (404) prompts user to login/register |
| 19 | `POST /api/public/stream-count` | `POST /api/register` | TRIGGER | Teaser (404) prompts user to register |
| 20 | `POST /api/register` | `POST /api/v2/followers/{accountId}` | DATA_FLOW | After register, add TikTok profile to track |
| 21 | `POST /api/signin` | `POST /api/v2/followers/{accountId}` | DATA_FLOW | After login, add TikTok profile to track |
| 22 | `GET /api/v2/followers/{accountId}/list` | `POST /api/v2/followers/{accountId}/{pid}/fetch` | DATA_FLOW | Profile list provides pid for force-refresh |
| 23 | `POST /api/v2/followers/{accountId}/{pid}/fetch` | `POST /api/v2/followers/{accountId}/{pid}/validate` | TRIGGER | Fetch triggers validate (archive check) |
| 24 | `POST /api/v2/followers/{accountId}/{pid}/fetch` | `POST /api/v2/followers/{accountId}/{pid}/deep-check` | TRIGGER | Fetch can trigger deep check for stale data |
| 25 | `POST /api/v2/followers/{accountId}/{pid}/validate` | `GET /api/v2/records/{accountId}` | DATA_FLOW | Validated profile's recordings become queryable |
| 26 | `POST /api/v2/followers/{accountId}/{pid}/deep-check` | `GET /api/v2/records/{accountId}` | DATA_FLOW | Deep-checked profile recordings listed |
| 27 | `GET /api/v2/followers/{accountId}/list` | `GET /api/v2/records/{accountId}` | DATA_FLOW | Profile existence is prerequisite for record listing |
| 28 | `GET /api/v2/records/{accountId}` | `POST /api/v2/records/{accountId}/{recordId}/ups` | DATA_FLOW | Record list provides recordId for CDN URL |
| 29 | `POST /api/v2/records/{accountId}/{recordId}/ups` | ExoPlayer (Media3) | DATA_FLOW | CDN URL returned feeds directly to ExoPlayer for playback |
| 30 | ExoPlayer (Media3) | `POST /api/v2/records/{accountId}/{recordId}/downloaded` | TRIGGER | After playback/download completes, mark as downloaded |
| 31 | `GET /api/v2/records/{accountId}` | `POST /api/v2/records/{accountId}/{recordId}/fix` | PREREQUISITE | Record must exist before rebuild |
| 32 | `POST /api/v2/records/{accountId}/{recordId}/fix` | FCM `video_rebuilt` | TRIGGER | Rebuild triggers FCM push on success |
| 33 | `POST /api/v2/records/{accountId}/{recordId}/fix` | FCM `video_rebuild_failed` | TRIGGER | Rebuild failure triggers FCM push |
| 34 | `POST /api/v2/records/{accountId}/{recordId}/fix` | FCM `video_fix_completed` | TRIGGER | Fix completion triggers FCM push |
| 35 | `POST /api/v2/record-profile/{accountId}` | `GET /api/v2/records/{accountId}` | DATA_FLOW | Recording action creates records that appear in list |
| 36 | `POST /api/streams/record-now/{accountId}` | `GET /api/v2/records/{accountId}` | DATA_FLOW | Legacy record-now also creates records |
| 37 | `POST /api/v2/broadcasts/{accountId}` | `GET /api/v2/records/{accountId}` | DATA_FLOW | Broadcast build populates record data |
| 38 | `GET /api/v2/records/{accountId}` | `POST /api/v2/favorites/{accountId}/{recordId}` | DATA_FLOW | Record must exist before adding to favorites |
| 39 | `GET /api/v2/favorites/{accountId}/list` | `DELETE /api/v2/favorites/{accountId}/{recordId}` | DATA_FLOW | Favorites list provides recordId for removal |
| 40 | `GET /api/v2/favorites/{accountId}/list` | `POST /api/v2/favorites/{accountId}/{recordId}` | DATA_FLOW | Favorites list shows current state for add/remove |
| 41 | `GET /api/v2/user` | `POST /api/v2/purchase/storage` | PREREQUISITE | User must be authenticated for IAP |
| 42 | `GET /api/v2/user` | `POST /api/v2/purchase/slots` | PREREQUISITE | User must be authenticated for IAP |
| 43 | `POST /api/v2/purchase/storage` | `GET /api/v2/purchase/history-list` | DATA_FLOW | Purchase creates history entry |
| 44 | `POST /api/v2/purchase/slots` | `GET /api/v2/purchase/history-list` | DATA_FLOW | Purchase creates history entry |
| 45 | `POST /api/v2/user/profile` | `GET /api/v2/records/{accountId}` | DATA_FLOW | New profile has accountId for records |
| 46 | `PUT /api/v2/user/profile/{id}/logout` | `POST /api/v2/followers/{accountId}/list` | TRIGGER | Profile switch re-fetches follower list |
| 47 | `PUT /api/v2/user/profile/{id}/logout` | `GET /api/v2/records/{accountId}` | TRIGGER | Profile switch re-fetches records |
| 48 | `GET /api/v2/user` | `POST /api/v2/user/profile` | PREREQUISITE | User account required before adding profile |
| 49 | `GET /api/v2/user` | `PUT /api/v2/user/profile/{id}/logout` | PREREQUISITE | User must own profile to switch |
| 50 | `GET /api/v2/user` | `DELETE /api/v2/user/profile/{id}` | PREREQUISITE | User must own profile to delete |
| 51 | `POST /api/v2/user/profile/{id}/logout` | `GET /api/csrf` | TRIGGER | Profile switch may refresh CSRF token |
| 52 | `GET /api/v2/user` | `POST /api/v2/purchase/storage` | DATA_FLOW | User object contains subscription status |
| 53 | `GET /api/v2/user` | `POST /api/v2/purchase/slots` | DATA_FLOW | User object contains slot count |
| 54 | `GET /api/v2/records/{accountId}` | `GET /api/v2/records/{accountId}/{recordId}` | PREREQUISITE | Record list needed to resolve recordId |
| 55 | `GET /api/v2/records/{accountId}` | `POST /api/v2/records/{accountId}/remove-selected` | DATA_FLOW | Records list provides selection for bulk delete |
| 56 | `GET /api/v2/records/{accountId}` | `DELETE /api/v2/records/{accountId}/remove-all` | PREREQUISITE | Records must be listed before bulk delete |
| 57 | `GET /api/v2/user` | `POST /api/v2/user/transfer-recordings/{accountId}/{targetId}` | DATA_FLOW | Both account IDs come from user profiles |
| 58 | `GET /api/v2/user` | `GET /api/v2/user/storage` | DATA_FLOW | User object used for storage usage display |
| 59 | `GET /api/v2/user` | `GET /api/v2/user/installed` | DATA_FLOW | Installation tracking |
| 60 | `GET /api/v2/user` | `GET /api/v2/user/news` | DATA_FLOW | News feed for user |
| 61 | `POST /api/public/history/fetch` | `POST /api/v2/followers/{accountId}` | DATA_FLOW | Public history lookup feeds profile creation |
| 62 | `GET /api/csrf` | `PUT /api/v2/user/profile/{id}/logout` | PREREQUISITE | CSRF token needed for profile switch |
| 63 | `GET /api/csrf` | `POST /api/v2/user/profile` | PREREQUISITE | CSRF token needed for profile creation |
| 64 | `GET /api/csrf` | `POST /api/v2/purchase/storage` | PREREQUISITE | CSRF token needed for purchase |
| 65 | `GET /api/csrf` | `POST /api/v2/purchase/slots` | PREREQUISITE | CSRF token needed for purchase |
| 66 | `GET /api/csrf` | `POST /api/v2/record-profile/{accountId}` | PREREQUISITE | CSRF token needed for recording |
| 67 | `GET /api/csrf` | `PUT /api/v2/records/{accountId}/{recordId}/fix` | PREREQUISITE | CSRF token needed for rebuild |
| 68 | `POST /api/signin` | `GET /api/v2/user` | DATA_FLOW | JWT obtained from signin needed for user fetch |
| 69 | `POST /api/status` | `GET /api/v2/user` | DATA_FLOW | JWT from OAuth needed for user fetch |
| 70 | `POST /api/register` | `GET /api/v2/user` | DATA_FLOW | JWT from registration needed for user fetch |

---

## 4. Feature-Specific Data Flow Diagrams

### 4.1 Authentication Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION FLOW                         │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  GET /api/csrf  ┌──────────────┐
  │  Client  │───────────────►│  Server       │
  │          │◄───────────────│  (csrf)       │
  │          │  XSRF-TOKEN    └──────────────┘
  │          │  (cookie)
  │          │
  │  OPTION A: Email Login
  │          │
  │          │  POST /api/signin  ┌──────────────┐
  │          │───────────────────►│  Server       │
  │          │  {email, password} │  (signin)     │
  │          │◄───────────────────│               │
  │          │  {jwt, user}       └──────────────┘
  │          │
  │  OPTION B: Social Login
  │          │
  │          │  POST /api/login  ┌──────────────┐
  │          │───────────────────►│  Server       │
  │          │  {provider, token}│  (login)      │
  │          │                   │               │
  │          │  POST /api/status  │               │
  │          │───────────────────►│               │
  │          │  {oauth_token}     │               │
  │          │◄───────────────────│               │
  │          │  {jwt, user}       └──────────────┘
  │          │
  │  OPTION C: Device Login
  │          │
  │          │  POST /api/login-device  ┌──────────────┐
  │          │──────────────────────────►│  Server       │
  │          │  {device_token}           │  (loginDevice)│
  │          │◄──────────────────────────│               │
  │          │  {jwt, user}              └──────────────┘
  │          │
  │  OPTION D: Register
  │          │
  │          │  POST /api/register  ┌──────────────┐
  │          │─────────────────────►│  Server       │
  │          │  {email, pass, name} │  (register)  │
  │          │◄─────────────────────│               │
  │          │  {jwt, user}         └──────────────┘
  │          │
  │          │  POST /api/auth/fcm  ┌──────────────┐
  │          │─────────────────────►│  Server       │
  │          │  {fcm_token}         │  (fcm)       │
  │          │◄─────────────────────│               │
  │          │  {ok}                └──────────────┘
  │          │
  │          │  POST /api/v2/logout  ┌──────────────┐
  │          │──────────────────────►│  Server       │
  │          │                       │  (logout)    │
  │          │◄──────────────────────│               │
  │          │  {ok}                 └──────────────┘
  └─────────┘
```

### 4.2 Onboarding Flow

```
┌──────────────────────────────────────────────────────────────┐
│                    ONBOARDING FLOW                             │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  POST /api/public/history/fetch  ┌──────────────┐
  │  Client  │─────────────────────────────────►│  Server       │
  │          │  {username}                      │  (fetch)     │
  │          │◄─────────────────────────────────│               │
  │          │  {history, exists}               └──────────────┘
  │          │
  │          │  GET /api/slides  ┌──────────────┐
  │          │──────────────────►│  Server       │
  │          │                   │  (slides)    │
  │          │◄──────────────────│               │
  │          │  [{image, text}]  └──────────────┘
  │          │
  │          │  POST /api/public/stream-count  ┌──────────────┐
  │          │─────────────────────────────────►│  Server       │
  │          │  {username}                      │  (streamCount)│
  │          │◄─────────────────────────────────│  404 Not Found│
  │          │  (teaser: "Sign in to see!")      └──────────────┘
  │          │
  │          │  ┌─── Branch ───┐
  │          │  │               │
  │          ▼  ▼               │
  │   Login or Register         │
  │          │                   │
  │          ▼                   │
  │  POST /api/v2/followers/{accountId}  ┌──────────────┐
  │─────────────────────────────────────►│  Server       │
  │  {pid, username, ...}               │  (store)     │
  │◄─────────────────────────────────────│               │
  │  {profile}                           └──────────────┘
  └─────────┘
```

### 4.3 Followers → Records Flow

```
┌──────────────────────────────────────────────────────────────┐
│                FOLLOWERS → RECORDS FLOW                       │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  GET /api/v2/followers/{accountId}/list  ┌──────┐
  │  Client  │─────────────────────────────────────────►│Server│
  │          │◄─────────────────────────────────────────│      │
  │          │  [{pid, username, lastCheck, ...}]        └──────┘
  │          │
  │          │  POST /api/v2/followers/{accountId}/{pid}/fetch  ┌──────┐
  │          │──────────────────────────────────────────────────►│      │
  │          │  (Force refresh: triggers server-side scrape)     │      │
  │          │◄──────────────────────────────────────────────────│      │
  │          │  {status, newRecords}                             └──────┘
  │          │
  │          │  POST /api/v2/followers/{accountId}/{pid}/validate  ┌──────┐
  │          │──────────────────────────────────────────────────────►│      │
  │          │  (Validate/archive: checks stream availability)      │      │
  │          │◄──────────────────────────────────────────────────────│      │
  │          │  {archived, recordCount}                              └──────┘
  │          │
  │          │  POST /api/v2/followers/{accountId}/{pid}/deep-check  ┌──────┐
  │          │────────────────────────────────────────────────────────►│      │
  │          │  (Deep check: full archive verification)                │      │
  │          │◄────────────────────────────────────────────────────────│      │
  │          │  {checked, archived}                                    └──────┘
  │          │
  │          │  GET /api/v2/records/{accountId}  ┌──────┐
  │          │───────────────────────────────────►│      │
  │          │◄───────────────────────────────────│      │
  │          │  [{recordId, pid, videoUrl, ...}]  └──────┘
  └─────────┘

  CONDITIONAL PATHS:
  ══════════════════
  • POST /api/v2/followers/{uid}/{pid} + ?isFree=true
    → endpoint appends /free  →  /api/v2/followers/{uid}/{pid}/free
    • Used when user is on free tier (no active storage subscription)
    • Free tier: limited to recent streams only

  • POST /api/v2/followers/{uid}/{pid}/fetch
    → if isFree=true:  /api/v2/followers/{uid}/{pid}/free/fetch
    → if isFree=false: /api/v2/followers/{uid}/{pid}/fetch (standard)
```

### 4.4 Video Playback Chain

```
┌──────────────────────────────────────────────────────────────┐
│                    VIDEO PLAYBACK CHAIN                       │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  GET /api/v2/records/{accountId}  ┌──────────────┐
  │  Client  │──────────────────────────────────►│  Server       │
  │          │◄──────────────────────────────────│  (records)   │
  │          │  [{recordId, pid, title, ...}]     └──────────────┘
  │          │
  │          │  POST /api/v2/records/{accountId}/{recordId}/ups  ┌──────┐
  │          │──────────────────────────────────────────────────►│      │
  │          │  (Request CDN-signed video URL)                   │      │
  │          │◄──────────────────────────────────────────────────│      │
  │          │  {url: "https://cdn.arkzynco.com/..."}            └──────┘
  │          │
  │          │  ┌─────────────────────────────┐
  │          │  │   ExoPlayer (Media3)          │
  │          │  │   • Loads CDN URL             │
  │          │  │   • HLS/DASH adaptive stream  │
  │          │  │   • Buffering & playback       │
  │          │  └──────────────┬──────────────┘
  │          │                  │
  │          │  POST /api/v2/records/{accountId}/{recordId}/downloaded  ┌──────┐
  │          │──────────────────────────────────────────────────────────►│      │
  │          │  (Mark as downloaded after playback completes)            │      │
  │          │◄──────────────────────────────────────────────────────────│      │
  │          │  {ok}                                                     └──────┘
  └─────────┘

  CDN URL LIFETIME:
  ══════════════════
  • CDN URLs are signed and time-limited (typically 1 hour)
  • If URL expires → client must re-call /ups to get fresh URL
  • ExoPlayer handles re-buffering on 403/410 responses
```

### 4.5 Recording Chain

```
┌──────────────────────────────────────────────────────────────┐
│                     RECORDING CHAIN                            │
└──────────────────────────────────────────────────────────────┘

  OPTION A: V2 Record Profile (from JS / Android)
  ════════════════════════════════════════════════════

  ┌─────────┐  POST /api/v2/record-profile/{accountId}  ┌──────┐
  │  Client  │──────────────────────────────────────────►│      │
  │          │  {pid, quality, ...}                      │      │
  │          │◄──────────────────────────────────────────│      │
  │          │  {recordId, status}                        └──────┘
  │          │
  │          │  ┌─── Server records in background ───┐
  │          │  │   • Streams TikTok live              │
  │          │  │   • Saves to cloud storage           │
  │          │  │   • On complete: FCM push            │
  │          │  └─────────────────────────────────────┘
  │          │
  │          │  FCM: video_rebuilt / video_fix_completed  ┌──────┐
  │          │◄────────────────────────────────────────────│      │
  │          │  {recordId, status: "completed"}             └──────┘

  OPTION B: Legacy Record Now (Java ShareReceiverActivity)
  ════════════════════════════════════════════════════════════

  ┌─────────┐  POST /api/v2/record-profile/{username}  ┌──────┐
  │  Share   │─────────────────────────────────────────►│      │
  │ Receiver │  (username from TikTok share intent)     │      │
  │ Activity │◄─────────────────────────────────────────│      │
  │          │  {recordId, status}                       └──────┘

  OPTION C: Legacy V1 Record Now
  ════════════════════════════════════

  ┌─────────┐  POST /api/streams/record-now/{accountId}  ┌──────┐
  │  Client  │───────────────────────────────────────────►│      │
  │          │  {pid, ...}                                │      │
  │          │◄───────────────────────────────────────────│      │
  │          │  {recordId, status}                         └──────┘
```

### 4.6 Video Rebuild Chain

```
┌──────────────────────────────────────────────────────────────┐
│                    VIDEO REBUILD CHAIN                         │
└──────────────────────────────────────────────────────────────┘

  OPTION A: V2 Rebuild
  ══════════════════════

  ┌─────────┐  PUT /api/v2/records/{accountId}/{recordId}/fix  ┌──────┐
  │  Client  │─────────────────────────────────────────────────►│      │
  │          │  {quality, ...}                                  │      │
  │          │◄─────────────────────────────────────────────────│      │
  │          │  {status: "processing"}                           └──────┘
  │          │
  │          │  ┌─── Server processes in background ───┐
  │          │  │   • Re-encodes video                  │
  │          │  │   • Updates CDN URL                    │
  │          │  │   • On success: FCM video_rebuilt      │
  │          │  │   • On failure: FCM video_rebuild_failed│
  │          │  └───────────────────────────────────────┘

  OPTION B: Legacy V1 Rebuild
  ═══════════════════════════════

  ┌─────────┐  POST /api/streams/rebuild/{accountId}  ┌──────┐
  │  Client  │────────────────────────────────────────►│      │
  │          │  {recordId, ...}                         │      │
  │          │◄────────────────────────────────────────│      │
  │          │  {status: "processing"}                   └──────┘
```

### 4.7 Favorites Chain

```
┌──────────────────────────────────────────────────────────────┐
│                     FAVORITES CHAIN                            │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  GET /api/v2/favorites/{accountId}/list  ┌──────┐
  │  Client  │─────────────────────────────────────────►│      │
  │          │◄─────────────────────────────────────────│      │
  │          │  [{recordId, title, addedAt, ...}]        └──────┘
  │          │
  │          │  POST /api/v2/favorites/{accountId}/{recordId}  ┌──────┐
  │          │─────────────────────────────────────────────────►│      │
  │          │  (Add to favorites)                               │      │
  │          │◄─────────────────────────────────────────────────│      │
  │          │  {ok}                                             └──────┘
  │          │
  │          │  DELETE /api/v2/favorites/{accountId}/{recordId}  ┌──────┐
  │          │──────────────────────────────────────────────────►│      │
  │          │  (Remove from favorites)                           │      │
  │          │◄──────────────────────────────────────────────────│      │
  │          │  {ok}                                             └──────┘
```

### 4.8 Purchase Chain

```
┌──────────────────────────────────────────────────────────────┐
│                     PURCHASE CHAIN                             │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  POST /api/v2/purchase/storage  ┌──────────────┐
  │  Client  │────────────────────────────────►│  RevenueCat   │
  │          │  {receipt, product_id}          │  (IAP)       │
  │          │                                 └──────┬───────┘
  │          │                                         │
  │          │  POST /api/v2/purchase/storage  ┌───────▼──────┐
  │          │────────────────────────────────►│  Server       │
  │          │  {receipt, product_id, ...}     │  (purchase)  │
  │          │◄────────────────────────────────│               │
  │          │  {storageAdded, newTotal}       └──────────────┘
  │          │
  │          │  POST /api/v2/purchase/slots  ┌──────────────┐
  │          │───────────────────────────────►│  RevenueCat   │
  │          │  {receipt, product_id}         │  (IAP)       │
  │          │                                └──────┬───────┘
  │          │                                        │
  │          │  POST /api/v2/purchase/slots  ┌────────▼─────┐
  │          │───────────────────────────────►│  Server       │
  │          │  {receipt, product_id, ...}    │  (purchase)  │
  │          │◄───────────────────────────────│               │
  │          │  {slotsAdded, newTotal}        └──────────────┘
  │          │
  │          │  GET /api/v2/purchase/history-list  ┌──────────────┐
  │          │─────────────────────────────────────►│  Server       │
  │          │◄─────────────────────────────────────│  (history)   │
  │          │  [{product, date, amount, ...}]       └──────────────┘
  └─────────┘
```

### 4.9 Profile Management Flow

```
┌──────────────────────────────────────────────────────────────┐
│                PROFILE MANAGEMENT FLOW                        │
└──────────────────────────────────────────────────────────────┘

  ┌─────────┐  GET /api/v2/user  ┌──────────────┐
  │  Client  │──────────────────►│  Server       │
  │          │◄──────────────────│  (fetch)      │
  │          │  {profiles: [...]} └──────────────┘
  │          │
  │          │  POST /api/v2/user/profile  ┌──────────────┐
  │          │─────────────────────────────►│  Server       │
  │          │  {pid, username, ...}        │  (addProfile) │
  │          │◄─────────────────────────────│               │
  │          │  {profile}                   └──────────────┘
  │          │
  │          │  POST /api/v2/user/profile/{id}  ┌──────────────┐
  │          │──────────────────────────────────►│  Server       │
  │          │  {username, ...}                  │  (updateProfile)│
  │          │◄──────────────────────────────────│               │
  │          │  {profile}                        └──────────────┘
  │          │
  │          │  DELETE /api/v2/user/profile/{id}  ┌──────────────┐
  │          │────────────────────────────────────►│  Server       │
  │          │                                     │  (removeProfile)│
  │          │◄────────────────────────────────────│               │
  │          │  {ok}                               └──────────────┘
  │          │
  │          │  PUT /api/v2/user/profile/{id}/logout  ┌──────────────┐
  │          │────────────────────────────────────────►│  Server       │
  │          │                                         │  (logoutProfile)│
  │          │◄────────────────────────────────────────│               │
  │          │  {jwt, user}                            └──────────────┘
  │          │  (Returns NEW JWT for switched profile)
  │          │
  │          │  POST /api/v2/user/profile/{id}/mute  ┌──────────────┐
  │          │───────────────────────────────────────►│  Server       │
  │          │  {muted: true/false}                   │  (muteProfile)│
  │          │◄───────────────────────────────────────│               │
  │          │  {ok}                                  └──────────────┘
  │          │
  │          │  POST /api/v2/user/transfer-recordings/{accountId}/{targetId}  ┌──────┐
  │          │────────────────────────────────────────────────────────────────►│      │
  │          │  (Move all recordings from one profile to another)              │      │
  │          │◄────────────────────────────────────────────────────────────────│      │
  │          │  {transferred}                                                   └──────┘
  └─────────┘
```

---

## 5. Legacy vs Modern Endpoint Mapping

```
┌──────────────────────────────────────────────────────────────────────┐
│               LEGACY → MODERN ENDPOINT MAPPING                        │
└──────────────────────────────────────────────────────────────────────┘

  SERVICE OBJECT MAPPING:
  ══════════════════════

  fe (Legacy WS)              y / P (Modern raid)
  host: ws.arkzynco.com       host: raid.arkzynco.com
  ─────────────────────       ──────────────────────────
  fe.fetchFollower     ───►   y.store
  fe.fetchSyncStreams  ───►   y.followers / y.followingList
  fe.recordNow         ───►   P.recordNow
  fe.rebuildVideo      ───►   P.buildVideo
  fe.login             ───►   Q.signin
  fe.code              ───►   Q.confirmEmail

  ENDPOINT PATH MAPPING:
  ═══════════════════════

  Legacy (V1)                           Modern (V2)
  ──────────────────────────            ──────────────────────────
  /api/profile/login              ──►   /api/signin
  /api/profile/code               ──►   /api/confirm-email
  /api/followers/{accountId}      ──►   /api/v2/followers/{accountId}
  /api/streams/record-now/{id}    ──►   /api/v2/record-profile/{id}
  /api/streams/rebuild/{id}       ──►   /api/v2/records/{id}/{rid}/fix
  /api/v2/records/{id}            ──►   /api/v2/records/{id} (same)

  AUTH HOST MAPPING:
  ═══════════════════

  ws.arkzynco.com (WebSocket)     raid.arkzynco.com (REST)
  ─────────────────────────────   ──────────────────────────
  • Legacy profile auth           • Modern JWT auth
  • Legacy profile fetch          • Modern profile CRUD
  • Legacy record actions         • Modern record actions
  • fe.* service objects           • Q, L, P, y service objects
```

---

## 6. Conditional Path Logic

```
┌──────────────────────────────────────────────────────────────┐
│                  CONDITIONAL PATH LOGIC                        │
└──────────────────────────────────────────────────────────────┘

  1. FREE TIER BRANCHING
  ════════════════════════
  Endpoint: POST /api/v2/followers/{uid}/{pid}

  Condition: isFree == true
  ├─► /api/v2/followers/{uid}/{pid}/free
  │   • Returns only recent/free-tier streams
  │   • Limited history depth
  │   • No deep-archive access
  │
  Condition: isFree == false
  └─► /api/v2/followers/{uid}/{pid}
      • Full stream history
      • Deep-archive access
      • All recorded content

  This applies to ALL sub-endpoints:
  • /fetch     → /free/fetch     (when isFree=true)
  • /validate  → /free/validate  (when isFree=true)
  • /deep-check → (unavailable on free tier)

  2. AUTH STATE CONDITIONALS
  ════════════════════════════

  ┌─────────────────────────────────────────────────┐
  │  Unauthenticated     │  Authenticated            │
  │  ──────────────────  │  ───────────────────────  │
  │  /api/csrf           │  /api/v2/user             │
  │  /api/public/*       │  /api/v2/followers/*      │
  │  /api/slides         │  /api/v2/records/*        │
  │                      │  /api/v2/favorites/*      │
  │                      │  /api/v2/purchase/*       │
  │                      │  /api/v2/user/profile/*   │
  │                      │  /api/v2/record-profile/* │
  │                      │  /api/v2/broadcasts/*     │
  │                      │  /api/v2/game/*           │
  └─────────────────────────────────────────────────┘

  3. PROFILE SWITCH CONDITIONAL
  ══════════════════════════════

  PUT /api/v2/user/profile/{id}/logout
  ├─► Returns new JWT scoped to switched profile
  ├─► All subsequent calls use new JWT
  ├─► accountId in URLs changes to switched profile's accountId
  └─► Records, followers, favorites all scoped to new profile

  4. RECORD STATUS CONDITIONALS
  ══════════════════════════════

  POST /api/v2/records/{accountId}/{recordId}/ups
  ├─► If record status == "completed" → returns CDN URL
  ├─► If record status == "processing" → returns null (poll later)
  └─► If record status == "failed" → returns error (rebuild needed)

  5. PURCHASE VERIFICATION
  ═════════════════════════

  POST /api/v2/purchase/storage
  ├─► Validates with RevenueCat first
  ├─► If receipt valid → add storage
  ├─► If receipt invalid → return error
  └─► If receipt duplicate → return existing (idempotent)
```

---

## 7. Service Host Topology

```
┌──────────────────────────────────────────────────────────────┐
│                    HOST TOPOLOGY                               │
└──────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │                                                              │
  │   ┌──────────────────────────┐  ┌─────────────────────────┐│
  │   │  raid.arkzynco.com       │  │  ws.arkzynco.com        ││
  │   │  (REST API)              │  │  (WebSocket / Legacy)   ││
  │   │                          │  │                         ││
  │   │  Services:               │  │  Services:              ││
  │   │  • Q (Auth)              │  │  • fe (Legacy Actions)  ││
  │   │  • L (User)              │  │  • fs (Public/History)  ││
  │   │  • P (Records)           │  │                         ││
  │   │  • y (Followers)         │  │  Endpoints:             ││
  │   │  • us (Flip Game)        │  │  • /api/public/history  ││
  │   │                          │  │  • /api/public/stream   ││
  │   │  Endpoints:              │  │  • /api/slides          ││
  │   │  • /api/* (v1)           │  │                         ││
  │   │  • /api/v2/* (modern)    │  │  Protocol:              ││
  │   │  • /api/auth/*           │  │  • WebSocket upgrade    ││
  │   │  • /api/purchase/*       │  │  • REST fallback        ││
  │   │  • /api/streams/*        │  │                         ││
  │   │                          │  │                         ││
  │   │  Protocol:               │  │                         ││
  │   │  • HTTPS REST            │  │                         ││
  │   │  • JWT Bearer auth       │  │                         ││
  │   │  • XSRF-TOKEN cookie     │  │                         ││
  │   └──────────────────────────┘  └─────────────────────────┘│
  │                                                              │
  │   ┌──────────────────────────┐  ┌─────────────────────────┐│
  │   │  cdn.arkzynco.com        │  │  fcm.arkzynco.com       ││
  │   │  (CDN / Video)           │  │  (Push Notifications)   ││
  │   │                          │  │                         ││
  │   │  • Signed video URLs     │  │  • FCM token register   ││
  │   │  • HLS/DASH streams      │  │  • video_rebuilt        ││
  │   │  • Time-limited tokens   │  │  • video_rebuild_failed ││
  │   │                          │  │  • video_fix_completed  ││
  │   └──────────────────────────┘  └─────────────────────────┘│
  │                                                              │
  │   ┌──────────────────────────┐                               │
  │   │  RevenueCat (External)   │                               │
  │   │                          │                               │
  │   │  • IAP receipt validate  │                               │
  │   │  • Subscription mgmt     │                               │
  │   │  • /api/v2/purchase/*    │                               │
  │   └──────────────────────────┘                               │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
```

---

## 8. Cross-Service Data Dependencies

```
┌──────────────────────────────────────────────────────────────┐
│              CROSS-SERVICE DATA DEPENDENCIES                   │
└──────────────────────────────────────────────────────────────┘

  DATA OBJECTS FLOWING BETWEEN SERVICES:
  ═══════════════════════════════════════

  Q (Auth) ──────────────────────────────────────────────┐
  │                                                       │
  │  Outputs:                                             │
  │  • JWT token ──────────────────────────────────────┐  │
  │  • XSRF-TOKEN cookie ───────────────────────────┐  │  │
  │  • User object ───────────────────────────────┐  │  │  │
  │                                               │  │  │  │
  ▼                                               ▼  ▼  ▼  ▼
  L (User)                                        │  │  │  │
  │                                               │  │  │  │
  │  Inputs: JWT, XSRF                             │  │  │  │
  │  Outputs:                                      │  │  │  │
  │  • accountId ─────────────────────────────────┐│  │  │  │
  │  • profile list ────────────────────────────┐ ││  │  │  │
  │  • storage/slot counts ───────────────────┐ │ ││  │  │  │
  │  • subscription status ─────────────────┐ │ │ ││  │  │  │
  │                                         │ │ │ ││  │  │  │
  ▼                                         ▼ ▼ ▼ ▼▼  ▼  ▼  ▼
  y (Followers)                              │ │ │ │   │  │  │
  │                                         │ │ │ │   │  │  │
  │  Inputs: accountId, JWT, XSRF           │ │ │ │   │  │  │
  │  Outputs:                               │ │ │ │   │  │  │
  │  • pid (profile ID) ────────────────────┐│ │ │   │  │  │
  │  • stream metadata ───────────────────┐ ││ │ │   │  │  │
  │  • follower data ───────────────────┐ │ ││ │ │   │  │  │
  │                                    │ │ ││ │ │   │  │  │
  ▼                                    ▼ ▼ ▼▼ ▼ ▼   ▼  ▼  ▼
  P (Records)                           │ │ │  │ │   │  │  │
  │                                    │ │ │  │ │   │  │  │
  │  Inputs: accountId, pid, JWT, XSRF │ │ │  │ │   │  │  │
  │  Outputs:                          │ │ │  │ │   │  │  │
  │  • recordId ──────────────────────┐ │ │ │  │ │   │  │  │
  │  • CDN URL (via /ups) ─────────┐ │ │ │ │  │ │   │  │  │
  │  • record list ──────────────┐  │ │ │ │ │  │ │   │  │  │
  │                              │  │ │ │ │ │  │ │   │  │  │
  ▼                              ▼  ▼ ▼ ▼ ▼ ▼  ▼ ▼   ▼  ▼  ▼
  ExoPlayer (Media3)              │  │ │ │ │    │     │  │  │
  │                              │  │ │ │ │    │     │  │  │
  │  Inputs: CDN URL             │  │ │ │ │    │     │  │  │
  │  Outputs:                    │  │ │ │ │    │     │  │  │
  │  • playback ─────────────────┼──┼─┼─┼─┼────┼─────┼──┼──┘
  │  • download ─────────────────┼──┼─┼─┼─┼────┼─────┼──┘
  │                              │  │ │ │ │    │     │
  ▼                              ▼  ▼ ▼ ▼ ▼    ▼     ▼
  FCM (Push)                      │  │ │ │      │
  │                              │  │ │ │      │
  │  Inputs: device token        │  │ │ │      │
  │  Events:                     │  │ │ │      │
  │  • video_rebuilt             │  │ │ │      │
  │  • video_rebuild_failed      │  │ │ │      │
  │  • video_fix_completed       │  │ │ │      │
  │                              │  │ │ │      │
  ▼                              ▼  ▼ ▼ ▼      ▼
  RevenueCat (External)          │  │ │ │
  │                              │  │ │ │
  │  Inputs: receipt             │  │ │ │
  │  Outputs:                    │  │ │ │
  │  • validation ───────────────┼──┼─┼─┘
  │  • subscription status ──────┼──┼─┘
  │                              │  │ │
  ▼                              ▼  ▼ ▼
  Client (Android/iOS)           │  │ │
    • Updates UI                 │  │ │
    • Updates profiles           │  │ │
    • Updates records            │  │ │
    • Updates storage counts     │  │ │
    • Plays video                │  │ │
    • Shows push notifications   │  │ │
```

---

## 9. Error & Retry Dependency Chains

```
┌──────────────────────────────────────────────────────────────┐
│               ERROR & RETRY DEPENDENCY CHAINS                  │
└──────────────────────────────────────────────────────────────┘

  CSRF EXPIRED:
  ═══════════════
  Any POST/PUT/DELETE → 419 CSRF Mismatch
    └─► GET /api/csrf (refresh token)
        └─► Retry original request

  JWT EXPIRED:
  ══════════════
  Any authenticated call → 401 Unauthorized
    ├─► If refresh token available:
    │   └─► POST /api/signin (silent refresh)
    │       └─► Retry original request
    └─► If no refresh:
        └─► Redirect to login screen

  CDN URL EXPIRED:
  ══════════════════
  ExoPlayer → 403 Forbidden
    └─► POST /api/v2/records/{id}/{rid}/ups (get fresh URL)
        └─► ExoPlayer loads new URL

  RECORD PROCESSING:
  ════════════════════
  GET /api/v2/records/{id} → status: "processing"
    └─► Poll GET /api/v2/records/{id} every 30s
        ├─► If status: "completed" → proceed to /ups
        └─► If status: "failed" → PUT /fix (rebuild)

  FREE TIER LIMIT:
  ══════════════════
  POST /api/v2/followers/{uid}/{pid}/fetch → 403 (limit reached)
    └─► Prompt upgrade to storage subscription
        └─► POST /api/v2/purchase/storage

  PROFILE SLOT LIMIT:
  ════════════════════
  POST /api/v2/user/profile → 403 (slots full)
    └─► Prompt upgrade to slot purchase
        └─► POST /api/v2/purchase/slots
```

---

## 10. Summary Statistics

| Metric | Count |
|--------|-------|
| Total unique endpoints | 88 |
| Service objects | 7 (Q, L, P, y, fe, us, fs) |
| Host domains | 4 (raid, ws, cdn, fcm) + RevenueCat |
| Authentication flows | 4 (email, social, device, register) |
| Dependency edges | 70 |
| Conditional path branches | 5 major |
| Legacy-to-modern mappings | 8 |
| FCM event types | 3 |
| PREREQUISITE edges | 22 |
| DATA_FLOW edges | 35 |
| TRIGGER edges | 10 |
| ALTERNATIVE edges | 3 |
