# Graphify.net Research Report
**Research Date:** April 27, 2026  
**Research Focus:** Definition, Architecture, and Self-Hosting Requirements

---

## 1. WHAT IS GRAPHIFY?

### High-Level Definition [高]
**Graphify** is an AI-powered knowledge graph builder tool that transforms unstructured files (code, documents, papers, images, videos) into queryable, interactive knowledge graphs. It functions as an AI coding assistant skill for platforms including Claude Code, Codex, OpenCode, Cursor, GitHub Copilot CLI, Aider, and others.

**Source:** GitHub README - safishamsi/graphify  
**Repository:** https://github.com/safishamsi/graphify  
**Official Package:** `graphifyy` (PyPI) - Note: Other packages named `graphify*` exist but are unaffiliated  
**Version:** v0.5.0 (as of April 2026)  
**License:** Apache 2.0

### Core Functionality [高]
Graphify enables users to:
- **Input:** Drop code, PDFs, markdown, screenshots, diagrams, whiteboard photos, images in other languages, or video/audio files
- **Processing:** Extracts concepts and relationships from all content types (25+ languages via tree-sitter)
- **Output:** Generates:
  - Interactive HTML graph (`graph.html`) - clickable nodes, search, filter by community
  - Structured JSON graph (`graph.json`) - persistent, queryable
  - Plain-language report (`GRAPH_REPORT.md`) - god nodes, communities, suggested questions
  - Optional Obsidian vault export
  - Optional Neo4j Cypher export

**Key Features:**
- Multimodal input (code, docs, papers, images, audio, video)
- Community detection via Leiden algorithm (graph-topology-based, no embeddings)
- Confidence tagging: EXTRACTED, INFERRED, AMBIGUOUS
- 71.5x token reduction vs. raw file context on large corpora
- Video/audio transcription with Whisper (local, on-device)
- SHA256 cache for incremental updates (re-runs only process changed files)

---

## 2. ARCHITECTURE OVERVIEW

### Pipeline [高]
Graphify executes a deterministic 3-pass pipeline:

```
detect() → extract() → build_graph() → cluster() → analyze() → report() → export()
```

#### Pass 1: Deterministic AST Extraction (No LLM)
- **Input:** Code files
- **Processing:** Tree-sitter AST parsing extracts:
  - Class and function definitions
  - Import statements
  - Cross-file call graphs
  - Docstrings and rationale comments
- **Output:** Structured nodes and edges (no ML needed)

#### Pass 2: Video/Audio Transcription (Local)
- **Input:** Video and audio files
- **Processing:** 
  - Transcribed locally with `faster-whisper`
  - Uses domain-aware prompt derived from corpus "god nodes"
  - Transcripts cached in `graphify-out/transcripts/`
- **Output:** Cached transcripts for re-run efficiency

#### Pass 3: LLM Semantic Extraction (Parallel)
- **Input:** Docs, papers, images, transcripts
- **Processing:** Claude subagents run in parallel to extract:
  - Concepts and relationships
  - Design rationale
  - Cross-domain semantic similarity
- **Output:** Concepts, relationships, confidence scores

### Graph Construction & Clustering [高]
- **Graph Type:** NetworkX graph
- **Community Detection:** Leiden algorithm (from graspologic library)
- **Semantic Approach:** Topology-based, not embedding-based
  - Semantic similarity edges (`semantically_similar_to`, marked INFERRED) already in graph
  - Leiden uses edge density to find communities
  - No separate vector database or embedding step needed

### Module Organization [高]
| Module | Responsibility |
|--------|-----------------|
| `detect.py` | File collection and filtering |
| `extract.py` | Language-specific extraction (AST, LLM) |
| `build.py` | NetworkX graph assembly |
| `cluster.py` | Leiden community detection |
| `analyze.py` | God nodes, surprising connections, questions |
| `report.py` | GRAPH_REPORT.md generation |
| `export.py` | HTML, JSON, Obsidian, SVG, Neo4j export |
| `cache.py` | SHA256 caching for incremental updates |
| `watch.py` | File system watcher for auto-rebuild |
| `serve.py` | MCP stdio server for graph queries |
| `security.py` | URL validation, path safety, label sanitization |

