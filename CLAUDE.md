# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a macOS security wrapper for Claude Code that uses Apple's deprecated but functional `sandbox-exec` to restrict file system access. The main deliverable is a single bash script (`claude-sandbox`) that generates sandbox profiles and runs Claude Code with restricted permissions.

## Architecture

- **Single executable**: `claude-sandbox` - a self-contained bash script
- **Profile generation**: Creates temporary macOS sandbox profiles in `/tmp/` with project-specific names
- **Template-based**: Uses embedded heredoc template with variable substitution for project directory and home directory paths
- **Security model**: Default deny with explicit allow rules for system files, project directory, and necessary Claude config files

## Key Components

### Script Modes
- `run` (default): Generate profile and run Claude Code
- `generate`: Create sandbox profile only
- `profile`: Show path to generated profile
- `help`: Display usage information

### Sandbox Profile Structure
The embedded profile template includes:
- System file access (read-only): `/System`, `/usr`, `/bin`, `/sbin`, homebrew, nix
- Project workspace (full access): Current working directory and subdirectories
- Claude config access: `~/.claude` directory and `~/.claude.json`
- Temp directory access: `/tmp` and macOS system temp directories
- Explicit denials: Documents, Desktop, Downloads, SSH keys, AWS credentials, etc.
- Network access: Unrestricted (required for Claude API)

## Installation and Usage

The script is designed to be installed to a directory in PATH (e.g., `~/.local/bin/`) and run from project directories:

```bash
# Install
curl -o ~/.local/bin/claude-code-sandbox https://raw.githubusercontent.com/paulsmith/claude-code-sandbox
chmod +x ~/.local/bin/claude-code-sandbox

# Use
cd /path/to/project
claude-code-sandbox
```

## Development Notes

- Script uses `set -euo pipefail` for strict error handling
- Profile names are based on current directory basename for uniqueness
- Variable replacement uses `sed` to substitute `__PROJECT_DIR__` and `__HOME__` placeholders
- Requires macOS with `sandbox-exec` (deprecated in macOS 14 but still functional)