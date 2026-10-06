# Privacy and Source

## What this repository is

A portfolio showcase for a production Android application distributed on Google Play as Sleep Sounds. It describes software engineering for an ambient audio player. It is not a source distribution, not medical or sleep advice, and makes no claims about health outcomes from listening.

## What is published

- Written descriptions of architecture, playback behaviour, design decisions and trade-offs
- Original vector diagrams in `assets/diagrams/`
- Real screenshots from the Android build, cropped to app content (system status bar and ad-placeholder strips removed), captured with sample and placeholder data, metadata removed
- The app icon, resized to WebP in `assets/branding/`
- Static code-size counts stated in the README (Dart files and lines under the app source tree, excluding bundled language tables, plus three test files)

## What is intentionally not published

- Production source code, in whole or in part
- Exact remote URLs, bucket names, object keys and CDN configuration for audio
- Build and signing configuration, certificates and keystores
- Firebase and Google service configuration files, API keys and tokens
- Ad network identifiers, placement identifiers and mediation account details
- Subscription and one-time product identifiers, price points and trial durations
- Internal Firestore collection and field names, security rules and remote-switch schema
- Analytics event names, parameter keys and sampling configuration
- Version control history of the production repository
- User data of any kind

Line, file and test counts are static counts; no test results, performance figures, download numbers, user counts or store ratings are claimed. Diagram labels are conceptual and do not correspond to production identifiers.

## Notes on figures

Counts of Dart files and lines were made against the application source tree under the main library folder, excluding the bundled localisation tables. The catalog size (24 sounds in six categories) was counted from the static catalog definition. These numbers are snapshots for this portfolio, not live metrics.

## About the app's data handling

This repository does not restate or replace the application's privacy policy or the data safety section on the store listing; those documents are the authoritative sources for end users.

At the engineering level, the following properties apply to the described release:

- **Playback and favourites** stay on the device. Favorites, volume, loop preferences, timer settings, ad-unlock expiries and cached audio files use local storage APIs; they are not uploaded as a user profile in the architecture described here.
- **Audio downloads** fetch sound files over HTTPS from remote object storage. Requests expose typical network metadata to the host and any intermediaries; no account is required to stream or cache free-tier sounds.
- **Purchases** interact with Google Play Billing. Transaction state is handled through the store SDK on device; this portfolio does not document server-side receipt storage.
- **Advertisements** use Google's mobile ads SDK with a consent form where required. Ad requests may share device and app identifiers as defined by the SDK and the user's consent choices.
- **Firebase** integrates Analytics, Crashlytics, Performance and Cloud Messaging (including topic subscription). Events and crash reports follow the SDK configurations shipped in the app; this repository does not enumerate which fields are collected or excluded.
- **Remote switches** read small documents from Cloud Firestore to enable or disable ad placements. These reads are not tied to a user account in the described design.
- **Bedtime reminder** schedules local notifications on the device; firing does not require network access.
- **Push messages** may arrive via FCM topics; handling depends on message payload rules implemented in the app.

No claim is made here that all processing is anonymous, that data is encrypted beyond platform defaults, or that the app is compliant with a specific regulation. Those questions belong to the published privacy policy and store disclosures.

## Source code availability

Production source is maintained privately. See [RIGHTS.md](../RIGHTS.md) for usage terms for this showcase repository.
