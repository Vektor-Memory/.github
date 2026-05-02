<div align="center">

# VEKTOR MEMORY

### Persistent memory for AI agents. Local-first. No cloud. No amnesia.

[![npm version](https://img.shields.io/npm/v/vektor-slipstream?style=flat-square&color=0F6E56&label=npm)](https://www.npmjs.com/package/vektor-slipstream)
[![npm downloads](https://img.shields.io/npm/dw/vektor-slipstream?style=flat-square&color=185FA5&label=downloads)](https://www.npmjs.com/package/vektor-slipstream)
[![LoCoMo benchmark](https://img.shields.io/badge/LoCoMo-66.9%25-854F0B?style=flat-square)](https://arxiv.org/abs/2402.17753)
[![recall latency](https://img.shields.io/badge/recall-sub--1ms-0F6E56?style=flat-square)](#performance)
[![license](https://img.shields.io/badge/license-Commercial-533AB7?style=flat-square)](https://vektormemory.com/product#pricing)
[![MCP tools](https://img.shields.io/badge/MCP_tools-44-185FA5?style=flat-square)](#mcp-tools)

**[Documentation](https://vektormemory.com/docs) · [Install](#install) · [Quick Start](#quick-start) · [MCP Tools](#mcp-tools) · [Pricing](https://vektormemory.com/product#pricing)**

</div>

---

<img width="1877" height="738" alt="banner1" src="https://github.com/user-attachments/assets/2011f985-a584-4c28-aa0f-08c786edd109" />
<div align="center">
<br />
VEKTOR MEMORY
Persistent memory infrastructure for AI agents.
Local-first. Sovereign. No cloud. No amnesia. No lock-in.
Show Image
Show Image
Show Image
Show Image
Show Image
vektormemory.com  ·  Docs  ·  Downloads  ·  Medium
<br />
</div>

What we build
AI agents are stateless by default. Every session starts blind — no memory of what was decided, what was tried, or why things are connected. VEKTOR fixes that.
We build memory infrastructure: the storage layer, the recall engine, and the tooling that lets agents carry context across time. Everything runs locally. Nothing leaves your machine.

Our values
Sovereignty first. Your agent's memory belongs to you. No telemetry, no cloud sync, no third-party access to the context your agent accumulates. Local SQLite. Your disk. Full stop.
Determinism over magic. Memory should behave predictably. We use spec-decoding retrieval, confidence-scored recall, and peer-reviewed benchmarks — not black-box embeddings that silently degrade.
Open where it counts. The interchange format is open. The migration tooling is open source. You are never trapped. If you want to move your memory to a different system, we ship the tool to do it.
Built for agents, not dashboards. Every design decision optimises for the MCP call path — low latency, structured output, composable tools. Not a UI. Not a SaaS admin panel. A substrate.

Products
<br />
Slipstream

The MCP memory server.

Slipstream is a 44-tool MCP server that plugs directly into Claude, Cursor, Windsurf, or any MCP-compatible agent runtime. It gives agents a structured, persistent memory that survives sessions — stored locally in SQLite, recalled in under 1ms via dual-channel BM25 and vector search fused through Reciprocal Rank Fusion.
Memory in Slipstream is not a flat key-value store. It is a self-organising 4-layer graph — episodic, semantic, procedural, and strategic — where facts are linked by Zettelkasten-style edges and weighted by a reinforcement learning scorer that promotes memories which have actually influenced past responses. Over time, the memory surface adapts to what the agent uses.
The intelligence layer adds confidence scoring on every recall, cosine deduplication on write, namespace isolation per project, an import-watch daemon, causal lineage reconstruction on merge, and a briefing scheduler that surfaces a context summary on session start.
Distributed via npm. Commercial licence. $9/month.
vektormemory.com  ·  npm  ·  Docs
<br />

Cloak

Hands for your agent. Bundled inside Slipstream.

Cloak is the action and operations layer. Where Slipstream manages what an agent knows, Cloak manages what it can do — SSH orchestration, stealth browser automation, and an encrypted credential vault, all exposed as MCP tools.
SSH orchestration gives agents the ability to execute commands on remote servers with a mandatory approval gate before any write or destructive operation, multi-command transaction planning, and one-step rollback. Designed for agents that manage infrastructure without human hands on the keyboard.
Browser automation runs a fingerprint-spoofed headless browser with human-realistic mouse and scroll behaviour injection, CAPTCHA detection and solving, semantic diffing between fetches of the same URL, and an llms.txt-aware smart fetch that avoids the browser entirely when the target is agent-native.
The vault is AES-256 encrypted credential storage — API keys, SSH keys, tokens — retrieved by name at tool call time. No plaintext credentials in config files or agent context windows.
Cloak ships inside every Slipstream installation. No separate install.
<br />

Vex

Open source. Free. Apache 2.0.

Vex is a vector interchange tool. It exports agent memory from VEKTOR and imports it into any major vector database — Pinecone, Qdrant, Weaviate, Chroma, and more — using the open .vmig.json interchange format.
It exists because we believe you should be able to leave. Lock-in is a design choice, and we chose against it. If you want to migrate your agent's accumulated memory to a different system, Vex is how you do it.
Zero dependencies. Node.js 18+. Runs on Windows, macOS, and Linux.
github.com/Vektor-Memory/Vex

<div align="center">
<br />
Built on peer-reviewed research  ·  LoCoMo 66.9%  ·  sub-1ms recall
hello@vektormemory.com  ·  Security  ·  Privacy
</div>
