# Project Roadmap & Session State

Canonical state file for stateless session resumption across Gemini Spark and Google Jules runs.

## Current Phase
- **Phase 2 (Active)**: Deep Ad-Layer Reconnaissance, ReVanced v22.0.1 Test Harness, and Core Ad-Blocking Bytecode Patches.
- **Target APK Snapshot**: `com.vk.vkvideo` `1.165` (`52786`, SHA-256 `d52c46e0af98a271a21d86d0694000e7718c210c194555c20982650c095cedac`) snapshotted in Jules VM (`/app/.cache/vkvideo-1.165.apk`).

## Completed Phases (Phase 0 & Phase 1)
- [x] **Issue #1** ([PR #4](https://github.com/byDimasik/vk-video-revanced-patches/pull/4)): Research Analogue Ad-Blocking Patterns & ReVanced v22 DSL (`research/analogue-patterns.md`).
- [x] **Issue #2** ([PR #6](https://github.com/byDimasik/vk-video-revanced-patches/pull/6)): Bootstrap Repo Structure, ReVanced v22 Gradle Scaffold, Makefile, and CI.
- [x] **Issue #3** ([PR #9](https://github.com/byDimasik/vk-video-revanced-patches/pull/9)): Deterministic `com.vk.vkvideo` APK Acquisition (`make fetch-apk`) & Initial Ad-Surface Recon (`docs/vk-video-ad-surface.md`).
- [x] **Issue #5**: Configure Google Jules, CodeRabbit, and VM Environment Snapshot.

## Active Backlog — [Epic #10](https://github.com/byDimasik/vk-video-revanced-patches/issues/10): Phase 2

### Wave 1 (Dispatched to Google Jules — Labeled `jules`, 100% Disjoint Files)
- [ ] **[Issue #11](https://github.com/byDimasik/vk-video-revanced-patches/issues/11)**: `docs(recon): extract in-stream video ad loader and InstreamAd Smali into docs/ad-layer-video-player.md`
- [ ] **[Issue #13](https://github.com/byDimasik/vk-video-revanced-patches/issues/13)**: `docs(recon): extract un-elided feed and catalog ad parser Smali into docs/ad-layer-feed-and-banners.md`
- [ ] **[Issue #16](https://github.com/byDimasik/vk-video-revanced-patches/issues/16)**: `test(patches): add ReVanced v22.0.1 Smali unit test helper and verified DSL reference`

### Wave 2 (Queued — Apply `jules` Label After Wave 1 Merges)
- [ ] **[Issue #17](https://github.com/byDimasik/vk-video-revanced-patches/issues/17)**: `feat(patches): implement disable-video-ads bytecode patch and unit tests` (Blocked by #11, #16)
- [ ] **[Issue #18](https://github.com/byDimasik/vk-video-revanced-patches/issues/18)**: `feat(patches): implement hide-feed-ads bytecode patch for MyTargetNativeAdEntry and PromoPost` (Blocked by #13, #16)

## Next Planned Phase (Phase 3 — Post-Wave 2)
- Catalog banner (`CatalogAdBannerDto`) and top-shelf layout (`catalog_video_topshelf`, `catalog_video_music_topshelf`) suppression patches.
- `.rvp` patch bundle release packaging and GitHub Releases workflow.
