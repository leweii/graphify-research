# Graphify Installation on Linux Server

## Prerequisites

```bash
# Ubuntu/Debian - install Python 3.10+
sudo apt-get update
sudo apt-get install -y python3.10 python3.10-venv python3.10-dev

# OR on Fedora/RHEL
sudo dnf install -y python3.10 python3.10-devel

# Verify installation
python3.10 --version
```

## Installation Methods for Linux Server

### Method 1: Using `uv` (Recommended)

Fastest, cleanest installation:

```bash
# Install uv (one-time setup)
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.cargo/env

# Install graphify
uv tool install graphifyy

# Verify
graphify --version
```

**Pros:**
- No PATH configuration needed
- Isolated environment
- Fast installation
- Best for shared servers

### Method 2: Using `pipx`

Isolated virtual environments per tool:

```bash
# Install pipx
sudo apt-get install -y pipx

# Install graphify
pipx install graphifyy

# Verify
graphify --version
```

If `graphify: command not found`, add to PATH:
```bash
export PATH="$HOME/.local/bin:$PATH"
# Add to ~/.bashrc or ~/.zshrc for persistence
```

### Method 3: Using `pip` with Virtual Environment (Safest)

Best practice for shared environments:

```bash
# Create project directory
mkdir -p ~/graphify-project
cd ~/graphify-project

# Create virtual environment
python3.10 -m venv venv

# Activate it
source venv/bin/activate

# Install graphify
pip install --upgrade pip
pip install graphifyy

# Verify
graphify --version

# Add to PATH (optional)
export PATH="$(pwd)/venv/bin:$PATH"
```

### Method 4: System-Wide Installation (Linux)

Use when you have sudo and want all users to access:

```bash
# Using pip with --break-system-packages (Python 3.12+)
sudo pip install graphifyy

# OR better: use deadsnakes PPA (Ubuntu)
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install -y python3.10-full
sudo python3.10 -m pip install graphifyy
```

**Note:** Not recommended on shared servers or with managed Python

## Platform Setup (Linux Server)

After installation, integrate with your AI coding assistant:

### For Claude Code on Linux
```bash
# Setup instructions and hooks
graphify install --platform claude

# This creates:
# - ~/.claude/skills/graphify/SKILL.md
# - ~/.claude/CLAUDE.md (instructions)
# - ~/.claude/settings.json (PreToolUse hook)
```

### For Codex on Linux
```bash
graphify install --platform codex

# This creates:
# - ~/.codex/skills/graphify/SKILL.md
# - ~/.codex/AGENTS.md
# - ~/.codex/hooks.json (PreToolUse hook)
```

### For VS Code Copilot Chat (WSL/Linux)
```bash
graphify vscode install

# Creates .github/copilot-instructions.md in project
```

### For Terminal-Only Tools (Aider, OpenClaw, etc.)
```bash
graphify install --platform aider

# Creates AGENTS.md in project
# No assistant platform integration needed
```

## Optional Dependencies for Linux

### Video/Audio Support (Whisper Transcription)
```bash
# Install system dependencies first
sudo apt-get install -y ffmpeg libsndfile1

# Then install Python packages
pip install 'graphifyy[video]'

# For large models, ensure disk space:
# - tiny: ~32 MB
# - base: ~140 MB
# - small: ~461 MB
# - medium: ~1.5 GB
# - large: ~2.9 GB (default)
```

### PDF Support
```bash
pip install 'graphifyy[pdf]'
```

### Neo4j Export
```bash
pip install 'graphifyy[neo4j]'
```

### Office Documents (DOCX, XLSX)
```bash
pip install 'graphifyy[office]'
```

### File System Watching (Auto-Sync)
```bash
pip install 'graphifyy[watch]'
```

### MCP Server (Graph Queries via Protocol)
```bash
pip install 'graphifyy[mcp]'
```

### All Optional Features
```bash
pip install 'graphifyy[all]'
```

## Verify Installation

```bash
# Check CLI
graphify --help
graphify --version

# Check Python module
python3 -c "import graphify; print(graphify.__version__)"

# Quick test (requires an AI assistant to be configured)
# In Claude Code or other assistant:
# /graphify .
```

## Server-Specific Configuration

### Running on Headless Linux Server

For servers without display/interactive shell:

```bash
# Use screen or tmux for persistent sessions
screen -S graphify

# Inside screen:
cd /path/to/project
graphify .

# Detach: Ctrl+A, then D
# Reattach: screen -r graphify
```

