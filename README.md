# Research Summary: Graphify.net

**Date:** April 27, 2026  
**Task:** Research graphify.net architecture, capabilities, and self-hosting requirements  
**Status:** ✅ Complete

---

## Key Findings

### 1. What is Graphify? ✅

**Graphify** is an open-source Python tool that transforms unstructured files (code, docs, papers, images, audio, video) into interactive knowledge graphs queryable via AI assistants.

- **Official Repository:** https://github.com/safishamsi/graphify
- **PyPI Package:** `graphifyy` (note: double-y)
- **License:** Apache 2.0
- **Latest Version:** v0.5.0 (as of April 2026)

**Use Cases:**
- Understand large codebases faster
- Find architectural relationships
- Integrate documentation with code
- Analyze papers and extract key concepts
- Maintain shared knowledge graphs across teams

### 2. Architecture Overview ✅

Graphify uses a **3-pass deterministic pipeline**:

```
Input Files (code, docs, images, video, audio)
    ↓
Pass 1: AST Extraction (tree-sitter, LOCAL, no LLM)
    └─ Classes, functions, imports, call graphs, docstrings
    ↓
Pass 2: Video/Audio Transcription (Whisper, LOCAL, cached)
    └─ Transcribed locally - audio never leaves your machine
    ↓
Pass 3: LLM Semantic Extraction (Claude/GPT, your API key)
    └─ Concepts, relationships, design rationale (parallel agents)
    ↓
Build NetworkX Graph
    ↓
Community Detection (Leiden algorithm, topology-based)
    ↓
Output: HTML graph, JSON structure, Markdown report, (optional Obsidian/Neo4j/Wiki)
```

**Key Architecture Properties:**
- **No embeddings:** Community detection uses graph topology, not vector embeddings
- **No vector database:** All relationships already in NetworkX graph
- **Deterministic:** AST extraction always produces same output
- **Cached:** SHA256 hashing prevents re-processing unchanged files
- **Local-first:** Code and audio never leave machine; docs/images use your API credentials

### 3. Self-Hosting: Fully Supported ✅

Graphify is **completely self-hosted** - runs on your Linux/Mac/Windows machine.

**System Requirements:**
- Python 3.10-3.13
- ~200MB disk for base installation
- Optional: GPU for faster Whisper transcription (CPU fine)
- No server or cloud infrastructure needed

**Installation (3 ways):**
```bash
# Best: uv (isolated, automatic PATH)
uv tool install graphifyy

# Alternative: pipx (isolated per tool)
pipx install graphifyy

# Fallback: pip (standard Python)
pip install graphifyy
```

### 4. Input Types Supported ✅

| Category | Types | Processing |
|----------|-------|-----------|
| **Code** | Python, JS/TS, Go, Rust, Java, C/C++, Ruby, C#, Kotlin, Scala, PHP, Swift, Lua, Zig, PowerShell, Elixir, Julia, Verilog, Vue, Svelte, Dart | Tree-sitter AST (local) |
| **Docs** | Markdown, HTML, reStructuredText, Plain text | Claude extraction |
| **Papers** | PDF | Citation + concept mining |
| **Images** | PNG, JPG, WebP, GIF | Claude vision |
| **Video/Audio** | MP4, MOV, MKV, WebM, AVI, MP3, WAV, M4A, OGG, YouTube | Whisper (local) |
| **Office** | DOCX, XLSX | Convert to markdown, extract |

### 5. Knowledge Base Integration ✅

**Export Formats:**
- **Obsidian Vault:** Structured markdown with bidirectional links
- **Neo4j:** Cypher export or direct push to Neo4j instance
- **Wiki:** Wikipedia-style articles per community
- **JSON:** Persistent queryable graph (`graph.json`)
- **HTML:** Interactive visualization with search/filter
- **GraphML:** Import to Gephi, yEd, or other tools
- **SVG:** Export as image

**Always-On Integration:**
- **Claude Code:** PreToolUse hooks + CLAUDE.md instructions
- **Codex:** Hooks in `.codex/hooks.json`
- **Cursor:** Rules in `.cursor/rules/graphify.mdc`
- **GitHub Copilot CLI:** Skill in `~/.copilot/skills/`
- **Aider, OpenClaw, Factory Droid:** AGENTS.md rules

### 6. Performance Characteristics ✅

**Token Efficiency:**
- **Large corpus** (52 files, mixed): **71.5x reduction** vs raw files
- **Medium corpus** (4 files, mixed): **5.4x reduction**
- **Small corpus** (6 files): ~1x (fits in context window anyway)

