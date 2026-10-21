# vk-video-revanced-patches-wiki

Internal agent knowledge base, PM roadmap, and Architecture Decision Records (ADRs) for [`byDimasik/vk-video-revanced-patches`](https://github.com/byDimasik/vk-video-revanced-patches).

## Write Boundaries
* **Gemini Spark (PM)**: Read/Write directly to `main` (`ROADMAP.md`, `decisions/`).
* **Google Jules**: **Read-Only**. All Jules-authored ad-layer reconnaissance and technical docs are committed to `docs/` in [`byDimasik/vk-video-revanced-patches`](https://github.com/byDimasik/vk-video-revanced-patches) (see [ADR-003](decisions/ADR-003-documentation-topology.md)).

## Directory Structure

```text
.
├── README.md
├── ROADMAP.md
├── decisions/
│   ├── ADR-001-project-bootstrap.md
│   ├── ADR-002-target-apk-acquisition-source.md
│   └── ADR-003-documentation-topology.md
└── research/
    ├── README.md
    └── analogue-patterns.md
```

## Decisions Index

* [ADR-001: Project Bootstrap, Team Topology, and Tooling](decisions/ADR-001-project-bootstrap.md)
* [ADR-002: Target APK Acquisition via APKPure API](decisions/ADR-002-target-apk-acquisition-source.md)
* [ADR-003: Documentation Topology & Single-Repo Constraint for Google Jules](decisions/ADR-003-documentation-topology.md)