Or background with nohup:
```bash
nohup graphify . > graphify.log 2>&1 &
```

### Git Hooks on Shared Server

Auto-rebuild graph on commits:

```bash
cd /path/to/project

# Install hooks
graphify hook install

# This creates .git/hooks/post-commit and post-checkout
# Verify:
ls -la .git/hooks/post-*

# If hooks don't execute, check permissions:
chmod +x .git/hooks/post-*
```

### Persistent Watch Mode on Server

Keep graph synchronized in background:

```bash
# Start in tmux/screen
tmux new-session -d -s graphify-watch

# Inside session:
cd /path/to/project
graphify watch ./src
```

Or with systemd (for long-running):

Create `/etc/systemd/user/graphify-watch.service`:
```ini
[Unit]
Description=Graphify Auto-Sync
After=network.target

[Service]
Type=simple
ExecStart=%h/.local/bin/graphify watch %h/project/src
Restart=always
RestartSec=10

[Install]
WantedBy=default.target
```

Then:
```bash
systemctl --user enable graphify-watch
systemctl --user start graphify-watch
systemctl --user status graphify-watch
```

## Disk Space Considerations

```
graphify installation:      ~100-200 MB (with all dependencies)
graph.json (typical):        1-50 MB (depends on corpus size)
graph.html:                  500 KB - 5 MB
Transcripts cache:           10-100 MB (per hour of video)
Whisper models:              30 MB - 2.9 GB (depends on model size)
Total per project:           100 MB - 3 GB
```

Check disk usage:
```bash
du -sh graphify-out/
```

Clean up transcripts cache (safely):
```bash
rm -rf graphify-out/transcripts/
# Will be regenerated on next run
```

## Common Issues on Linux

### Issue: `graphify: command not found`

**Solution 1:** Add to PATH
```bash
# If using pip/pipx
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Solution 2:** Use full path
```bash
~/.local/bin/graphify --version
```

**Solution 3:** Use Python module
```bash
python3 -m graphify --version
```

### Issue: Python 3.10 not found

```bash
# Install it
sudo apt-get install python3.10

# Or use available version (3.11+)
python3.11 -m venv venv
source venv/bin/activate
pip install graphifyy
```

### Issue: Permission denied on git hooks

```bash
# Make hooks executable
chmod +x .git/hooks/post-commit
chmod +x .git/hooks/post-checkout
```

### Issue: Out of disk space

```bash
# Clear cache (safe)
rm -rf graphify-out/cache/

# Clear transcripts (safe, will regenerate)
rm -rf graphify-out/transcripts/

# Check what's using space
du -sh graphify-out/*
```

### Issue: Whisper model too slow on first use

```bash
# Pre-download model
graphify --whisper-model small  # Choose: tiny, base, small, medium, large

# Models cached to ~/.cache/huggingface/
```

### Issue: Memory error during processing

```bash
# Use smaller Whisper model
pip install 'graphifyy[video]'
/graphify . --whisper-model base  # Not 'large'

# Or split processing
/graphify ./src                    # Code only
/graphify ./docs --update          # Docs separately
```

## Running in Docker (Optional)

For consistent environments:

```dockerfile
FROM python:3.10-slim

RUN apt-get update && apt-get install -y \
    git \
    ffmpeg \
    && rm -rf /var/lib/apt/lists/*

RUN pip install graphifyy[all]

WORKDIR /workspace

CMD ["/bin/bash"]
```

Build and run:
```bash
docker build -t graphify .
docker run -it -v $(pwd):/workspace graphify
```

Inside container:
```bash
graphify .
```

## Next Steps

1. **Build initial graph:**
   ```bash
   graphify /path/to/project
   ```

2. **Setup assistant integration:**
   ```bash
   graphify install --platform <platform>
   # or
   graphify setup-mcp  # for MCP server
   ```

3. **Commit to git (recommended):**
   ```bash
   git add graphify-out/graph.json graphify-out/GRAPH_REPORT.md
   git commit -m "Add graphify knowledge graph"
   ```

4. **Install hooks (optional):**
   ```bash
   graphify hook install
   ```

5. **Test in your assistant:**
   - Type `/graphify query "what are god nodes in this codebase?"`
   - Or use always-on integration: assistant reads graph automatically

---

## Resources

- **Official Repo:** https://github.com/safishamsi/graphify
- **Installation Docs:** README.md in repo
- **Architecture:** ARCHITECTURE.md
- **Security:** SECURITY.md
- **PyPI Package:** `graphifyy`