**Incremental Updates:**
- Code-only changes: Instant (AST rebuild, no LLM)
- Doc/image changes: `--update` flag (LLM semantic re-extraction)
- SHA256 cache prevents re-processing unchanged files

### 7. Privacy & Security ✅

**Data Handling:**
- **Code:** Tree-sitter AST parsing → LOCAL (file bytes never sent)
- **Audio/Video:** Whisper transcription → LOCAL (audio never sent)
- **Docs/Papers/Images:** Sent to your AI assistant's API (Claude, GPT-4, etc.)
- **No telemetry:** Zero usage tracking

**Network Calls (Optional Only):**
- `graphify ingest <URL>` - fetch external content (validated URLs only)
- `yt-dlp` - download video audio (user-initiated)
- `graphify --neo4j-push` - upload to Neo4j (user-initiated)

**Security Controls:**
- URL validation (http/https only, blocks private IPs)
- Path validation (XSS prevention, path traversal prevention)
- Size caps (50MB binary, 10MB text)
- No code execution (AST parsing only, no eval)
- Symlink safety (explicit `followlinks=False`)

### 8. Open Source Status ✅

- **Repository:** https://github.com/safishamsi/graphify
- **License:** Apache 2.0 (confirmed)
- **Active Development:** Last commit April 25, 2026
- **Contribution-Friendly:** Architecture.md documents module responsibilities
- **Worked Examples:** Reference outputs in `worked/` directory for verification

---

## Installation Quick Start

```bash
# 1. Install
uv tool install graphifyy

# 2. Setup platform
graphify install --platform claude  # or your platform

# 3. Build first graph
cd /path/to/project
/graphify .

# 4. Make assistant always use it
graphify claude install

# 5. Share with team (optional)
git add graphify-out/{graph.json,GRAPH_REPORT.md}
git commit -m "Add knowledge graph"
```

---

## Files Created

This research produced three comprehensive documents:

1. **RESEARCH_REPORT.md** (19.2 KB)
   - Detailed findings on all aspects
   - Confidence ratings [高/中/低]
   - Full installation instructions
   - Architecture deep-dive

2. **QUICK_REFERENCE.md** (5.8 KB)
   - TL;DR installation
   - Common commands
   - Troubleshooting tips
   - Team workflow

3. **LINUX_INSTALLATION.md** (8.5 KB)
   - Linux-specific setup (4 methods)
   - Server configuration
   - Systemd integration
   - Common issues & solutions

All documents hosted at: **https://github.com/leweii/graphify-research**

---

## Confidence Ratings

| Finding | Level | Evidence |
|---------|-------|----------|
| Graphify is open-source knowledge graph builder | [高] | Official GitHub repo, Apache 2.0 license |
| Architecture: 3-pass pipeline with Leiden clustering | [高] | ARCHITECTURE.md + README detailed explanation |
| Self-hostable on Python 3.10+ | [高] | pyproject.toml, PyPI package, installation docs |
| Supports 25+ code languages via tree-sitter | [高] | README language table + pyproject.toml dependencies |
| Knowledge base integration (Obsidian, Neo4j, etc.) | [高] | README usage section with explicit commands |
| Privacy model (local code/audio, API for docs) | [高] | README Privacy + SECURITY.md |
| Performance metrics (71.5x token reduction) | [中] | README claims with worked/ examples provided |
| Enterprise layer (Penpax) in development | [中] | README mentions with waitlist link |

---

## What I Did

1. **Searched GitHub** - Located official repository (safishamsi/graphify v5 branch)
2. **Retrieved and analyzed:**
   - README.md (31.9 KB) - Full feature documentation
   - ARCHITECTURE.md (3.8 KB) - System design details
   - pyproject.toml (2.4 KB) - Python package config & dependencies
   - SECURITY.md (3.1 KB) - Privacy & threat model
3. **Cross-referenced** - Verified claims against multiple source documents
4. **Verified open-source status** - Confirmed Apache 2.0 license
5. **Created three comprehensive guides** - For different audiences (deep research, quick ref, Linux-specific)

---

## Any Issues Encountered

None. The official repository is well-documented with:
- Clear architecture documentation
- Explicit dependency declarations
- Security policy
- Worked examples
- Multiple language support documentation

All research questions answered from first-hand sources (official GitHub repo).

---

## Next Steps for User

1. **If quick start needed:** Follow QUICK_REFERENCE.md
2. **If Linux server install:** Follow LINUX_INSTALLATION.md
3. **If deep understanding needed:** Read RESEARCH_REPORT.md
4. **If integrating with AI assistant:** See installation commands in any guide
5. **If deploying to team:** Commit graph outputs to git per instructions

All documents cross-verified and confidence-rated.
