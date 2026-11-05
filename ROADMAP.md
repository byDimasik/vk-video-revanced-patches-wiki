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

### Wave 2 remainder (COMPLETED 2026-11-05)
- [x] **[Issue #27](https://github.com/byDimasik/vk-video-revanced-patches/issues/27)** ([PR #34](https://github.com/byDimasik/vk-video-revanced-patches/pull/34), merged `d061817`): `feat(patches): implement hide-catalog-banners bytecode patch for CatalogAdBannerDto`
  - Implementation note: the issue assumed a JSON deserializer; the DTO is a plain Parcelable, so the patch short-circuits the real choke points (`CatalogAdBannerDto.b()` slotId getter, `CatalogSectionDto.d()` adBanner accessor) with a full null-safety trace. Also dropped a stale class-name constraint in `instreamAdLoaderFingerprint` (classes obfuscated-rename-drifted in the 1.168 snapshot).
  - 8 e2e tests, 0 skipped, 0 failures. CodeRabbit: 0 threads, clean pass.
- [x] **[Issue #28](https://github.com/byDimasik/vk-video-revanced-patches/issues/28)** ([PR #33](https://github.com/byDimasik/vk-video-revanced-patches/pull/33), merged `b37e09f`): `feat(patches): implement suppress-topshelf resource patch for catalog promo shelves` — CodeRabbit clean first pass, 0 threads.

## Epic #23 Status: ALL SUB-ISSUES COMPLETE (2026-11-05)
Phase 3 fully delivered. Ready for Phase 4 scoping.

### APK 1.168 re-pin (DONE 2026-11-05, [PR #36](https://github.com/byDimasik/vk-video-revanced-patches/pull/36), merged `6571390`)
- `version.mk` re-pinned to the actual upstream APK: **1.168** (`54200`, sha `11cb77d7…`) — `version.mk` remains the single source of truth; e2e snapshot guards follow it (1.165 → 1.168 in both e2e tests).
- `myTargetNativeAdEntryFingerprint` drift fixed: 1.168 parser signature is `a(JSONObject, bp/e, Map)`; strings remain the anchor, stale `parameters()` dropped.
- All 7 tests green against the real 1.168 APK, CI + Release re-verified.

### Remaining follow-ups (backlog candidates, not blockers)
- **Fingerprint drift maintenance**: obfuscated names (`Lmo/d5;` → `Lno/x7;` etc.) shift between APK builds; prefer string/structure anchors over class-name pins where safe.
- **e2e strictness in CI**: `APKTOOL_FRAMEWORK_DIR` must be present for resource decoding in tests; consider provisioning the framework APK in CI or vendoring it.

### Post-epic hotfix (2026-11-05)
- [x] **[PR #35](https://github.com/byDimasik/vk-video-revanced-patches/pull/35)** (merged `6d681dc`): `fix(ci): disambiguate tag push refspec in release workflow` — the first real post-#34 Release run failed because the `main` **tag** now collides with the `main` **branch** (`git push origin main` → "matches more than one"); the workflow now pushes `refs/tags/main`. **Release workflow re-verified green**, rolling release asset refreshed.

## Infrastructure Notes (2026-11-05)
- **Build host**: JDK 17 (openjdk-17-jdk-headless) installed; Android build-tools `d8` at `~/android/build-tools/34.0.0` (executable). Local bundle builds verified end-to-end.
- **Gradle auth**: GitHub Packages (revanced-patcher) requires `export GITHUB_ACTOR=byDimasik GITHUB_TOKEN=$(gh auth token)` before `./gradlew` locally.
- **CI**: `build-and-check` + `Release` workflows on main; rolling release `main` serves the `.rvp` asset.
- **Review pipeline**: CodeRabbit (assertive profile, `.coderabbit.yaml`). Merge rule: all review threads resolved → squash merge (merge commits disabled on repo). Escalation cap: 5 fix rounds.
