# ADR-001: Project Bootstrap, Team Topology, and Tooling

* **Status**: Accepted
* **Date**: 2026-10-16
* **Deciders**: Human Lead, Gemini Spark

## 1. Context & Mission

Build, test, and release an open-source [ReVanced Patcher v22](https://github.com/ReVanced/revanced-patcher) patch suite (`.rvp`) for the standalone VK Video Android application (`com.vk.vkvideo`), executed 100% asynchronously at $0 additional infrastructure cost beyond an existing Google AI Pro subscription.

* **Initial Feature Scope**: Ad removal (pre-roll/mid-roll video ads, feed/banner ad injection, sponsored shelf filtering), followed by UI and player cleanup.
* **Strict Prerequisite ("No Code Before Setup")**: Zero patch implementation code is written until repository governance, role boundaries, quota guardrails, licensing, and agent instructions are committed.
* **Artifact Policy**: Release `.rvp` patch bundles only for ReVanced Manager and future ReVanced catalog inclusion. Never distribute raw or patched `.apk` files.

## 2. Multi-Agent Team Topology & Strict Role Boundaries

| Role | Actor / Tool | Execution Environment | Core Responsibilities | Hard Boundaries (Never Do) |
| :--- | :--- | :--- | :--- | :--- |
| **Lead / Architect** | Human Lead | Asynchronous | High-level direction, architectural sign-off, PR merge approval, on-device APK smoke testing. | Never writes boilerplate, manually edits Smali, or babysits synchronous chat sessions. |
| **PM & Researcher** | Gemini Spark | Google Cloud VM + Headless Chrome + GitHub MCP | Async research (studying upstream ReVanced patches), writing specs & docs, decomposing work into small GitHub Issues, labeling issues for Jules, lightweight config/code authoring, orchestrating PR reviews via MCP. | Cannot run Linux CLI Android binaries (`apktool`, `jadx`, `./gradlew`). Delegates binary decompilation and build verification to Jules or GHA. |
| **Software Engineer** | Google Jules (Jules Pro) | Ephemeral Ubuntu VM (Bash, JDK, Gradle, Git, apktool, jadx) | Triggered via `jules` GitHub Issue label. Decompiles target APKs, inspects Smali, writes ReVanced v22 Kotlin DSL patches & unit tests, runs `./gradlew check` locally before opening PRs. | Never works on multi-feature mega-PRs (strictly 1 atomic issue = 1 focused PR). Never commits raw APKs, DEX files, or full decompiled trees. |
| **Independent Reviewer** | CodeRabbit (OSS Pro) | Sandboxed PR Diff & Static Analyzer (Claude/OpenAI + ast-grep) | Automated line-by-line review on every PR. Enforces `.coderabbit.yaml` rules, Kotlin idioms, Smali register safety, and unit test coverage. | Advisory/gating only; does not push commits directly without explicit trigger. |
| **CI/CD & Release** | GitHub Actions (GHA) | Standard Public Ubuntu Runners | Runs deterministic `./gradlew check` (unit tests + linter) on PRs, compiles `.rvp` patch bundles, and publishes signed releases for ReVanced Manager. | No AI reasoning loops inside GHA. Keep workflows deterministic and fast. |

## 3. Technology Stack & Licensing

* **Framework**: [ReVanced Patcher v22.0.1](https://github.com/ReVanced/revanced-patcher/releases/tag/v22.0.1) Kotlin DSL (`bytecodePatch`, `resourcePatch`, `fingerprint`).
* **License**: [GNU General Public License v3.0 (GPLv3)](https://www.gnu.org/licenses/gpl-3.0.en.html), matching the upstream ReVanced ecosystem.
* **Target APK Source**: `com.vk.vkvideo` fetched deterministically via the unauthenticated RuStore API.

## 4. Discussed & Rejected Alternatives (Decision Log)

* **Google Antigravity IDE / CLI (Rejected)**: Does not include top-tier models on the Google AI Pro tier (requires AI Ultra or paid API billing) and requires a desktop environment.
* **GitHub Cloud Copilot Agent & Copilot PR Review (Rejected)**: Paid features ($10+/mo Copilot Pro + Premium Request burn + GHA runner burn). Never invoke `assign_copilot_to_issue`, `create_pull_request_with_copilot`, or `request_copilot_review` tools in GitHub MCP.
* **Private GitHub Repository for Code (Rejected)**: Private repos cap GitHub Actions at 2,000 mins/month and lock CodeRabbit's line-by-line review behind a $24/mo paywall. The code repository (`vk-video-revanced-patches`) must remain Public (OSS) to unlock unlimited $0 GHA minutes and free CodeRabbit Pro reviews.
* **Native GitHub Wiki (Rejected)**: Lacks GitHub REST/MCP API endpoints and PR support; replaced by this dedicated repository (`vk-video-revanced-patches-wiki`).
* **Synchronous Chat Development (Rejected)**: Incompatible with asynchronous workflow. All heavy lifting is dispatched via Spark tasks and atomic Jules GitHub Issues.

## 5. Rate-Limit & Quota Guardrails ("Consumable Pieces" Rule)

* **Jules Pro Limits (100 tasks/day, 15 concurrent VMs)**:
  * Decompose work so each Jules issue takes less than 15 minutes of VM time as an atomic unit.
  * Never combine decompilation, recon, and multi-hook implementation into a single issue. Split into: (1) Decompile & extract ad-related Smali report, (2) Write fingerprint & unit test for Hook A, (3) Implement bytecode mutation for Hook A.
* **CodeRabbit OSS Fair-Use Limits (~3 back-to-back PR burst, 4 PRs/hour, 200 files/hour)**:
  * Exclude generated/decompiled files (`!decompiled/**`, `!**/*.smali`, `!**/*.apk`, `!**/*.rvp`) in `.coderabbit.yaml`.
  * Keep PRs small (fewer than 10 files) and pace PR creation to 3 or fewer PRs per hour.
* **GitHub Actions Public Limits (Unlimited minutes, 20 concurrent jobs)**:
  * Cache Gradle dependencies (`actions/cache` or `gradle/actions/setup-gradle`) and never commit large binaries to git history.
