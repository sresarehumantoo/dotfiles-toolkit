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
      "method": "apt|go|pipx|cargo|git_clone|appimage|deb|release_binary",
      "package": "package-name",
      "binary": "binary-to-check",
      "app_repo": "owner/repo",
      "git_repo": "https://github.com/owner/repo.git",
      "deb_repo": "owner/repo",
      "release_repo": "owner/repo",
      "asset_pattern": "linux",
      "distros": ["debian", "arch", "fedora"]
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
| `method` | Yes | Install method (see table below) |
| `package` | apt/go/pipx/cargo | Package name, Go import path, or pipx spec |
| `binary` | Yes | For PATH-based methods, the binary name to check; for `appimage` and `git_clone` it determines the install path (`~/.local/bin/<binary>.AppImage` and `~/.local/share/toolkit/<binary>` respectively) |
| `app_repo` | appimage | GitHub `owner/repo` for release downloads |
| `git_repo` | git_clone | Full git URL for cloning |
| `deb_repo` | deb | GitHub `owner/repo` whose latest release publishes a `.deb` |
| `release_repo` | release_binary | GitHub `owner/repo` whose latest release publishes a binary or tarball |
| `asset_pattern` | release_binary (optional) | Substring filter for picking the right release asset (e.g. `"linux-musl"`, `"gnu"`); arch tokens like `x86_64`/`arm64` are matched automatically |
| `distros` | optional | Restrict to these distros: `debian`, `arch`, `fedora`. Omit to allow all. |

### Install Methods

| Method | How it installs | Skip condition |
|--------|----------------|----------------|
| `apt` | `sudo apt-get install <package>` (bulk) | Package already installed |
| `go` | `go install <package>` | Binary in `$PATH` |
| `cargo` | `cargo install <package>` | Binary in `$PATH` |
| `pipx` | `pipx install <package>` | Package in `pipx list` |
| `git_clone` | `git clone --depth=1` to `~/.local/share/toolkit/<binary>` | Directory exists |
| `appimage` | Download from GitHub releases to `~/.local/bin/<binary>.AppImage` | File exists |
| `deb` | Download `.deb` asset from latest GitHub release, install via `dpkg -i` | Binary in `$PATH` |
| `release_binary` | Download asset from latest GitHub release, extract binary if tarball, place at `~/.local/bin/<binary>` | File exists |

## Adding a Tool

1. Add an entry to the `tools` array in `registry.json`
2. Ensure the `name` is unique and matches the regex pattern
3. Set the correct `method` and required fields for that method
4. Test with: `dfinstall install --toolkit --registry ./registry.json --dry-run`

## Categories

| Category | Description |
|----------|-------------|
| Active Directory | AD enumeration and exploitation |
| Applications | Desktop applications (AppImage) |
| Browsers | Web browsers |
| Development | Build tools, language servers, dev utilities |
| DevOps | Configuration management, automation, infra tooling |
| DFIR | Digital forensics & incident response |
| Forensics & Stego | File analysis, steganography |
| Network Tools | Proxying, tunneling, packet capture |
| Office | Productivity & document tooling |
| Password Cracking | Hash crackers, brute-force tools |
| Post-Exploitation | C2 frameworks, post-exploitation toolkits |
| Recon & Scanning | Port scanners, network discovery |
| Reverse Engineering | Debuggers, disassemblers, GDB plugins |
| System | Shells, sudo replacements, low-level system tools |
| Terminal Emulators | Terminal applications |
| Web Testing | Web fuzzers, vulnerability scanners |
| Wordlists | Fuzzing and discovery wordlists |
