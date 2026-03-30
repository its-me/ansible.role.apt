# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Linting

CI runs ansible-lint via the GitLab component pipeline (`.gitlab-ci.yml`). To lint locally:

```bash
ansible-lint
```

## Role Overview

This is an Ansible role (`apt`) that manages APT configuration and package installation on Debian-based systems. It handles:
- `/etc/apt/sources.list` generation
- External repository and GPG key management (`/etc/apt/sources.list.d/`)
- Package pinning (`/etc/apt/preferences.d/`)
- APT recommends/suggests settings

## Variable Architecture

Repository definitions live in two dicts that get merged:
- `apt_all` — inventory-wide repository definitions
- `apt_env` — environment/host overrides
- `apt` — the merged result (`apt_all | combine(apt_env, recursive=True)`)

Each entry in `apt_all`/`apt_env` is keyed by a repo name and contains:
- `repo`: the full APT source line(s)
- `key`: GPG key reference, either `<url>` (URL-only) or `<keyserver>=<id>` (split on `=` to get keyserver + key ID)

`apt_repo` selects which entry to activate. `apt_path` and `apt_key` are derived defaults that resolve the selected repo's values.

## Task Flow (tasks/main.yml)

1. Generate `/etc/apt/sources.list` from template — registers `_sources_list`
2. Allow expired keys for archived Debian releases (`< apt_latest_supported_debian`)
3. Configure recommends/suggests if those vars are defined
4. Validate `apt_repo` exists in `apt` dict (fails fast otherwise)
5. Add GPG key (keyserver or URL) when `apt_key` is set
6. Add repository file to `sources.list.d/` — registers `_new_repository`
7. Update APT cache only when `_sources_list` or `_new_repository` changed
8. Pin packages via `preferences.d/` template
9. Install packages from `apt_packages`

## Package Pinning Logic

`templates/etc/apt/preferences.d/package.j2` handles two cases:
- `pkg=<version>` → Pin by version, priority 1001
- `pkg` (no `=`) → Pin by origin hostname extracted from `apt_path`, priority 999

The pin file name is derived from the package name (`.` → `_`, `*` → `_wildcard`).

## Usage as a Dependency

```yaml
dependencies:
  - role: apt
    apt_packages: [curl, wget]           # standard Debian
  - role: apt
    apt_repo: backports
    apt_packages: [some-package]          # from backports
  - role: apt
    apt_repo: repo_name                   # from external repo defined in apt_all
    apt_packages: [some-package]
```

`apt_latest_supported_debian` (default: `10`) controls which Debian versions get `-updates` sources and expired-key allowances. Update this when Debian releases move to archive.
