<div align="center">

# VEKTOR MEMORY

### Vector memory for agentic AI. Local-first. Privacy Enhanced Technology. Data Sovereignty.
[![Longmemeeval benchmark](https://img.shields.io/badge/longmemeval-81%25-854F0B?style=flat-square)](https://arxiv.org/abs/2605.12493)
[![LoCoMo benchmark](https://img.shields.io/badge/LoCoMo-66.9%25-854F0B?style=flat-square)](https://arxiv.org/abs/2402.17753)
[![recall latency](https://img.shields.io/badge/recall-sub--28ms-0F6E56?style=flat-square)](#performance)
[![license](https://img.shields.io/badge/license-Commercial-533AB7?style=flat-square)](https://vektormemory.com/product#pricing)
[![MCP tools](https://img.shields.io/badge/MCP_tools-50+-185FA5?style=flat-square)](#mcp-tools)

**[Documentation](https://vektormemory.com/docs) · [Install](#install) · [Quick Start](#quick-start) · [MCP Tools](#mcp-tools) · [Pricing](https://vektormemory.com/product#pricing)**

<img width="1877" height="738" alt="banner1" src="https://github.com/user-attachments/assets/2011f985-a584-4c28-aa0f-08c786edd109" />

<div align="center">

<br />

</div>

---

## What we build

AI agents are stateless by default. Every session starts blind — no memory of what was decided, what was tried, or why things are connected. VEKTOR fixes that.

We build memory infrastructure: the storage layer, the recall engine, and the tooling that lets agents carry context across time. Everything runs locally. Nothing leaves your machine.

---

## Our values

**Sovereignty first.** Your agent's memory belongs to you. No telemetry, no cloud sync, no third-party access to the context your agent accumulates. Local SQLite. Your disk. Full stop.

**Determinism over magic.** Memory should behave predictably. We use spec-decoding retrieval, confidence-scored recall, and peer-reviewed benchmarks — not black-box embeddings that silently degrade.

**Open where it counts.** The interchange format is open. The migration tooling is open source. You are never trapped. If you want to move your memory to a different system, we ship the tool to do it.

**Built for agents, not dashboards.** Every design decision optimises for the MCP call path — low latency, structured output, composable tools. Not a UI. Not a SaaS admin panel. A substrate.

---

## Tools

<br />

<div style="text-align:center;padding:2rem 0 1.5rem">
<h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Slipstream</h1>

> *The MCP memory server.*

Slipstream is a 50+ tool MCP server that plugs directly into Claude, Cursor, Windsurf, or any MCP-compatible agent runtime. It gives agents a structured, persistent memory that survives sessions — stored locally in SQLite, recalled in under 1ms via dual-channel BM25 and vector search fused through Reciprocal Rank Fusion.

Memory in Slipstream is not a flat key-value store. It is a self-organising 4-layer graph — episodic, semantic, procedural, and strategic — where facts are linked by Zettelkasten-style edges and weighted by a reinforcement learning scorer that promotes memories which have actually influenced past responses. Over time, the memory surface adapts to what the agent uses.

The intelligence layer adds confidence scoring on every recall, cosine deduplication on write, namespace isolation per project, an import-watch daemon, causal lineage reconstruction on merge, and a briefing scheduler that surfaces a context summary on session start.

Distributed via npm. Commercial licence. $9/month.

[vektormemory.com](https://vektormemory.com) &nbsp;·&nbsp; [npm](https://www.npmjs.com/package/vektor-slipstream) &nbsp;·&nbsp; [Docs](https://vektormemory.com/docs)

<br />

---

<div align="center">

<div style="text-align:center;padding:2rem 0 1.5rem">
<h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Cloak</h1>

> *Hands for your agent. Bundled inside Slipstream.*

Cloak is the DXT & CLI action and operations layer. Where Slipstream manages what an agent knows, Cloak manages what it can do — SSH orchestration, stealth browser automation, and an encrypted credential vault, all exposed as MCP tools.

**SSH orchestration** gives agents the ability to execute commands on remote servers with a mandatory approval gate before any write or destructive operation, multi-command transaction planning, and one-step rollback. Designed for agents that manage infrastructure without human hands on the keyboard.

**Browser automation** runs a fingerprint-spoofed headless browser with human-realistic mouse and scroll behaviour injection, CAPTCHA detection and solving, semantic diffing between fetches of the same URL, and an `llms.txt`-aware smart fetch that avoids the browser entirely when the target is agent-native.

**The vault** is AES-256 encrypted credential storage — API keys, SSH keys, tokens — retrieved by name at tool call time. No plaintext credentials in config files or agent context windows.

Cloak ships inside every Slipstream installation. No separate install.

<br />

---

<div align="center">

<div style="text-align:center;padding:2rem 0 1.5rem">
  <h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Vex</h1>

> *Open source. Free. Apache 2.0.*

Vex is a vector interchange tool. It exports agent memory from VEKTOR and imports it into any major vector database — Pinecone, Qdrant, Weaviate, Chroma, and more — using the open `.vmig.json` interchange format.

It exists because we believe you should be able to leave. Lock-in is a design choice, and we chose against it. If you want to migrate your agent's accumulated memory to a different system, Vex is how you do it.

Zero dependencies. Node.js 18+. Runs on Windows, macOS, and Linux.

[github.com/Vektor-Memory/Vex](https://github.com/Vektor-Memory/Vex)

---

<div align="center">

<div style="text-align:center;padding:2rem 0 1.5rem">
  <h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Vek-Sync</h1>

> *Open source. Free. Apache 2.0.*

Vek-sync is a standalone CLI tool that keeps your MCP (Model Context Protocol) server configurations in sync across all your AI editors — Claude Desktop, Cursor, VS Code, Windsurf, and Claude Code. No account. No cloud. Just a single .mcp.json file and one command.

Define your servers once in .mcp.json. Push to every editor with one command. Pull from any editor to bootstrap the file. Store secrets safely in an encrypted local vault.

https://github.com/Vektor-Memory/Vek-Sync

---
<div align="center">
<div style="text-align:center;padding:2rem 0 1.5rem">
  <h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Provenance</h1>
</div>
</div>

> *Open source. Free. Apache 2.0.*

Provenance is a standalone CLI tool that gives you cryptographic proof of what your code looked like, and when. No account. No cloud. Just your source tree, a Merkle-tree manifest, and two independent timestamp authorities anchoring it in time.

Stamp proprietary headers into every file with `prov stamp`. Snapshot the codebase into a tamper-evident manifest bound to your git commit with `prov manifest`. Anchor that manifest with `prov timestamp` — dual-anchored via RFC 3161 (FreeTSA) and OpenTimestamps (Bitcoin), so no single timestamping authority is a point of failure. Verify any snapshot later with `prov verify`.

https://github.com/Vektor-Memory/Provenance
</div>
</div>
<div align="center">
<div style="text-align:center;padding:2rem 0 1.5rem">
  <h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Vörwatch</h1>
</div>
</div>

> *Open source. Free. Apache 2.0.*
Vörwatch is a standalone CLI tool that watches a Linux server for the signs that usually show up early in a compromise, and tells you what it found. No daemon. No database. No cloud dashboard. Just one bash script, your cron table, and flat files on the box itself.
Capture a known-good baseline with vorwatch baseline. Run a detection pass — file integrity, listening ports, outbound connections, process trees, package vulnerabilities, rootkit signatures, hardening drift — with vorwatch check. Wire it up to run on its own with vorwatch install. Pull a text or JSON summary anytime with vorwatch report, or let it land in your inbox on a schedule. It's recommend-only by design — Vörwatch never bans, blocks, or touches iptables. Every alert tells you the exact command to run yourself.
https://github.com/Vektor-Memory/Vorwatch

</div>
</div>
> *Open source. Free. Apache 2.0.*
<div style="border-left:2px solid var(--color-border-secondary);padding:0 0 0 1rem;margin:0 0 1.5rem">
  <p style="font-size:15px;line-height:1.7;margin:0">Via is the universal integration layer for AI tools. It connects Claude, Cursor, Windsurf, and ChatGPT to a shared memory, task board, and context bus — so your work follows you across every tool, every session, every machine.</p>
</div>


<div style="border-top:0.5px solid var(--color-border-tertiary);padding-top:1.25rem;margin:0 0 1rem">
  <p style="font-size:13px;color:var(--color-text-secondary);margin:0 0 8px;font-weight:500">Architecture</p>
  <p style="font-size:13px;line-height:1.7;color:var(--color-text-primary);margin:0">SQLite locally for everything. No embeddings, no API calls, no cloud. The memory graph is pure SQLite — nodes are files, edges are import relationships. For semantic search and team-shared context, upgrade to <a href="https://vektormemory.com" style="color:var(--color-text-info)">Vektor Slipstream</a>.</p>
</div>

<div style="border-top:0.5px solid var(--color-border-tertiary);padding-top:1.25rem;display:flex;gap:1rem;flex-wrap:wrap;align-items:center">
  <a href="https://github.com/Vektor-Memory/Via" style="font-size:13px;color:var(--color-text-info);text-decoration:none"><i class="ti ti-brand-github" aria-hidden="true">
    
  </i> https://github.com/Vektor-Memory/Via</a>
  </div>
  <span style="font-size:13px;color:var(--color-text-secondary)">Node.js 18+</span>
  <span style="font-size:13px;color:var(--color-text-secondary)">Windows · macOS · Linux</span>
  <span style="font-size:13px;color:var(--color-text-secondary)">Zero runtime dependencies</span>
</div>
</div>
<div align="center">
<div style="text-align:center;padding:2rem 0 1.5rem">
  <h1 style="font-size:22px;font-weight:500;margin:0 0 4px">Provenance</h1>
</div>
</div>


<br />

*Built on peer-reviewed research &nbsp;·&nbsp; Longmemeval 81% - LoCoMo 66.9% &nbsp;·&nbsp; sub-28ms recall*

[hello@vektormemory.com](mailto:hello@vektormemory.com) &nbsp;·&nbsp; [Security](https://vektormemory.com/security) &nbsp;·&nbsp; [Privacy](https://vektormemory.com/privacy)

</div>
