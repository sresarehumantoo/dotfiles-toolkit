# dotfiles-toolkit

External tool registry for [dotfiles](https://github.com/sresarehumantoo/dotfiles).

This repository contains the toolkit registry — a JSON file describing security, CTF, and development tools that can be installed via `dfinstall --toolkit`. Keeping tool names and metadata in a separate repository ensures the main dfinstall binary stays clean and free of strings that might trigger EDR heuristics.

## Usage

```bash
# Interactive menu — fetches latest registry automatically
dfinstall install --toolkit

# Use a local registry file
dfinstall install --toolkit --registry ./registry.json

# Set a permanent override in config
dfinstall config set toolkit_registry_url file:///path/to/registry.json
```

The registry is cached locally at `~/.local/share/dfinstall/toolkit-registry.json` after the first fetch.

## Registry Schema

```json
{
  "version": 1,
  "tools": [
    {
      "name": "toolname",
      "description": "What the tool does",
      "category": "Category Name",
      "method": "apt|go|pipx|cargo|git_clone|appimage",
      "package": "package-name",
      "binary": "binary-to-check",
      "app_repo": "owner/repo",
      "git_repo": "https://github.com/owner/repo.git"
    }
  ]
}
```

### Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier. Must match `^[a-zA-Z0-9][a-zA-Z0-9_-]*$` |
| `description` | Yes | Short description shown in the selection menu |
| `category` | Yes | Group label for organizing the menu |
| `method` | Yes | Install method: `apt`, `go`, `pipx`, `cargo`, `git_clone`, `appimage` |
| `package` | apt/go/pipx/cargo | Package name or Go import path |
| `binary` | Yes | Binary name to check for existence (used for skip-if-installed) |
| `app_repo` | appimage | GitHub `owner/repo` for release downloads |
| `git_repo` | git_clone | Full git URL for cloning |

### Install Methods

| Method | How it installs | Skip condition |
|--------|----------------|----------------|
| `apt` | `sudo apt-get install <package>` (bulk) | Package already installed |
| `go` | `go install <package>` | Binary in `$PATH` |
| `cargo` | `cargo install <package>` | Binary in `$PATH` |
| `pipx` | `pipx install <package>` | Package in `pipx list` |
| `git_clone` | `git clone --depth=1` to `~/.local/share/toolkit/<name>` | Directory exists |
| `appimage` | Download from GitHub releases to `~/.local/bin/<name>.AppImage` | File exists |

## Adding a Tool

1. Add an entry to the `tools` array in `registry.json`
2. Ensure the `name` is unique and matches the regex pattern
3. Set the correct `method` and required fields for that method
4. Test with: `dfinstall install --toolkit --registry ./registry.json --dry-run`

## Categories

| Category | Description |
|----------|-------------|
| Recon & Scanning | Port scanners, network discovery |
| Web Testing | Web fuzzers, vulnerability scanners |
| Password Cracking | Hash crackers, brute-force tools |
| Network Tools | Proxying, tunneling, packet capture |
| Forensics & Stego | File analysis, steganography |
| Reverse Engineering | Debuggers, disassemblers, GDB plugins |
| Active Directory | AD enumeration and exploitation |
| Post-Exploitation | C2 frameworks, post-exploitation toolkits |
| Wordlists | Fuzzing and discovery wordlists |
| Development | Build tools, servers |
| Applications | Desktop applications (AppImage) |