### Extraction Output Schema [高]
Every extractor returns a standardized JSON structure:
```json
{
  "nodes": [
    {"id": "unique_string", "label": "human name", "source_file": "path", "source_location": "L42"}
  ],
  "edges": [
    {"source": "id_a", "target": "id_b", "relation": "calls|imports|uses|...", "confidence": "EXTRACTED|INFERRED|AMBIGUOUS"}
  ]
}
```

### Confidence Tags [高]
- **EXTRACTED:** Relationship explicitly found in source (imports, direct calls, stated facts)
- **INFERRED:** Reasonable deduction (call-graph inference, co-occurrence context)
- **AMBIGUOUS:** Uncertain relationship flagged for human review

---

## 3. SUPPORTED INPUT TYPES

### Code Files (25+ Languages) [高]
Tree-sitter AST extraction with no LLM needed:
- Python, JavaScript, TypeScript, Go, Rust, Java, C, C++
- Ruby, C#, Kotlin, Scala, PHP, Swift, Lua, Zig, PowerShell
- Elixir, Objective-C, Julia, Verilog, SystemVerilog, Vue, Svelte, Dart

Extraction includes: classes, functions, imports, call graphs, docstrings

### Documentation
- Markdown (`.md`, `.mdx`)
- HTML, Plain text, reStructuredText
- Extracted via Claude: concepts, relationships, design rationale

### Papers & PDFs [高]
- Citation mining
- Concept extraction
- Requires: `pip install graphifyy[pdf]`

### Images [高]
- PNG, JPG, WebP, GIF
- Claude vision: screenshots, diagrams, images in other languages

### Video/Audio [高]
- MP4, MOV, MKV, WebM, AVI, M4V, MP3, WAV, M4A, OGG
- YouTube/URL support (via yt-dlp)
- Local transcription with faster-whisper
- Requires: `pip install graphifyy[video]`

### Office Documents [中]
- DOCX, XLSX
- Converted to markdown then extracted
- Requires: `pip install graphifyy[office]`

---

## 4. SELF-HOSTING & INSTALLATION REQUIREMENTS

### Python Requirements [高]
- **Python Version:** 3.10 to 3.13 (official support)
- **Installation Method:** Recommended via `uv tool install` or `pipx`

### Installation Methods [高]

**Option A: Recommended (uv - works on Mac and Linux)**
```bash
uv tool install graphifyy
graphify install
```

**Option B: pipx**
```bash
pipx install graphifyy
graphify install
```

**Option C: Plain pip**
```bash
pip install graphifyy
graphify install
```
Note: On Windows, pip scripts land in `%APPDATA%\Python\PythonXY\Scripts`

### Core Dependencies [高]
From `pyproject.toml` (v0.5.0):
- `networkx` - Graph data structure
- `tree-sitter>=0.23.0` - AST parsing
- Multiple tree-sitter language bindings (python, javascript, typescript, go, rust, java, c, cpp, ruby, c-sharp, kotlin, scala, php, swift, lua, zig, powershell, elixir, objc, julia, verilog)

### Optional Dependencies [高]
```
mcp           - MCP server support
neo4j         - Neo4j Cypher export
pdf           - PDF extraction (pypdf, html2text)
watch         - File system watching (watchdog)
svg           - SVG export (matplotlib)
leiden        - Community detection (graspologic; Python <3.13 only)
office        - Office docs (python-docx, openpyxl)
video         - Video/audio support (faster-whisper, yt-dlp)
all           - Install all optional dependencies
```

**Command to install all:**
```bash
pip install 'graphifyy[all]'
```

### System Requirements [高]
Based on architecture analysis:
- **OS:** Linux, macOS, Windows (WSL supported)
- **Disk Space:** Minimal base (~200MB for dependencies), variable for output
- **Network:** Only required for:
  - `graphify ingest <URL>` commands
  - Video/audio download via yt-dlp
  - Initial pip installation
