# Quick Reference Guide: Graphify Installation & Usage

## TL;DR Installation

```bash
# Recommended: Universal installer for Mac/Linux
uv tool install graphifyy
graphify install --platform claude  # For Claude Code users

# Or with pipx
pipx install graphifyy
graphify install --platform claude

# Or with pip
pip install graphifyy
graphify install --platform claude
```

## System Requirements
- **Python:** 3.10-3.13
- **OS:** Linux, macOS, Windows (WSL)
- **Disk:** ~200MB for base + output space
- **No special permissions needed**

## First-Time Setup

```bash
# 1. Install the package
uv tool install graphifyy

# 2. Setup platform integration (choose one)
graphify install --platform claude    # Claude Code
graphify install --platform cursor    # Cursor IDE
graphify install --platform copilot   # GitHub Copilot CLI
graphify install --platform aider     # Aider

# 3. Build your first graph
cd /path/to/your/project
/graphify .

# 4. Make your assistant always consult the graph
graphify claude install  # or your platform equivalent
```

## What Gets Created

After running `/graphify .`:

```
graphify-out/
├── graph.html              # Interactive graph (open in browser)
├── GRAPH_REPORT.md         # Summary (god nodes, communities, questions)
├── graph.json              # Persistent queryable graph
├── cache/                  # SHA256 cache (skip on re-run if not changed)
└── transcripts/            # Cached video/audio transcripts
```

## Common Commands

### Build & Update Graph
```bash
/graphify .                     # Build on current directory
/graphify ./src --update        # Update only changed files
/graphify ./docs --mode deep    # More aggressive inference
/graphify . --watch             # Auto-sync on file changes
```

### Query the Graph
```bash
/graphify query "what connects auth to user_management?"
/graphify path "LoginHandler" "AuthService"
/graphify explain "OAuth2"
```

### Export Formats
```bash
/graphify . --obsidian                    # Generate Obsidian vault
/graphify . --neo4j                       # Export Neo4j Cypher
/graphify . --neo4j-push bolt://host      # Push to Neo4j instance
/graphify . --wiki                        # Generate Wikipedia-style docs
/graphify . --graphml                     # Export for Gephi/yEd
```

### Maintenance
```bash
graphify hook install          # Auto-rebuild on git changes
graphify watch ./src           # Background auto-sync
graphify check-update ./src    # Check if update needed (cron-safe)
```

## Optional Features (Install as Needed)

```bash
# Video/audio transcription
pip install 'graphifyy[video]'

# PDF extraction
pip install 'graphifyy[pdf]'

# Office docs (DOCX, XLSX)
pip install 'graphifyy[office]'

# Neo4j export
pip install 'graphifyy[neo4j]'

# MCP server for graph queries
pip install 'graphifyy[mcp]'

# All features at once
pip install 'graphifyy[all]'
```

## Excluding Files

Create `.graphifyignore` (like `.gitignore`):

```
# .graphifyignore
vendor/
node_modules/
dist/
AGENTS.md          # Don't extract your own instructions
*.generated.py
docs/translations/
```

## Sharing Graphs with Team

1. **Create graph:** `graphify .`
2. **Commit to git:** `graphify-out/graph.json`, `GRAPH_REPORT.md`, `graph.html`
3. **Gitignore these:**
   - `graphify-out/cache/`
   - `graphify-out/manifest.json`
   - `graphify-out/cost.json`
4. **Others pull:** Assistant reads `GRAPH_REPORT.md` automatically
5. **Setup auto-rebuild:** `graphify hook install`

## Always-On Assistant Integration

After building your graph, make your AI assistant always consult it:

```bash
# For Claude Code
graphify claude install

# For GitHub Copilot CLI
graphify copilot install

# For Cursor
graphify cursor install

# For Aider
graphify aider install

# For VS Code Copilot Chat
graphify vscode install
```

The assistant will now read `GRAPH_REPORT.md` before answering architecture questions.

## What Gets Sent to LLM (Privacy)

- **Stays Local:** Code files (tree-sitter AST only), video/audio (Whisper transcription)
- **Sent to Your API:** Documentation, papers, images (via your AI assistant's API)
- **No Telemetry:** No usage tracking or analytics

## Performance Tips

- **First run is slowest** (extracts and builds graph)
- **Subsequent queries are 71.5x faster** (read compact graph, not raw files)
- **Code-only updates:** Use `--update` (no LLM calls, instant)
- **Docs/images changed:** Run `--update` with LLM semantic re-extraction

## Debugging

```bash
# Check if graph needs updating
graphify check-update ./src

# Watch for changes in background (console output)
graphify watch ./src

# Manually rebuild (clears cache if needed)
/graphify . --no-viz           # Skip HTML visualization
```

## Key Confidence Levels in Graph

- **EXTRACTED:** Found directly in source (explicit import, direct call)
- **INFERRED:** Reasonable deduction (function could call another, co-occurrence)
- **AMBIGUOUS:** Uncertain, flagged for review

You always know what was found vs. guessed.

## Architecture at a Glance

```
Your Files
    ↓
[Pass 1] AST Extraction (tree-sitter, local, no LLM)
    ↓
[Pass 2] Video/Audio Transcription (faster-whisper, local, cached)
    ↓
[Pass 3] LLM Semantic Extraction (Claude, parallel agents, your API)
    ↓
Build NetworkX Graph
    ↓
Community Detection (Leiden algorithm, topology-based)
    ↓
Output: graph.html, graph.json, GRAPH_REPORT.md, (Obsidian/Neo4j/Wiki if requested)
```

## Resources

- **GitHub:** https://github.com/safishamsi/graphify
- **Official Package:** `graphifyy` on PyPI
- **Documentation:** README in repo
- **Architecture Details:** ARCHITECTURE.md in repo
- **Security Model:** SECURITY.md in repo

---

**Note:** The PyPI package is named `graphifyy` (double-y). The CLI and skill command are still `graphify` (single-y).
