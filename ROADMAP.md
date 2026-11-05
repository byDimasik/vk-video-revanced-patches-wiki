# VK Video ReVanced Patches — Project Roadmap & Session State

Canonical state file for stateless session resumption across Gemini Spark and Google Jules runs.

## Current Phase
- **Phase 3 (Active)**: Packaging, Rolling `main` Release, Real-APK Smoke Testing & Catalog Ad Suppression.
- **Target APK Snapshot**: `com.vk.vkvideo` `1.165` (`52786`, SHA-256 `d52c46e0af98a271a21d86d0694000e7718c210c194555c20982650c095cedac`) snapshotted in Jules VM (`/app/.cache/vkvideo-1.165.apk`).
- **Execution model change (2026-11-05)**: Google Jules retired; implementation now driven by OpenClaw agents (model `tkf/deepseek-v4.1` via opencode) with the CodeRabbit review loop: auto-merge when all threads resolved, 5-round cap, human escalation only after cap.

## Completed Phases

### Phase 0 & Phase 1 (Bootstrap & Reconnaissance)
- [x] **Issue #1** ([PR #4](https://github.com/byDimasik/vk-video-revanced-patches/pull/4)): Research Analogue Ad-Blocking Patterns & ReVanced v22 DSL (`research/analogue-patterns.md`).
- [x] **Issue #2** ([PR #6](https://github.com/byDimasik/vk-video-revanced-patches/pull/6)): Bootstrap Repo Structure, ReVanced v22 Gradle Scaffold, Makefile, and CI.
- [x] **Issue #3** ([PR #9](https://github.com/byDimasik/vk-video-revanced-patches/pull/9)): Deterministic `com.vk.vkvideo` APK Acquisition (`make fetch-apk`) & Initial Ad-Surface Recon (`docs/vk-video-ad-surface.md`).
- [x] **Issue #5**: Configure Google Jules, CodeRabbit, and VM Environment Snapshot.

### Phase 2 ([Epic #10](https://github.com/byDimasik/vk-video-revanced-patches/issues/10): Deep Ad-Layer Recon, Test Harness & Core Patches)
- [x] **Issue #11** ([PR #19](https://github.com/byDimasik/vk-video-revanced-patches/pull/19)): `docs(recon): extract in-stream video ad loader and InstreamAd Smali into docs/ad-layer-video-player.md`.
- [x] **Issue #13** ([PR #15](https://github.com/byDimasik/vk-video-revanced-patches/pull/15)): `docs(recon): extract un-elided feed and catalog ad parser Smali into docs/ad-layer-feed-and-banners.md`.
- [x] **Issue #16** ([PR #20](https://github.com/byDimasik/vk-video-revanced-patches/pull/20)): `test(patches): add ReVanced v22.0.1 Smali unit test helper and verified DSL reference`.
- [x] **Issue #17** ([PR #22](https://github.com/byDimasik/vk-video-revanced-patches/pull/22)): `feat(patches): implement disable-video-ads bytecode patch and unit tests`.
- [x] **Issue #18** ([PR #21](https://github.com/byDimasik/vk-video-revanced-patches/pull/21)): `feat(patches): implement hide-feed-ads bytecode patch for MyTargetNativeAdEntry and PromoPost`.

## Active Backlog — [Epic #23](https://github.com/byDimasik/vk-video-revanced-patches/issues/23): Phase 3

### Wave 1 (COMPLETED 2026-11-05 — merged via OpenClaw CodeRabbit loop)
- [x] **[Issue #24](https://github.com/byDimasik/vk-video-revanced-patches/issues/24)** ([PR #29](https://github.com/byDimasik/vk-video-revanced-patches/pull/29), merged `b6421bc`): `build(patches): add .rvp bundle packaging Gradle task and metadata`
- [x] **[Issue #25](https://github.com/byDimasik/vk-video-revanced-patches/issues/25)** ([PR #30](https://github.com/byDimasik/vk-video-revanced-patches/pull/30), merged `ae8c2d5`): `test(e2e): add real APK fingerprint resolution smoke test and PR test template`
  - Regression note: the #30 squash initially dropped the `rvp` task from a stale-base push; restored in [PR #32](https://github.com/byDimasik/vk-video-revanced-patches/pull/32) (merged `f6a55f3`) which also added the d8 DEX step and restored the `vkvideo.e2e` system property.

### Wave 2 (COMPLETED 2026-11-05)
- [x] **[Issue #26](https://github.com/byDimasik/vk-video-revanced-patches/issues/26)** ([PR #31](https://github.com/byDimasik/vk-video-revanced-patches/pull/31), merged `358b1d3`): `ci(release): add rolling main GitHub release workflow for .rvp bundle`
  - Final shape: create-or-update rolling release `main`, asset upload with 3× retry + `--clobber`, `gh release edit --draft=false` publish, stale-run SHA guard, workflow-level concurrency (queued, not cancelled). **Live-verified**: rolling release publishes `vk-video-patches.rvp` containing `classes.dex` + `patch-bundle.json`.

### Wave 2 remainder (IN PROGRESS — dispatched to OpenClaw agents 2026-11-05, CodeRabbit loop)
- [ ] **[Issue #27](https://github.com/byDimasik/vk-video-revanced-patches/issues/27)**: `feat(patches): implement hide-catalog-banners bytecode patch for CatalogAdBannerDto` — dispatched (agent run 75173f4a)
- [ ] **[Issue #28](https://github.com/byDimasik/vk-video-revanced-patches/issues/28)**: `feat(patches): implement suppress-topshelf resource patch for catalog promo shelves` — dispatched (agent run b4d0f0d1)

## Infrastructure Notes (2026-11-05)
- **Build host**: JDK 17 (openjdk-17-jdk-headless) installed; Android build-tools `d8` at `~/android/build-tools/34.0.0` (executable). Local bundle builds verified end-to-end.
- **Gradle auth**: GitHub Packages (revanced-patcher) requires `export GITHUB_ACTOR=byDimasik GITHUB_TOKEN=$(gh auth token)` before `./gradlew` locally.
- **CI**: `build-and-check` + `Release` workflows on main; rolling release `main` serves the `.rvp` asset.
- **Review pipeline**: CodeRabbit (assertive profile, `.coderabbit.yaml`). Merge rule: all review threads resolved → squash merge (merge commits disabled on repo). Escalation cap: 5 fix rounds.