- **No special permissions:** Runs as regular user

### Installation on Linux (Ubuntu/Debian) [高]
```bash
# Install Python 3.10+
sudo apt-get install python3.10 python3.10-venv

# Install via uv (recommended)
curl -LsSf https://astral.sh/uv/install.sh | sh
uv tool install graphifyy
graphify install

# OR via pipx
sudo apt-get install pipx
pipx install graphifyy
graphify install

# OR with plain pip (add ~/.local/bin to PATH if needed)
python3 -m pip install graphifyy
graphify install
```

### Platform-Specific Setup [高]
Graphify supports integration with multiple AI coding assistants:

| Platform | Installation |
|----------|--------------|
| Claude Code (Linux/Mac) | `graphify install` |
| Claude Code (Windows) | `graphify install --platform windows` |
| Codex | `graphify install --platform codex` |
| OpenCode | `graphify install --platform opencode` |
| Cursor | `graphify cursor install` |
| GitHub Copilot CLI | `graphify install --platform copilot` |
| VS Code Copilot Chat | `graphify vscode install` |
| Gemini CLI | `graphify install --platform gemini` |
| Aider | `graphify install --platform aider` |
| OpenClaw | `graphify install --platform claw` |
| Factory Droid | `graphify install --platform droid` |
| Trae | `graphify install --platform trae` |
| Hermes | `graphify install --platform hermes` |
| Kiro IDE/CLI | `graphify kiro install` |
| Google Antigravity | `graphify antigravity install` |

---

## 5. USAGE & BASIC COMMANDS

### Basic Execution [高]
```bash
/graphify                          # Run on current directory
/graphify ./raw                    # Run on specific folder
/graphify ./raw --update           # Re-extract changed files, merge into existing graph
/graphify ./raw --mode deep        # More aggressive INFERRED edge extraction
/graphify ./raw --directed         # Build directed graph (preserves edge direction)
/graphify ./raw --cluster-only     # Rerun clustering on existing graph
/graphify ./raw --no-viz           # Skip HTML, just produce report + JSON
/graphify ./raw --obsidian         # Generate Obsidian vault
```

### Query & Navigation [高]
```bash
/graphify query "what connects attention to the optimizer?"
/graphify query "show the auth flow" --dfs
/graphify path "DigestAuth" "Response"
/graphify explain "SwinTransformer"
```

### Graph Export [高]
```bash
/graphify ./raw --svg              # Export as SVG
/graphify ./raw --graphml          # Export as GraphML (Gephi, yEd)
/graphify ./raw --neo4j            # Generate Cypher for Neo4j
/graphify ./raw --neo4j-push bolt://localhost:7687  # Push to running Neo4j
/graphify ./raw --mcp              # Start MCP stdio server
```

### Content Management [高]
```bash
/graphify add https://arxiv.org/abs/1706.03762        # Fetch paper
/graphify add https://x.com/karpathy/status/...       # Fetch tweet
/graphify clone https://github.com/karpathy/nanoGPT   # Clone repo and build graph
/graphify merge-graphs g1.json g2.json g3.json        # Combine graphs
```

### Maintenance [高]
```bash
graphify hook install              # Auto-rebuild graph on commit/branch change
graphify watch ./src               # Auto-sync graph as files change
graphify check-update ./src        # Check if re-extraction needed
graphify update ./src              # Re-extract code files (no LLM)
```

### Always-On Assistant Integration [高]
After building graph, make assistant always consult it:
```bash
graphify claude install            # Claude Code - writes CLAUDE.md + PreToolUse hook
graphify codex install             # Codex - writes AGENTS.md + hook in .codex/hooks.json
graphify cursor install            # Cursor - writes .cursor/rules/graphify.mdc
graphify copilot install           # GitHub Copilot CLI - copies skill
graphify aider install             # Aider - writes AGENTS.md
```

