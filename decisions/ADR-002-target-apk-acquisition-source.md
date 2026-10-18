# ADR-002: Target APK Acquisition via APKPure API

## Status
Accepted (partially supersedes ADR-001)

## Context
ADR-001 specified RuStore API (`backapi.rustore.ru`) for automated target APK retrieval (`com.vk.vkvideo`).
RuStore enforces strict CIS/RU regional IP blocks, causing automated downloads to fail in standard cloud CI and Google Jules VMs.
Spike of APKMirror failed: Cloudflare Turnstile bot challenges and ephemeral session tokens block automated headless curl downloads.
Spike of APKPure succeeded: mobile CMS endpoint (`https://api.pureapk.com/m/v3/cms/app_version`) serves direct CDN APK links headlessly without geo-restrictions or captcha challenges.

## Decision
Switch target APK acquisition in `make fetch-apk` from RuStore to APKPure mobile API.

Implementation contract:
1. Target package: `com.vk.vkvideo`.
2. Endpoint: `https://api.pureapk.com/m/v3/cms/app_version?hl=en-US&package_name=com.vk.vkvideo`.
3. Required headers: `x-sv: 29`, `x-abis: arm64-v8a,armeabi-v7a,armeabi`, `x-gp: 1`.
4. Storage: `.cache/vkvideo-<version>.apk` (gitignored).
5. Version tracking: `version.mk` in repo root.

## Consequences
* Positive: automated headless downloads execute globally in CI and Jules without proxy or browser emulation.
* Positive: retains identical `make fetch-apk` interface and caching behavior for developers.
* Negative: third-party mirror dependency; if APKPure alters mobile API schemas, download script requires header or endpoint updates.
