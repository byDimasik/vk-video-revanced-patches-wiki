# vk-video-revanced-patches-wiki

Internal agent knowledge base, research repository, and Architecture Decision Records (ADRs) for [`byDimasik/vk-video-revanced-patches`](https://github.com/byDimasik/vk-video-revanced-patches).

## Why This Is a Separate Repository (Not Native GitHub Wiki)

1. **No GitHub REST/GraphQL API for Native Wikis**: GitHub does not expose native repository wikis (`*.wiki.git`) through the Contents or Git Trees REST APIs. Hosting the wiki as a standard Git repository allows Model Context Protocol (MCP) tools (`get_file_contents`, `push_files`, `search_code`) to read and write documentation directly.
2. **Pull Request & Branch Support**: Native GitHub Wikis do not support branches or Pull Requests. A standard repository allows autonomous agents (Google Jules, Gemini Spark) to propose, review, and version documentation updates alongside code workflows.
3. **Private Operational Scope**: Keeps internal agent coordination, reconnaissance notes, and operational guardrails separate from the public patch repository.

## Directory Structure

```text
.
├── README.md
├── ROADMAP.md
├── decisions/
│   └── ADR-001-project-bootstrap.md
└── research/
    └── README.md
```

## Index

* [Project Roadmap & Session State](ROADMAP.md)
* **Decisions (`decisions/`)**
  * [ADR-001: Project Bootstrap, Team Topology, and Tooling](decisions/ADR-001-project-bootstrap.md)
* **Research (`research/`)**
  * [Research Directory Index](research/README.md)
