# Product Overview

Sleep Sounds is an Android ambient-audio application: a catalog of looping recordings (rain, ocean, nature, fans, room ambience and coloured noise) that the user browses, favourites and plays one at a time. This document describes product behaviour from the engineering implementation. It is not medical or sleep advice and does not claim that listening improves sleep, health or mood.

The app’s own onboarding and store copy sometimes uses sleep-oriented language; those strings are product marketing inside the app, not claims made by this portfolio.

## Who it is for

People who want steady background audio on a phone: pick a sound, set volume, optionally arm a countdown that stops playback, and leave the session running with notification controls. The design assumes long sessions and intermittent network quality.

## Platforms

The released build is **Android only** on Google Play (minimum SDK 26). The Flutter project also contains iOS configuration (launcher icons, a tracking-consent dependency, background-audio initialisation), but there is no App Store listing in the release configuration and this showcase does not describe an iOS ship date.

## Navigation

A bottom bar with four roots:

| Tab | Role |
|---|---|
| Home | Greeting, featured carousel, category explore row, popular list, persistent mini player |
| Library | Full catalog with category filter chips, search entry, unlock affordances |
| Favourites | Saved sounds |
| Settings | Playback defaults, reminders, language, theme, storage, legal links, other apps |

Additional full screens (not tabs) include onboarding, a full-screen player, category drill-down from Home, paywall, unlock sheet, language picker, timer and reminder editors, clear-storage breakdown, splash and optional update prompt.

## Catalog

Six categories and **24** sounds defined in a static catalog (counted from the catalog list). Each sound carries title, subtitle, category, and tier flags (free sample, premium-only, or category-gated).

| Category | Examples |
|---|---|
| Rain | Gentle rain, rain on window, heavy rain, rain on roof |
| Ocean | Ocean waves, gentle shore, calm sea, beach waves |
| Nature | Forest, birds, flowing river, rustling leaves |
| Fan | Ceiling, box, desk and wall fan |
| Ambience | Fireplace, café murmur, train ride, airplane cabin |
| Noise | Brown, white, pink and green noise |

Two sounds are marked always-free samples (one rain, one ocean). Two sounds are premium-only (not unlockable by rewarded ads). Rain and ocean use per-sound tiers within the category; other categories unlock as a whole or per sound via ads or purchase.

Audio files are mapped to bundled asset paths for packaging reference, but runtime playback prefers a device cache filled from remote object storage, with HTTPS streaming when the cache is empty.

## Playback (single stream)

**One sound plays at a time.** There is no user-facing control to layer multiple catalog sounds concurrently; playback uses a single streaming engine instance.

From any list or card the user can start or resume the current sound and open the full player. The player shows a category backdrop, transport controls (previous, play or pause, next), a loop toggle, volume, and a sleep-timer control. Skip moves to the next entitled sound in catalog order.

When loop is off and the encoded clip reaches its end, playback automatically advances to the next entitled sound. Loop and an armed sleep timer are mutually exclusive: enabling loop cancels the timer, and arming a timer turns loop off.

Volume is a 0–1 gain on the active player. During the last minute of a running sleep timer, an optional fade-out setting scales volume down linearly until stop.

## Sleep timer

Preset durations appear on the player (off, 15, 30, 45, 60 and 120 minutes). Settings offer a default timer (15–90 minutes in the dedicated timer screen) that can auto-apply when a new sound starts. The countdown ticks only while playback is running; pausing playback freezes the remaining time display. When the timer elapses, playback stops.

## Favourites and search

Favourites persist locally. Search filters the catalog by title from a dedicated route reachable from Home and Library.

## Access and monetisation

Free tier includes ads (subject to remote switches) and limited catalog access. Premium access paths:

- A subscription product (monthly and yearly base plans) that removes ads and unlocks all sounds.
- Legacy one-time purchases still honoured on restore (remove ads, unlock all, bundled premium).
- Rewarded video ads that grant temporary unlock for a category or a single sound, shown with a countdown while active.

Locked taps open an unlock sheet (watch ad or go to paywall). Premium-only sounds skip the ad path.

## Reminders and notifications

An optional **bedtime reminder** schedules local notifications on chosen weekdays and time; tapping can start a configured sound. Playback uses a foreground media notification via the background-audio integration (play/pause/stop affordances, return to player). Push messaging is wired through Firebase Cloud Messaging with topic subscription; notification permission is requested in context.

## Onboarding and guidance

Three onboarding pages after first launch, then a coach-mark tour over Home and Settings (can be replayed). An in-app review prompt and Play in-app update check run from the main shell when appropriate.

## Settings and storage

Settings include default sleep timer, fade-out toggle, bedtime reminder, language, dark mode, privacy options entry when the ads consent SDK requires it, tutorial replay, clear storage, share and rate links, and cross-promotion tiles for other apps from the same publisher list.

Clear storage can reset favourites, timer preferences, mixer volume map (stored but not tied to live multi-track playback), and downloaded audio cache with per-item sizes.

## Languages

Thirteen selectable languages via bundled string tables (English plus twelve translations).

## What this app is not

- Not a sleep tracker, sleep stage analyser or health device.
- Not a multi-track mixer in the shipped UI (only one audible catalog stream).
- Not a guarantee of offline use before each sound has been cached at least once.

For engineering detail see the other documents in this folder.