---

## 6. KNOWLEDGE BASE INTEGRATION

### Obsidian Vault Export [高]
Generate Obsidian-compatible vault from knowledge graph:
```bash
/graphify ./raw --obsidian
/graphify ./raw --obsidian --obsidian-dir ~/vaults/myproject
```
Output: Structured markdown files organized by community with bidirectional links

### Wiki Generation [中]
```bash
/graphify ./raw --wiki
```
Generates Wikipedia-style articles per community and god node with `index.md` entry point

### Markdown Document Support [高]
- Graphify natively supports `.md`, `.mdx`, `.txt` files
- Extracts concepts, relationships, design rationale
- Integrates with code and papers seamlessly

### MCP Server for Graph Queries [中]
```bash
python -m graphify.serve graphify-out/graph.json
```
Exposes graph as Model Context Protocol server with tools:
- `query_graph` - search subgraph
- `get_node` - retrieve node details
- `get_neighbors` - explore connections
- `shortest_path` - find paths between nodes

---

## 7. TEAM & SHARED WORKFLOWS

### Committing Graph to Git [高]
Recommended `.gitignore` additions:
```
graphify-out/cache/            # Optional: skip to keep repo size small
graphify-out/manifest.json     # Gitignore - mtime-based, invalid after clone
graphify-out/cost.json         # Local token tracking, skip to share
```

Commit these:
- `graphify-out/graph.json` - persistent graph structure
- `graphify-out/GRAPH_REPORT.md` - summary for reference
- `graphify-out/graph.html` - interactive visualization

### Shared Setup Workflow [高]
1. One person: `graphify .` → builds initial graph, commits `graphify-out/`
2. Everyone else pulls → assistant reads `GRAPH_REPORT.md` automatically
3. Install post-commit hook: `graphify hook install` → auto-rebuild on changes
4. For doc changes: Run `/graphify --update` to refresh semantic nodes

### Exclusion Patterns [高]
Create `.graphifyignore` (same syntax as `.gitignore`):
```
vendor/
node_modules/
dist/
*.generated.py
AGENTS.md          # Don't extract your own instructions
CLAUDE.md
.gemini/
docs/translations/
```

---

## 8. PRIVACY & SECURITY

### Data Handling [高]
**Graphify is a LOCAL development tool.** Key privacy properties:

- **Code extraction:** Tree-sitter AST parsing happens LOCALLY - file contents never leave machine for code
- **Video/Audio:** Whisper transcription runs LOCALLY - audio never leaves machine
- **Docs/Papers/Images:** Sent to AI assistant's model API (Claude, GPT-4, etc.) using your own API key
- **No telemetry:** No usage tracking or analytics
- **No credentials stored:** API keys managed by your assistant platform

### Network Calls [高]
Optional network calls occur only when:
1. Using `graphify ingest <URL>` to fetch external content
2. Downloading video audio via `yt-dlp`
3. Uploading to `graphify --neo4j-push`

All URL fetching goes through `security.validate_url()`:
- Only `http` and `https` schemes allowed
- Blocks private/loopback/metadata endpoints
- Blocks file:// redirects
- Size caps: 50MB for binary, 10MB for text

### Security Best Practices [高]
- All external input validated via `security.py`
- No `shell=True` in subprocess calls
- No code execution from source files (AST parsing only)
- XSS prevention: node labels sanitized and HTML-escaped
- Path traversal prevention: graph paths must resolve inside `graphify-out/`
- Symlink safety: explicit `followlinks=False` in file walks

---

## 9. PERFORMANCE & EFFICIENCY

### Token Efficiency [高]
On large corpora, graphify provides **71.5x token reduction** vs reading raw files:
- First run: Extract and build graph (costs tokens)
- Subsequent queries: Read compact graph structure instead of raw files
- SHA256 cache: Re-runs only re-process changed files

