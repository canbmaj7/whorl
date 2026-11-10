# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

Bootstrap stage (2026-11-10): the repo contains the plan (`docs/PLAN.md`), the sensor catalog seed (`data/sensors.yaml`), field notes from the first working install (`docs/field-notes-27c6-5125.md`) and community files. **There is no application code, build system or test suite yet.** Do not invent commands; add them here when milestone M0/M1 creates them.

## What Whorl is

A GUI + CLI wizard that takes a Linux user from "I have a fingerprint reader" to "my finger works for `sudo`, polkit prompts, the lock screen (and the password manager unlock in the browser)". It detects the sensor, checks whether libfprint supports it, installs a community driver from a curated catalog when it does not, enrolls fingers through fprintd, and toggles PAM integration with backups and one-click rollback. Name = "whorl", a fingerprint pattern.

The maintainer, Can, writes Turkish: reply in Turkish in chat, but keep all repo content (docs, code, issues, commits) in English because outside contributors are expected.

## Decisions already made (see docs/PLAN.md for reasoning)

- Whorl is an installer/manager, **not** a driver project. New drivers belong upstream (libfprint) or in community forks; Whorl only catalogs them.
- Stack: Python + PySide6/QML for the GUI, one shared core library, a thin `whorl` CLI on top. fprintd is driven over its D-Bus API (`net.reactivated.Fprint`), never by shelling out to `fprintd-*` tools in library code.
- Privileged work (PAM files, package install, sensor key writes) goes through a small separate helper run via polkit/`pkexec`; the GUI process itself never runs as root.
- First target: Arch/CachyOS + KDE Plasma + Goodix `27c6:5125`. Other sensors/distros come through the catalog later. Packaging: AUR first (Flatpak is a poor fit because it must edit `/etc/pam.d`).
- Browser story is honest and limited: Chrome/Firefox on Linux do not use fprintd for password fields. Supported route = Bitwarden desktop "unlock with system authentication" (polkit) + browser extension. Never promise per-site fingerprint prompts.

## Safety invariants (never relax without the maintainer's explicit say-so)

- Never write a key or anything else to a sensor implicitly. Sensor writes (e.g. Goodix PSK) are irreversible, need an explicit, specific confirmation naming what is overwritten, and a log entry.
- No firmware flashing, ever. No telemetry. Never read, copy or upload enrolled fingerprint data or key files.
- Every edit of `/etc/pam.d/*` is preceded by a timestamped backup, uses `sufficient` (password must keep working), and has a tested rollback. Verify the result with the user's existing session open: a broken PAM stack can lock out `sudo`; `pkexec` has its own stack and is the recovery path.
- Catalog entries must state `risk` flags honestly (`writes_key`, `replaces_system_package`).

## Working method

- Claude orchestrates; long or multi-file code is delegated to Antigravity (agy, `mcp__antigravity__*`, skill `agy-is-verme`); verify its output yourself (diff, lint, tests). Small fixes, git, SQL-like checks and verification stay with Claude.
- **Use Fable as a subagent** for independent reviews (security review of the PAM manager and the helper, review of the plan). Spawn with the Agent tool and `model: "fable"`; ask for a read-only report with ranked findings and verify its claims. The Fable weekly quota is separate from the main one and the account's current allowance resets 2026-11-11 16:00 (Europe/Istanbul); if it is exhausted, say so and use the normal model.
- The test machine is the maintainer's laptop (HONOR MagicBook 14, CachyOS, 7 GB RAM): use `-j2` for compiles, do not run heavy builds in parallel.

## Next steps (handoff)

1. Finish repo bootstrap if anything is missing: GitHub repo `canbmaj7/whorl` (public, issues on), labels, initial milestone issues.
2. M0: project skeleton (pyproject, `src/whorl`, pytest, ruff, CI on GitHub Actions), then fill the Commands section here.
3. M1: `whorl detect` (USB id from sysfs, libfprint support check via the installed hwdb, catalog lookup) with unit tests against fixture data.
4. Fable review of `docs/PLAN.md` before M2.
