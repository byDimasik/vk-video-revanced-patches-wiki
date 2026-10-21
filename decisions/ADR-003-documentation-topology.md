# ADR-003: Documentation Topology & Single-Repo Constraint for Google Jules

* **Status**: Accepted
* **Date**: 2026-10-20
* **Deciders**: Human Lead, Gemini Spark

## Context
Google Jules executes each GitHub Issue inside an isolated VM cloned from a single target repository (`byDimasik/vk-video-revanced-patches`) and can open only one Pull Request per task. Instructing Jules to commit reconnaissance or technical documentation to a second repository (`byDimasik/vk-video-revanced-patches-wiki`) while also opening a PR in the main code repository either fails or is rejected by Jules as a multi-task request.

## Decision
1. **Main Repository (`byDimasik/vk-video-revanced-patches/docs/`)**:
   - All Jules-authored technical documentation, APK ad-layer Smali reconnaissance (`docs/vk-video-ad-surface.md`, `docs/ad-layer-*.md`), and DSL/testing reference docs (`docs/revanced-v22-dsl-reference.md`) live in `docs/` inside the main repository.
   - Parallel recon tasks must write to dedicated per-layer files (`docs/ad-layer-video-player.md`, `docs/ad-layer-feed-and-banners.md`) rather than editing a single shared file to eliminate merge conflicts across concurrent Wave PRs.
2. **Wiki Repository (`byDimasik/vk-video-revanced-patches-wiki`)**:
   - **Write-enabled for Gemini Spark (PM) only** (direct commits to `main` via GitHub MCP `push_files` for `ROADMAP.md` and `decisions/ADR-*.md`).
   - **Strictly read-only for Google Jules**.

## Consequences
* **Positive**: Every Jules task produces a single self-contained PR in `byDimasik/vk-video-revanced-patches` Reviewed by CodeRabbit.
* **Positive**: Disjoint `docs/ad-layer-*.md` files allow 3 concurrent Wave 1 Jules tasks to merge cleanly without rebase conflicts.
