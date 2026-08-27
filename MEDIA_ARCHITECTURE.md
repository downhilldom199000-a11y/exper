# TKREC Media Architecture

> Generated: 2026-08-26
> Comprehensive analysis of media handling, video playback, CDN delivery, and storage

---

## Table of Contents

1. [Media Asset Inventory](#1-media-asset-inventory)
2. [Video Playback Pipeline](#2-video-playback-pipeline)
3. [Thumbnail System Architecture](#3-thumbnail-system-architecture)
4. [Download Pipeline](#4-download-pipeline)
5. [CDN and Delivery Architecture](#5-cdn-and-delivery-architecture)
6. [Storage Management](#6-storage-management)
7. [Rebuild / Fix Pipeline](#7-rebuild--fix-pipeline)
8. [Ad Integration](#8-ad-integration)
9. [Real-Time Event System](#9-real-time-event-system)
10. [Caching Strategy](#10-caching-strategy)
11. [Bandwidth Analysis](#11-bandwidth-analysis)
12. [Security Considerations](#12-security-considerations)

---

## 1. Media Asset Inventory

### 1.1 Media Types Overview

| Type | Source | Storage | Auth Required | Format |
|------|--------|---------|---------------|--------|
| Thumbnails | TikTok CDN | Remote (CDN) | Bearer token | JPEG/WebP |
| Profile Pictures | TikTok CDN | Remote (CDN) | Bearer token | JPEG/PNG |
| Video Recordings | TikTok CDN (dynamic) | Remote (CDN) | Bearer token + EDT | MP4 (H.264/H.265) |
| Onboarding Slides | TKREC API | Remote (server) | None | JPEG/PNG |
| CMS Pages | TKREC API | Remote (server) | None | HTML |
| App Assets | Bundled | Local (APK) | None | PNG, JSON, etc. |

### 1.2 Thumbnail URLs

```
Base CDN Pattern:
  https://p16-common-sign.tiktokcdn-us.com/tos-useast8-p-*

Variants:
  - *-c516 (low-res thumbnail, ~5-10 KB)
  - *-c720 (full-res thumbnail, ~30-80 KB)
```

**Origin**: TikTok's internal CDN distribution network (`p16-common-sign` is TikTok's image signing domain).

**Storage location**: `record.pic` and `profile.pic` fields contain CDN URLs returned by the API.

### 1.3 Profile Pictures

```
Stored in:  record.pic  (per-record creator avatar)
            profile.pic (user's own profile)
CDN Pattern: Same TikTok CDN as thumbnails
Resolution:  ~150x150 to 720x720
```

### 1.4 Video Recording URLs

```
Resolution: Dynamic — fetched per-playback via POST /ups
Protocol:   HTTPS (TCP or QUIC/UDP)
CDN:        AWS NLB / Cloudflare edge
Codec:      H.264 (AVC) or H.265 (HEVC)
Container:  MP4
```

Video URLs are **ephemeral** — they are generated on demand through the `/ups` endpoint and cannot be used across sessions.

### 1.5 Onboarding Slides

```
Endpoint:  GET /api/slides
Response:  [{ id: string, img: string (CDN URL), url: string (link target) }]
```

These are promotional/educational images served to new users during first-run onboarding.

### 1.6 CMS Content

```
Endpoint:  GET /api/pages/{slug}
Response:  { content: "HTML" }
Use:       Help pages, terms of service, FAQ, etc.
```

---

## 2. Video Playback Pipeline

### 2.1 End-to-End Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        VIDEO PLAYBACK PIPELINE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────────────┐   │
│  │  User UI  │───▶│  Framework7  │───▶│   WebView (Chromium)     │   │
│  │  (Vue.js) │    │  + Vue.js    │    │   150.0.7871.181         │   │
│  └──────────┘    └──────────────┘    └────────────┬─────────────┘   │
│                                                     │                 │
│                                        ┌────────────▼─────────────┐  │
│                                        │   HTML5 <video> Element   │  │
│                                        │   src = CDN_URL           │  │
│                                        └────────────┬─────────────┘  │
│                                                     │                 │
│  ┌──────────────────────────────────────────────────▼─────────────┐  │
│  │              Android Media3 ExoPlayer (Native)                  │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌───────────────────────┐ │  │
│  │  │ CCodecBuffers│  │ AidlBufferPool│  │ HW Video Decoder      │ │  │
│  │  │ (video/raw)  │  │ (19.6 MB)    │  │ video-scaling = 1     │ │  │
│  │  └─────────────┘  └──────────────┘  └───────────────────────┘ │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                   Native Java Layer                             │  │
│  │  VideoPlayerActivity.java                                       │  │
│  │    └─ Intercepts video requests → Adds Authorization: Bearer   │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                      │
│                         ┌──────────────┐                             │
│                         │  TikTok CDN   │                             │
│                         │  (CDN_URL)   │                             │
│                         └──────────────┘                             │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Playback Sequence

```
User taps play on record
         │
         ▼
┌─────────────────────┐
│ playVideo(record)   │
│  (Vue.js component) │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐     ┌─────────────────────────┐
│  Ad Check            │────▶│  Show interstitial ad    │
│  (subscriber?)       │ YES │  or rewarded video ad    │
└────────┬────────────┘     └─────────────────────────┘
         │ NO (subscribed)
         ▼
┌─────────────────────┐
│  Determine quality   │
│  (settings or auto)  │
└────────┬────────────┘
         │
         ▼
┌──────────────────────────────────────────────────────────┐
│  POST /api/v2/records/{accountId}/{recordId}/ups         │
│  Body: {                                                  │
│    "edt": "base64(JSON({e: eventType, t: unixTime}))"  │
│    "quality": "720p"                                     │
│  }                                                        │
│  Response: { video: "https://CDN_URL" }                  │
└────────┬─────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────┐
│  Native Java Layer   │
│  VideoPlayerActivity │
│  ├─ Adds Auth header │
│  └─ Passes URL to    │
│     WebView <video>  │
└────────┬────────────┘
         │
         ▼
┌──────────────────────────────────────────────────────────┐
│  Chromium WebView                                         │
│  <video src="CDN_URL" autoplay />                        │
│  └─ Hardware decode via Media3 ExoPlayer                  │
│     ├─ CCodecBuffers (video/raw)                         │
│     ├─ AidlBufferPool (19.6 MB allocation)               │
│     └─ video-scaling=1 (hardware scaling)                │
└──────────────────────────────────────────────────────────┘
```

### 2.3 EDT Token (Enhanced Delivery Token)

```
EDT = Base64Encode(JSON({
    "e": eventType,    // Event type code
    "t": unixTimestamp  // Current Unix timestamp
}))

Purpose: CDN access authorization
Format:  Base64-encoded JSON payload
Expiry:  Short-lived (timestamp-based)
Sent in: POST /ups request body
```

### 2.4 Native Java Components

| File | Role |
|------|------|
| `VideoPlayerActivity.java` | Intercepts video requests, injects `Authorization: Bearer` header into WebView requests |
| `ThumbnailImageLoader.java` | Glide-based loader with Bearer auth for authenticated thumbnail fetches |
| `ShareReceiverActivity.java` | Receives TikTok share intents, calls `POST /api/v2/record-profile/{username}` |

---

## 3. Thumbnail System Architecture

### 3.1 Loading Pipeline

```
┌────────────────────────────────────────────────────────────────┐
│                    THUMBNAIL LOADING PIPELINE                    │
├────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────┐                                              │
│  │ Record List UI  │                                              │
│  │ (Vue.js scroll) │                                              │
│  └───────┬────────┘                                              │
│          │                                                        │
│          ▼                                                        │
│  ┌────────────────────────────────────┐                          │
│  │ IntersectionObserver (lazy load)    │                          │
│  │  ├─ Enters viewport → trigger load │                          │
│  │  ├─ Adjacent preload (1-2 ahead)   │                          │
│  │  └─ Priority: visible > preload    │                          │
│  └───────┬────────────────────────────┘                          │
│          │                                                        │
│          ▼                                                        │
│  ┌────────────────────────────────────┐                          │
│  │ ThumbnailImageLoader.java (Native) │                          │
│  │  ├─ Glide library (image pipeline) │                          │
│  │  ├─ Authorization: Bearer <token>  │                          │
│  │  ├─ Memory cache (L1)              │                          │
│  │  ├─ Disk cache (L2)                │                          │
│  │  └─ Fallback: /no-profile-1.3.0.jpg│                          │
│  └───────┬────────────────────────────┘                          │
│          │                                                        │
│          ▼                                                        │
│  ┌────────────────────┐                                          │
│  │  TikTok CDN         │                                          │
│  │  p16-common-sign.*  │                                          │
│  └────────────────────┘                                          │
│                                                                  │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 IntersectionObserver Configuration

```
Threshold:     0 (triggers as soon as any pixel is visible)
Root Margin:   "200px" (preload 200px before entering viewport)
Preload Count: 1-2 adjacent thumbnails ahead
Priority:      Visible items > preload candidates
```

### 3.3 Error Handling

```
Load attempt
    │
    ├── Success → Display thumbnail
    │
    └── Failure → Retry (1-2 attempts)
                     │
                     └── Final failure → Fallback image
                                          /no-profile-1.3.0.jpg
```

### 3.4 Glide Configuration

```java
// ThumbnailImageLoader.java (conceptual)
Glide.with(context)
    .load(cdnUrl)
    .apply(new RequestOptions()
        .override(targetWidth, targetHeight)
        .diskCacheStrategy(DiskCacheStrategy.ALL)
        .placeholder(R.drawable.placeholder)
        .error(R.drawable.no_profile))
    .into(imageView);

// Auth header injection via OkHttpGlideModule
// Adds: Authorization: Bearer <stored_token>
```

---

## 4. Download Pipeline

### 4.1 Download Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                       DOWNLOAD PIPELINE                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│  User taps download                                               │
│         │                                                          │
│         ▼                                                          │
│  ┌─────────────────────────┐                                       │
│  │  Subscription Check      │                                       │
│  │  ├─ Subscribed + instant │──▶ Highest quality auto-selected     │
│  │  └─ Not subscribed       │──▶ Quality picker dialog             │
│  └────────┬────────────────┘                                       │
│           │                                                        │
│           ▼                                                        │
│  ┌─────────────────────────┐     ┌─────────────────────┐          │
│  │  Rewarded Ad             │────▶│  User watches ad     │          │
│  │  (non-subscribers)       │     │  → unlock download   │          │
│  └────────┬────────────────┘     └─────────────────────┘          │
│           │                                                        │
│           ▼                                                        │
│  ┌──────────────────────────────────────────────────┐             │
│  │  POST /api/v2/records/{accountId}/{recordId}/ups │             │
│  │  { "edt": "...", "quality": "720p" }             │             │
│  │  Response: { video: "https://CDN_URL" }          │             │
│  └────────┬─────────────────────────────────────────┘             │
│           │                                                        │
│           ▼                                                        │
│  ┌─────────────────────────┐                                       │
│  │  Disk Space Check        │                                       │
│  │  Requires: 1.5x video   │                                       │
│  │  size + 2GB minimum      │                                       │
│  └────────┬────────────────┘                                       │
│           │                                                        │
│           ▼                                                        │
│  ┌─────────────────────────────────────────────────┐             │
│  │  Capacitor saveVideo()                           │             │
│  │  {                                               │             │
│  │    url: "https://CDN_URL",                       │             │
│  │    album: "TKREC",                               │             │
│  │    filename: "<record_id>.mp4",                  │             │
│  │    extension: ".mp4"                             │             │
│  │  }                                               │             │
│  └────────┬────────────────────────────────────────┘             │
│           │                                                        │
│           ▼                                                        │
│  ┌─────────────────────────┐                                       │
│  │  Android Download Manager│                                       │
│  │  (native HTTP download)  │                                       │
│  └────────┬────────────────┘                                       │
│           │                                                        │
│           ▼                                                        │
│  ┌─────────────────────────────────────────────────┐             │
│  │  Progress Tracking                               │             │
│  │  downloadProgress: {                              │             │
│  │    bytesDownloaded: number,                       │             │
│  │    totalBytes: number                             │             │
│  │  }                                               │             │
│  │  Status flow:                                     │             │
│  │  STATUS_PENDING → STATUS_RUNNING                  │             │
│  │      → STATUS_SUCCESSFUL | STATUS_FAILED          │             │
│  └─────────────────────────────────────────────────┘             │
│                                                                    │
└──────────────────────────────────────────────────────────────────┘
```

### 4.2 Quality Selection Logic

```
┌──────────────────────────────────────────────────────┐
│                 QUALITY DECISION TREE                  │
├──────────────────────────────────────────────────────┤
│                                                        │
│  Is user subscribed?                                   │
│  ├─ YES ──▶ Is instantDownload enabled?               │
│  │          ├─ YES ──▶ Auto-select highest available   │
│  │          └─ NO  ──▶ Show quality picker            │
│  │                                                        │
│  └─ NO  ──▶ Show quality picker (216p/360p only)      │
│              └─ User selects ──▶ Watch rewarded ad     │
│                                   └─ Unlock download   │
│                                                        │
└──────────────────────────────────────────────────────┘
```

### 4.3 Quality Levels

| Quality | Resolution | Typical Size (1hr) | Subscriber Only | Bitrate (est.) |
|---------|------------|---------------------|-----------------|----------------|
| 216p | 384 x 216 | ~50 MB | No | ~1.1 Mbps |
| 360p | 640 x 360 | ~100 MB | No | ~2.2 Mbps |
| 480p | 854 x 480 | ~200 MB | Yes | ~4.4 Mbps |
| 720p | 1280 x 720 | ~500 MB | Yes | ~11 Mbps |

### 4.4 Disk Space Requirements

```
Minimum free space = 1.5 x video file size + 2 GB

Example (720p, 1hr video):
  1.5 x 500 MB + 2 GB = 2.75 GB minimum

The 2 GB floor ensures the OS and app caches
remain functional after the download completes.
```

---

## 5. CDN and Delivery Architecture

### 5.1 Observed CDN Endpoints

| IP | Port | Protocol | Service | Purpose |
|----|------|----------|---------|---------|
| `172.67.219.78` | 443 | UDP/QUIC | Cloudflare | WebSocket (Pusher), real-time broadcast |
| `3.163.248.4` | 443 | UDP/QUIC | AWS NLB | Media delivery (video, thumbnails) |
| `172.217.171.35` | 443 | TCP | Google | API services, FCM |
| `172.217.171.46` | 443 | UDP/QUIC | Google | Services (GMS, ads) |

### 5.2 Network Topology

```
┌─────────────────────────────────────────────────────────────────────┐
│                      CDN & DELIVERY TOPOLOGY                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│                        ┌──────────────┐                              │
│                        │  TKREC App    │                              │
│                        │  (Android)    │                              │
│                        └──────┬───────┘                              │
│                               │                                       │
│              ┌────────────────┼────────────────┐                     │
│              │                │                │                      │
│              ▼                ▼                ▼                      │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│   │  Cloudflare    │  │  AWS NLB      │  │  Google       │             │
│   │  172.67.219.78 │  │  3.163.248.4  │  │  172.217.171. │             │
│   │  UDP/QUIC:443  │  │  UDP/QUIC:443 │  │  TCP/443      │             │
│   ├──────────────┤  ├──────────────┤  ├──────────────┤             │
│   │ WebSocket      │  │ Video CDN     │  │ API Gateway   │             │
│   │ (Pusher)       │  │ Thumbnails    │  │ FCM Push      │             │
│   │ Broadcast      │  │ Media assets  │  │ AdMob         │             │
│   │ status events  │  │ Downloads     │  │ GMS Services  │             │
│   └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                      │
│   ┌──────────────────────────────────────────────────────────────┐  │
│   │  TikTok CDN (p16-common-sign.tiktokcdn-us.com)                │  │
│   │  ├─ Thumbnail origin (tos-useast8-p-*)                        │  │
│   │  ├─ Profile picture origin                                    │  │
│   │  └─ Signed URLs with expiration                               │  │
│   └──────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.3 Protocol Selection

```
QUIC/UDP (preferred):
  ├─ Faster connection establishment (0-RTT)
  ├─ Built-in congestion control
  ├─ Connection migration (network switches)
  └─ Used for: video streams, thumbnail batches, WebSocket

TCP (fallback):
  ├─ Guaranteed delivery
  ├─ Better compatibility with proxies
  └─ Used for: API calls, FCM, ad requests
```

---

## 6. Storage Management

### 6.1 User-Facing Storage

| Tier | Storage Quota | Video Quality Access | Instant Download |
|------|--------------|---------------------|------------------|
| Free | 5 GB | 216p, 360p | No |
| Pro (Subscription) | Unlimited* | All (up to 720p) | Configurable |

*Subject to fair use policy.

### 6.2 Storage Usage Tracking

```
Endpoint:  GET /api/v2/records/{accountId}/storage-usage
Returns:   { used: number, total: number, unit: "bytes" }
Update:    Real-time on upload/download/delete
Display:   Progress bar in settings UI
```

### 6.3 Local Caching Hierarchy

```
┌─────────────────────────────────────────────────────┐
│              STORAGE HIERARCHY                        │
├─────────────────────────────────────────────────────┤
│                                                       │
│  Tier 1: Vuex Store (RAM)                             │
│  ├─ User data, settings, record metadata              │
│  ├─ Session-scoped (lost on app restart)              │
│  └─ Size: ~1-5 MB typical                             │
│                                                       │
│  Tier 2: localStorage                                 │
│  ├─ Auth token, onboarding state, dark theme          │
│  ├─ Persistent across sessions                        │
│  └─ Size: ~50-100 KB                                  │
│                                                       │
│  Tier 3: WebView Cache (Chromium HTTP Cache)          │
│  ├─ Cached thumbnails, static assets                  │
│  ├─ LRU eviction                                      │
│  └─ Size: ~20 MB max                                  │
│                                                       │
│  Tier 4: Session Storage (per-tab)                    │
│  ├─ Archive validation polls                          │
│  ├─ Active download state                             │
│  └─ Size: ~100 KB - 1 MB                              │
│                                                       │
│  Tier 5: Disk (downloaded videos)                     │
│  ├─ Saved to device gallery via Capacitor             │
│  ├─ Album: "TKREC"                                    │
│  └─ Size: 50-500 MB per video                         │
│                                                       │
└─────────────────────────────────────────────────────┘
```

---

## 7. Rebuild / Fix Pipeline

### 7.1 Rebuild Trigger

```
POST /api/v2/records/{accountId}/{recordId}/fix

Purpose: Request server-side rebuild of a recording
         (e.g., corrupted processing, encoding failure)
```

### 7.2 Rebuild State Machine

```
                    ┌──────────┐
                    │   NONE   │ (initial state)
                    └────┬─────┘
                         │ POST /fix
                         ▼
                    ┌──────────┐
                    │ PENDING  │ (queued on server)
                    └────┬─────┘
                         │ Server begins processing
                         ▼
                    ┌──────────────┐
                    │  PROCESSING  │ (actively rebuilding)
                    └───┬──────┬───┘
                        │      │
              Success   │      │  Failure
                        ▼      ▼
              ┌──────────┐  ┌──────────┐
              │ COMPLETED │  │  FAILED  │
              └──────────┘  └──────────┘
```

### 7.3 FCM Events for Rebuild

| Event | Meaning |
|-------|---------|
| `video_rebuilt` | Rebuild completed successfully |
| `video_rebuild_failed` | Rebuild failed (server error) |
| `video_fix_completed` | Fix request processed |
| `updateRecord` | Record metadata updated with new video URL |

### 7.4 Rebuild Flow

```
1. User/Admin triggers rebuild
         │
         ▼
2. POST /fix → Server queues rebuild job
         │
         ▼
3. Server processes (encoding, transcoding)
         │
         ├──▶ FCM: video_rebuilt ──▶ updateRecord
         │       └─ Record updated with new CDN URL
         │
         └──▶ FCM: video_rebuild_failed
                 └─ User notified, retry option shown
```

---

## 8. Ad Integration

### 8.1 AdMob Configuration

```
Ad Unit ID:    ca-app-pub-7501332842730537/7935013498
Format:        Rewarded Video
Provider:      Google AdMob
Consent:       Google UMP (User Messaging Platform)
```

### 8.2 Ad Flow

```
┌──────────────────────────────────────────────────────────┐
│                     AD INTEGRATION FLOW                    │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─────────────────┐                                       │
│  │  App Start        │                                       │
│  │  (or new session) │                                       │
│  └────────┬────────┘                                       │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────────────┐                                │
│  │  Google UMP Consent      │                                │
│  │  (GDPR/privacy check)    │                                │
│  └────────┬────────────────┘                                │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────────────────────────────┐               │
│  │  User action: Play or Download video     │               │
│  └────────┬────────────────────────────────┘               │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────────────┐     ┌──────────────────┐      │
│  │  Is user subscribed?     │     │  Show rewarded     │      │
│  ├─ YES ──▶ Skip ad        │     │  video ad          │      │
│  └─ NO  ──▶ Show ad ──────▶│────▶│  (ca-app-pub-...)  │      │
│                              │     └────────┬─────────┘      │
│                              │              │                 │
│                              │              ▼                 │
│                              │  ┌──────────────────┐        │
│                              │  │  User watches     │        │
│                              │  │  complete ad      │        │
│                              │  └────────┬─────────┘        │
│                              │           │                   │
│                              │           ▼                   │
│                              │  ┌──────────────────┐        │
│                              │  │  Unlock:          │        │
│                              │  │  - Play video     │        │
│                              │  │  - Download video │        │
│                              │  └──────────────────┘        │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

### 8.3 Consent Management (Google UMP)

```
1. App init → Check UMP consent status
2. If unknown → Present consent form
3. If consented → Load ads normally
4. If denied → No ads, show upgrade prompt
5. Consent stored in: localStorage / UMP SDK
```

---

## 9. Real-Time Event System

### 9.1 Event Transport

TKREC uses **two distinct real-time channels**:

| Channel | Transport | Protocol | Purpose |
|---------|-----------|----------|---------|
| FCM (Firebase Cloud Messaging) | TCP/HTTPS | HTTP/2 | Server → Client push events |
| Pusher/WebSocket | UDP/QUIC (Cloudflare) | WebSocket | Broadcast status updates |

### 9.2 FCM Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `recording_start` | Recording session begins | `{recordId, accountId}` |
| `recording_finished` | Recording session ends | `{recordId, accountId, duration}` |
| `video_ready` | Video processing complete | `{recordId, videoUrl}` |
| `video_rebuilt` | Rebuild completed | `{recordId, videoUrl}` |
| `video_rebuild_failed` | Rebuild failed | `{recordId, error}` |
| `video_fix_completed` | Fix request processed | `{recordId, status}` |
| `updateRecord` | Record metadata changed | `{recordId, changes}` |

### 9.3 Event Flow Diagram

```
┌────────────────────────────────────────────────────────────────┐
│                   REAL-TIME EVENT ARCHITECTURE                   │
├────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────┐         ┌──────────────────────────────┐   │
│  │  TKREC Server   │         │  Firebase Cloud Messaging     │   │
│  │  (Backend)      │────────▶│  (FCM)                        │   │
│  └────────────────┘         └──────────────┬───────────────┘   │
│                                             │                    │
│                                             │ Push notification  │
│                                             ▼                    │
│                                  ┌──────────────────────┐       │
│                                  │  Android FCM Service   │       │
│                                  │  (FirebaseMessaging    │       │
│                                  │   Service)             │       │
│                                  └──────────┬───────────┘       │
│                                             │                    │
│                                             ▼                    │
│                                  ┌──────────────────────┐       │
│                                  │  App State Router      │       │
│                                  │  ├─ Foreground → Event │       │
│                                  │  ├─ Background → Notif │       │
│                                  │  └─ Killed → System    │       │
│                                  └──────────────────────┘       │
│                                                                  │
│  ┌────────────────┐         ┌──────────────────────────────┐   │
│  │  TKREC Server   │         │  Cloudflare Edge              │   │
│  │  (WebSocket)    │────────▶│  172.67.219.78:443 (QUIC)     │   │
│  └────────────────┘         └──────────────┬───────────────┘   │
│                                             │                    │
│                                             ▼                    │
│                                  ┌──────────────────────┐       │
│                                  │  Pusher WebSocket     │       │
│                                  │  (client channel)     │       │
│                                  │  └─ Broadcast events  │       │
│                                  └──────────────────────┘       │
│                                                                  │
└────────────────────────────────────────────────────────────────┘
```

### 9.4 Channel Routing

```
FCM (primary):
  ├─ Recording lifecycle events
  ├─ Video processing status
  ├─ Record updates
  └─ Push notifications (when app backgrounded/killed)

WebSocket/Pusher (secondary):
  ├─ Broadcast status updates
  ├─ Real-time connection health
  └─ Presence/online indicators
```

---

## 10. Caching Strategy

### 10.1 Cache Layers

```
┌──────────────────────────────────────────────────────────────────┐
│                       CACHING ARCHITECTURE                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Layer 1: In-Memory (Vuex Store)                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Contents: user data, settings, records list, auth state    │  │
│  │  Eviction: App restart (session-scoped)                     │  │
│  │  Size: ~1-5 MB                                              │  │
│  │  TTL: Session lifetime                                      │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Layer 2: localStorage                                           │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Contents: auth token, onboarding flag, dark theme pref     │  │
│  │  Eviction: Manual clear only                                │  │
│  │  Size: ~50-100 KB                                           │  │
│  │  TTL: Persistent until user clears or app uninstall         │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Layer 3: WebView HTTP Cache (Chromium)                          │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Contents: thumbnails, static assets, API responses         │  │
│  │  Eviction: LRU (20 MB max)                                  │  │
│  │  Size: ~20 MB                                               │  │
│  │  TTL: Server-controlled (Cache-Control headers)             │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Layer 4: Session Storage (per-tab)                              │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Contents: archive validation polls, active download state  │  │
│  │  Eviction: Tab close                                        │  │
│  │  Size: ~100 KB - 1 MB                                       │  │
│  │  TTL: Tab lifetime                                          │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  Layer 5: Disk (Downloaded Files)                                │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  Contents: downloaded MP4 videos, saved images              │  │
│  │  Eviction: Manual delete or storage full                    │  │
│  │  Size: Up to 5 GB (free) / unlimited (Pro)                 │  │
│  │  TTL: Persistent until user deletes                         │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                    │
└──────────────────────────────────────────────────────────────────┘
```

### 10.2 Cache Invalidation

```
Vuex Store:
  ├─ On login: refresh all user data
  ├─ On logout: clear all
  ├─ On record update: patch affected record
  └─ On FCM updateRecord: sync changes

localStorage:
  ├─ Token: replaced on each successful auth
  ├─ Onboarding: set once, cleared on logout
  └─ Theme: updated on preference change

WebView Cache:
  ├─ Automatic: server Cache-Control headers
  ├─ Force refresh: on new app version
  └─ Manual: user-triggered clear cache
```

---

## 11. Bandwidth Analysis

### 11.1 Per-Activity Bandwidth

| Activity | Data per Event | Events/Day | Total (Daily) |
|----------|---------------|------------|---------------|
| App startup | ~50 KB | 5 | 250 KB |
| Profile sync | ~100 KB | 10 | 1 MB |
| Record list fetch | ~200 KB | 20 | 4 MB |
| Thumbnail load | ~10 KB | 100 | 1 MB |
| Video playback (1hr) | ~50-500 MB | 1 | 50-500 MB |
| Download (1hr video) | ~50-500 MB | 1 | 50-500 MB |
| FCM heartbeat | ~1 KB | 100 | 100 KB |

### 11.2 User Profiles

```
┌──────────────────────────────────────────────────────────────┐
│                    BANDWIDTH PROFILES                         │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  Light User (browse only, no video):                           │
│  ├─ API calls:     ~5 MB/day                                  │
│  ├─ Thumbnails:    ~1 MB/day                                  │
│  ├─ No video/play: 0 MB                                       │
│  └─ Total:         ~6 MB/day                                  │
│                                                                │
│  Medium User (1-2 videos, no download):                        │
│  ├─ API calls:     ~5 MB/day                                  │
│  ├─ Thumbnails:    ~2 MB/day                                  │
│  ├─ Video:         ~100-300 MB/day                             │
│  └─ Total:         ~105-305 MB/day                             │
│                                                                │
│  Heavy User (watch + download):                                │
│  ├─ API calls:     ~5 MB/day                                  │
│  ├─ Thumbnails:    ~2 MB/day                                  │
│  ├─ Video:         ~100-500 MB/day                             │
│  ├─ Download:      ~100-500 MB/day                             │
│  └─ Total:         ~200-1000 MB/day                            │
│                                                                │
│  Monthly estimates:                                            │
│  ├─ Light:   ~180 MB/month                                    │
│  ├─ Medium:  ~3-9 GB/month                                    │
│  └─ Heavy:   ~6-30 GB/month                                   │
│                                                                │
└──────────────────────────────────────────────────────────────┘
```

### 11.3 Bandwidth Optimization

```
Optimizations in place:
  ├─ Lazy thumbnail loading (IntersectionObserver)
  │   └─ Only visible + 1-2 adjacent loaded
  ├─ Thumbnail preloading (200px margin)
  │   └─ Prepares next thumbnails before scroll
  ├─ Quality selection (user-controlled)
  │   └─ Lower quality = less bandwidth
  ├─ QUIC/UDP transport
  │   └─ Faster handshake, better loss recovery
  ├─ WebView HTTP cache (20 MB)
  │   └─ Avoids re-fetching thumbnails/assets
  └─ FCM push (not polling)
      └─ Event-driven, no heartbeat polling
```

---

## 12. Security Considerations

### 12.1 Authentication on Media

```
┌──────────────────────────────────────────────────────────────────┐
│                    MEDIA SECURITY LAYERS                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Layer 1: Bearer Token Authentication                             │
│  ├─ All media requests include: Authorization: Bearer <token>    │
│  ├─ Token injected by VideoPlayerActivity.java (native)          │
│  ├─ Token injected by ThumbnailImageLoader.java (Glide/OkHttp)  │
│  └─ Token: JWT or opaque token from login                        │
│                                                                    │
│  Layer 2: EDT (Enhanced Delivery Token)                           │
│  ├─ Required for video CDN access                                 │
│  ├─ Base64(JSON({e: eventType, t: unixTimestamp}))              │
│  ├─ Short-lived (timestamp-based expiry)                         │
│  ├─ Unique per request (not reusable across sessions)            │
│  └─ Generated server-side via POST /ups                          │
│                                                                    │
│  Layer 3: CDN Signing (TikTok)                                    │
│  ├─ Thumbnail URLs contain: p16-common-sign (signed URL)         │
│  ├─ URLs expire after TTL                                         │
│  ├─ Signed at origin (TikTok CDN)                                │
│  └─ Cannot be guessed/replayed                                   │
│                                                                    │
│  Layer 4: Protocol Security                                       │
│  ├─ HTTPS/TLS 1.3 on all connections                             │
│  ├─ QUIC/UDP with TLS 1.3 (0-RTT with replay protection)        │
│  ├─ Certificate pinning (Android Network Security Config)        │
│  └─ No HTTP fallback allowed                                     │
│                                                                    │
└──────────────────────────────────────────────────────────────────┘
```

### 12.2 Token Flow

```
┌──────────────┐
│  Login API    │──▶ Returns: JWT/auth token
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  localStorage │──▶ Stores token persistently
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Vuex Store   │──▶ Caches token in memory
└──────┬───────┘
       │
       ├──────────────────────────────────────────────┐
       │                                              │
       ▼                                              ▼
┌────────────────────┐                 ┌────────────────────────┐
│  Native Java Layer  │                 │  WebView Requests       │
│  ├─ VideoPlayerAct. │                 │  ├─ Cookie-based auth   │
│  │  Authorization:  │                 │  ├─ Or header injection │
│  │  Bearer <token>  │                 │  └─ Chromium handles    │
│  └─ ThumbnailLoader │                 │     TLS/QUIC            │
│     Authorization:  │                 └────────────────────────┘
│     Bearer <token>  │
└────────────────────┘
       │
       ▼
┌──────────────┐
│  CDN Server   │──▶ Validates: Bearer + EDT + Signed URL
└──────────────┘
```

### 12.3 Security Weak Points

```
Potential concerns:
  ├─ localStorage token storage (XSS-vulnerable in WebView)
  ├─ EDT token replay within short TTL window
  ├─ No DRM on downloaded MP4 files (user has full file)
  ├─ Bearer token transmitted in HTTP headers (visible in proxy)
  ├─ WebView cache may store sensitive responses
  ├─ No certificate pinning enforcement check observed
  └─ FCM token could be intercepted for spoofed notifications
```

### 12.4 Data Sensitivity Map

| Data | Sensitivity | Protection |
|------|------------|------------|
| Auth token | HIGH | Bearer header, HTTPS, short TTL |
| EDT token | HIGH | Base64, short-lived, per-request |
| Video CDN URL | MEDIUM | Ephemeral, requires auth |
| Thumbnail URL | LOW | Signed CDN, public-ish |
| Profile data | MEDIUM | Bearer auth on fetch |
| Downloaded video | LOW | Local file, no DRM |
| FCM token | MEDIUM | Firebase SDK, app-bound |
| localStorage data | HIGH | XSS-vulnerable, persistent |

---

## Appendix A: Key Source Files

| File | Role in Media Pipeline |
|------|----------------------|
| `VideoPlayerActivity.java` | Native video player, auth header injection |
| `ThumbnailImageLoader.java` | Glide-based thumbnail loading with auth |
| `ShareReceiverActivity.java` | TikTok share intent handling |
| `record-profile` endpoint caller | Share-to-record sync |

## Appendix B: API Endpoints Referenced

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/v2/records/{accountId}/{recordId}/ups` | Get video CDN URL |
| POST | `/api/v2/records/{accountId}/{recordId}/fix` | Trigger rebuild |
| GET | `/api/v2/records/{accountId}/storage-usage` | Check storage |
| GET | `/api/slides` | Onboarding slides |
| GET | `/api/pages/{slug}` | CMS content |
| POST | `/api/v2/record-profile/{username}` | Share-to-record |

## Appendix C: Ad Unit Reference

| Unit | ID | Format |
|------|----|--------|
| Rewarded Video | `ca-app-pub-7501332842730537/7935013498` | Rewarded interstitial |

---

*End of analysis.*