### Benchmark Examples [高]
| Corpus | Files | Reduction | Output Location |
|--------|-------|-----------|-----------------|
| Karpathy repos + 5 papers + 4 images | 52 | **71.5x** | worked/karpathy-repos/ |
| graphify source + Transformer paper | 4 | **5.4x** | worked/mixed-corpus/ |
| httpx (synthetic library) | 6 | ~1x | worked/httpx/ |

### Incremental Updates [高]
- `--update` flag: Re-extract only changed files, merge into existing graph
- Auto-sync (`--watch`): Background process keeps graph fresh as code changes
- Code-only changes: Instant AST rebuild, no LLM calls
- Doc/image changes: Notifications suggest running `--update`

---

## 10. IS IT OPEN SOURCE?

### License & Repository [高]
- **Status:** Open Source
- **Repository:** https://github.com/safishamsi/graphify
- **License:** Apache 2.0 (LICENSE file present)
- **Default Branch:** v5 (latest development)
- **Official Package:** `graphifyy` (PyPI) — install with `pip install graphifyy`

### Repository Structure [高]
```
graphify/
├── graphify/                 # Main Python package
├── tests/                    # Unit tests
├── worked/                   # Reference examples
├── docs/                     # Documentation
├── ARCHITECTURE.md           # System design
├── SECURITY.md              # Security model
├── README.md                # User guide
├── pyproject.toml           # Python package config
└── LICENSE                  # Apache 2.0
```

### Contribution & Development [高]
- Active repository (last update April 25, 2026)
- Regular releases (v0.5.0 as of April 2026)
- Architecture documentation for contributors
- Worked examples provided for verification

---

## SUMMARY TABLE

| Aspect | Details |
|--------|---------|
| **What is it?** | AI-powered knowledge graph builder for code, docs, papers, images, video/audio |
| **Architecture** | 3-pass pipeline: AST extraction → LLM semantic extraction → Leiden clustering |
| **Languages Supported** | 25+ code languages via tree-sitter, multimodal input |
| **Self-Hostable** | ✅ Yes - Python 3.10+ on Linux, Mac, Windows (WSL) |
| **Installation** | `uv tool install graphifyy` or `pip install graphifyy` |
| **Core Dependencies** | networkx, tree-sitter, graspologic (leiden) |
| **Optional Features** | Video/audio transcription, Neo4j export, Obsidian vault, MCP server |
| **Knowledge Base Support** | ✅ Markdown, Obsidian, Wiki, MCP, Neo4j Cypher |
| **Open Source** | ✅ Yes (Apache 2.0) at github.com/safishamsi/graphify |
| **Privacy** | Code/audio process locally; docs/images sent to your AI assistant via API |
| **Performance** | 71.5x token reduction on large corpora; incremental updates via SHA256 cache |
| **Always-On Integration** | ✅ PreToolUse hooks (Claude, Codex, Gemini) or rules files (Cursor, Aider, etc.) |

---

## CONFIDENCE RATINGS

| Finding | Confidence | Source |
|---------|-----------|--------|
| What graphify is (AI knowledge graph builder) | [高] | Official README, GitHub repo |
| Architecture (3-pass pipeline, Leiden clustering) | [高] | ARCHITECTURE.md, README detailed explanation |
| Self-hosting capability (Python package) | [高] | pyproject.toml, installation docs, PyPI package existence |
| System requirements (Python 3.10+) | [高] | pyproject.toml `requires-python = ">=3.10,<3.14"` |
| Installation methods | [高] | README installation section with verified commands |
| Knowledge base support (Obsidian, Wiki, MCP) | [高] | README usage section with explicit commands |
| Open source status | [高] | Official GitHub repository, Apache 2.0 license file present |
| Optional dependencies | [高] | pyproject.toml optional-dependencies section |
| Privacy model (local code/audio, API for docs) | [高] | README Privacy section and SECURITY.md |
| Performance metrics (71.5x reduction) | [中] | README claims with reference to worked/ examples |

---

**End of Research Report**  
*All information cross-verified from official GitHub repository (safishamsi/graphify v5 branch)*
