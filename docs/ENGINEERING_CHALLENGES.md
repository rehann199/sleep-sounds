# Engineering Challenges

Five problems that matter for a catalog ambient player meant to run for hours on Android. Outcomes describe intended behaviour from the implementation, not measured sleep or engagement metrics.

---

## 1. Large library, small install

**Problem**  
Two dozen long MP3 assets cannot all ship inside the store binary without inflating download size, yet users expect immediate playback after tapping a row.

**Context**  
The catalog is static Dart data mapping each id to a relative audio path. Bytes live on HTTPS object storage; the APK may still bundle copies for packaging or fallback references, but the runtime path prefers disk cache then network.

**Engineering approach**  
- Resolve cache path deterministically from the sound id.
- On hit, load a file source into the single player.
- On miss, attach a streaming URI and start a de-duplicated background download writing through a temp file with size verification.
- Scan existing files at service init to rebuild the in-memory downloaded set.
- Optionally prefetch remaining ids sequentially after startup with throttling.

**Trade-offs**  
First play of an uncached sound needs connectivity unless the user previously downloaded it. Prefetch consumes storage and metered data silently unless the user clears cache.

**Outcome**  
After a successful download, replay works offline for that id. Play attempts without cache and without network fail fast with a user-visible message instead of hanging in a buffering state.

---

## 2. One player, many entry points

**Problem**  
Home cards, library rows, favourites, bedtime reminder taps, mini-player controls and the full-screen player must agree on what is playing while loads race.

**Context**  
Flutter rebuilds asynchronously; `setAudioSource` and `play` complete after navigation events.

**Engineering approach**  
- Hold one shared player view-model for the whole app.
- Increment a play request token before each async load; ignore stale completions.
- Centralise entitlement and connectivity checks in a play helper used by lists and reminders.
- Track whether the player route is already open to avoid duplicate pushes.
- Mirror loading and playing flags for UI spinners and button states.

**Trade-offs**  
Skip previous/next exist only on the full player, not every surface. Mini player shows summary state only.

**Outcome**  
Switching sounds cancels in-flight loads safely. Tapping the same sound toggles pause/resume without reloading.

---

## 3. Loop, finite clips and sleep timer

**Problem**  
Users want either endless repetition or a timed stop with a gentle fade, but encoded files have finite duration and marketing copy suggests multi-hour sessions.

**Context**  
Catalog entries carry a long nominal duration for display; the decoded media file may still end.

**Engineering approach**  
- Expose loop as engine loop-mode one versus off.
- When loop is off, listen for processing-state completed and advance to the next entitled track.
- Make loop and sleep timer mutually exclusive so two stop mechanisms never fight.
- Pause the timer ticker when playback pauses so remaining time reflects listening time intent.
- Apply linear volume fade over the final minute when fade-out setting is on.

**Trade-offs**  
Auto-advance without loop may surprise users expecting silence at clip end; it prioritises continuous ambience over hard stop.

**Outcome**  
Timer expiry stops playback cleanly. Fade reduces abrupt cut-off when enabled. Loop provides true repeat of the same file when timer is off.

---

## 4. Tiered access without server truth

**Problem**  
Free samples, category locks, premium-only tracks, subscription, legacy SKUs and expiring rewarded unlocks must compose without a backend entitlement service.

**Context**  
Play Billing callbacks and local persistence are the source of truth on device.

**Engineering approach**  
- Encode tier rules in catalog flags plus category metadata (including categories with per-sound granularity).
- Persist expiring millisecond timestamps for ad unlock keys.
- Honour legacy one-time flags alongside active subscription state.
- Short-circuit ads when any remove-ads entitlement is true.
- Surface countdown chips in UI while ad unlocks remain valid.

**Trade-offs**  
Client-side gates are simpler to ship but are not tamper-proof; a server validator would be the hardening step.

**Outcome**  
Denied plays never start audio; unlock sheet explains ad or purchase paths. Restore purchases reloads flags from the store plugin.

---

## 5. Ads that fail closed

**Problem**  
Ad formats must be disableable remotely during incidents or policy changes, and a failed config fetch must not default to aggressive monetisation.

**Context**  
Multiple placements (shell banner, player banner, interstitials on navigation, rewarded unlock, app open) share one remote document.

**Engineering approach**  
- Require an explicit successful fetch before any placement returns enabled.
- Treat each placement as off unless its boolean is literally true.
- Respect a global ads-enabled flag in the same document.
- Subscribe to live updates so toggles apply without restart.
- Disable all formats when premium or remove-ads entitlement is active.

**Trade-offs**  
Cold start may show no ads until Firestore responds, reducing revenue in offline-first seconds but avoiding accidental over-serving.

**Outcome**  
Misconfiguration or offline config reads yield zero ad impressions rather than uncontrolled defaults.

---

## Honest limits

These sections describe software trade-offs, not therapeutic efficacy. The app does not validate sleep, does not mix multiple user-selected streams in the shipped UI, and does not assert that fade-out or timers improve rest.
