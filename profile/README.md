<div align="center">

# VEKTOR MEMORY

<img width="480" height="270" alt="VEKTOR Memory preview" src="https://github.com/user-attachments/assets/32c4a591-ae27-427d-a221-7fbc5123de31" />

### Local-first memory for AI agents that lasts between sessions.

[![LongMemEval](https://raw.githubusercontent.com/Vektor-Memory/.github/main/profile/badges/longmemeval.svg)](https://vektormemory.com/research)
[![recall latency](https://raw.githubusercontent.com/Vektor-Memory/.github/main/profile/badges/recall.svg)](https://vektormemory.com/research)
[![license](https://raw.githubusercontent.com/Vektor-Memory/.github/main/profile/badges/license.svg)](https://vektormemory.com/product#pricing)
[![MCP tools](https://raw.githubusercontent.com/Vektor-Memory/.github/main/profile/badges/mcp-tools.svg)](https://vektormemory.com/docs/dxt)

**[Download](https://vektormemory.com/downloads) · [Documentation](https://vektormemory.com/docs) · [Pricing](https://vektormemory.com/product#pricing) · [Slipstream README](https://github.com/Vektor-Memory/Vektor-memory)**

</div>

---

## What we build

AI agents forget everything when a session ends. We build the memory layer that lets them carry context forward. Your memories are stored on your own machine, with no telemetry. If you choose a cloud model, the request and any recalled context go to that provider.

## Slipstream

A memory SDK, CLI and MCP server, with a Windows app.

- Memories in local SQLite, embedded on your machine
- Hybrid recall: BM25 and vector search, fused with Reciprocal Rank Fusion
- A five-layer graph: semantic, causal, temporal, entity and procedural links
- Each write is added, updated, deleted or skipped against existing memories

Commercial licence, from $9 a month. Download it from [vektormemory.com](https://vektormemory.com/downloads).

## Cloak

Bundled with Slipstream: SSH with an approval gate and rollback, headless browser fetch, and an AES-256 encrypted credential vault, exposed as MCP tools.

## Open source tools

Apache 2.0. Slipstream itself is not open source.

| Tool | What it does |
|---|---|
| [Vex](https://github.com/Vektor-Memory/Vex) | Migrates memory between vector databases using the `.vmig.jsonl` format |
| [Vek-Sync](https://github.com/Vektor-Memory/Vek-Sync) | Keeps MCP server configs in sync across AI editors |
| [Via](https://github.com/Vektor-Memory/Via) | Shares memory, tasks and context across AI tools |
| [Provenance](https://github.com/Vektor-Memory/Provenance) | Signs and timestamps code, documents and media |
| [Vörwatch](https://github.com/Vektor-Memory/Vorwatch) | Watches a Linux server for signs of compromise, in one bash script |

---

<div align="center">

Benchmark and latency figures are our own measurements; see [method and caveats](https://vektormemory.com/research).

[hello@vektormemory.com](mailto:hello@vektormemory.com) · [Security](https://vektormemory.com/security) · [Privacy](https://vektormemory.com/privacy)

</div>
