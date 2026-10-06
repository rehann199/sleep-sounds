# Technical Case Study

This case study explains how Sleep Sounds implements long-form ambient playback, gated catalog access and mobile platform services on Android. Statements reflect the production Dart codebase structure; nothing here was benchmarked on devices, and no download, retention or sleep-quality figures are claimed.

## 1. Problem shape

The product is a **single-stream loop player** over a catalog that is too large to ship entirely inside the APK. Users expect instant tap-to-play, background continuity, lock-screen controls and graceful behaviour on weak networks. Monetisation mixes subscription, legacy store entitlements, rewarded unlocks and mediated ads that must be remotely switchable without shipping a new binary.

## 2. Application layering

- **Flutter UI** with `provider` view-models for player, timer, entitlements, favourites, settings, theme, locale, onboarding and navigation.
- **Repositories** for preferences and sound lookup (thin wrappers over the static catalog).
- **Services** for downloads, purchases, analytics, notifications, FCM, and Play in-app updates.
- **Cross-cutting modules** for ads (consent, remote config, placement widgets), routing, feedback sounds, and connectivity checks.

Startup initialises Firebase inside guarded timeouts, registers crash handlers, configures a music-category audio session, initialises background notification audio, loads shared preferences, then mounts the widget tree. Heavy work (billing query, download index scan, ad SDK, notifications) is deferred so the first frame is not blocked on network plugins.

## 3. Audio path

### Session and interruptions

The player view-model configures `AudioSession` for music playback and listens for interruption events. A phone call or similar pause interruption saves whether playback was active, pauses, and optionally resumes when the interruption ends with a pause-type end event.

### One engine, one sound

A single `AudioPlayer` instance backs the mini player and full player route. Starting a new sound increments a monotonic play request id so late async completions from an abandoned load are ignored.

Load sequence:

1. Entitlement check; abort if locked.
2. If no local cache and no connectivity, surface an error string and stop.
3. Mark loading, persist last sound id, emit analytics.
4. If switching tracks, stop the engine, build a `MediaItem` tag (title, category as album, fixed artist label), choose file or HTTPS URI source.
5. On set-source failure, retry once with the remote URI.
6. Apply loop mode (single repeat or off), seek to start if the engine reports completed, attach position listeners, set volume (including fade factor), start play.

Loop mode interacts with the sleep timer: loop on forces timer cancel; timer apply forces loop off.

### End-of-clip behaviour

When loop is off and processing state becomes completed, the player flushes listen-duration analytics and skips forward to the next entitled catalog entry. This differs from a pure infinite loop player: clips have finite decoded length even though catalog metadata labels an eight-hour nominal duration for UI copy.

### Volume and fade

User volume persists in preferences. While the sleep timer runs and fade-out is enabled in settings, volume scales down over the final sixty seconds of timer remaining by multiplying the user volume with a linear factor.

### Background controls

Initialisation wraps the app in `JustAudioBackground`, supplying Android notification channel metadata. Tags on each `AudioSource` feed lock-screen and shade metadata. This is lighter than a custom `audio_service` handler but still exposes system transport affordances tied to the one active source.

### UI feedback audio

Short tap clicks use a separate lightweight player so UI sounds do not steal focus from the main ambient stream.

## 4. Cache and network

A singleton download service owns an app-documents subdirectory per relative audio path. At init it scans known catalog ids and records non-empty files.

On play miss it streams remote HTTPS while enqueueing a deduplicated download (temp file, length check, rename). Background prefetch can walk remaining ids sequentially after launch. Clear storage removes cached bytes and resets related preferences.

Connectivity gating applies only when neither cache nor network is available; cached files play offline.

## 5. Entitlements

Entitlement state merges:

- Boolean flags persisted from Play Billing (subscription active, legacy remove-ads / unlock-all flags).
- Expiring map entries for rewarded unlock keys (sound id or category enum name).

Play checks consult free flags, premium-only flags, whole-category free rules, per-category ad unlock expiry, and per-sound ad unlock expiry. Premium subscription implies all sounds and no ads.

Purchase flow uses the standard in-app purchase plugin with Android subscription base plans; errors and cancellations feed analytics and non-fatal crash breadcrumbs.

Entitlement verification is **client-side** against store callbacks and local persistence; there is no server receipt gate in this codebase.

## 6. Sleep timer semantics

Timer state lives in its own view-model with selected minutes, remaining duration, running flag and ticker. Important rules:

- `start` sets remaining to the full selection but does not tick until `resume` (called when playback runs).
- `pause` on playback freezes countdown without clearing armed state.
- Elapsed listeners stop the player when time hits zero.

Default timer minutes from settings auto-apply on `play` when no timer is already running.

## 7. Ads and remote config

Ad placements consult a Firestore-backed document with a global kill switch and per-placement booleans. Until the first fetch succeeds, placements are treated as disabled (fail closed). A listener keeps toggles live.

Premium entitlement short-circuits all formats. Consent management integrates with the mobile ads SDK where required.

## 8. Analytics and listen telemetry

Analytics wraps Firebase Analytics with typed helpers (plays, pauses, timer sets, setting changes, purchase funnel). Listen duration accumulates only during audible segments, flushing when switching sounds, stopping, or finishing a clip naturally, with listened milliseconds and clip length attached.

Performance traces optionally wrap startup phases when Firebase Performance is available.

## 9. Notifications and reminders

Local notifications schedule bedtime reminders with timezone-aware scheduling plugins. FCM registers a background handler and topic subscription for promotional pushes; permission is requested via a dedicated sheet when the product flow requires it.

## 10. Testing surface

Three Dart test files cover widget smoke, download path helpers and language list integrity. This portfolio does not claim CI status or coverage percentages.

## 11. Deliberate non-features

- **No concurrent multi-sound mixing** in playback code: a mixer view-model persists volume levels in preferences but is not wired to any screen or second audio channel.
- **No health integrations** (wearables, sleep APIs, step correlation).
- **No user accounts** for sync; all state is device-local except store and Firebase SDK traffic.

These boundaries keep the engineering story honest: a catalog player with cache, timer, gates and platform services, not a clinical or multi-track DAW.
