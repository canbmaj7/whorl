# Whorl

**Fingerprint setup for Linux, from sensor to `sudo`.**

Whorl is a wizard (GUI and CLI) that gets a fingerprint reader working on Linux and wires it into the places you actually use it: terminal (`sudo`), polkit prompts, the lock screen, and, through a password manager, the browser.

> **Status: planning / bootstrap.** There is no installable app yet. The plan is in [`docs/PLAN.md`](docs/PLAN.md). Issues, sensor reports and ideas are welcome.

## Why

Many fingerprint readers are not supported by upstream libfprint. Working drivers exist as scattered community forks, and nothing tells you which one fits your sensor, what it replaces on your system, or what it writes to the device. Then you still need to edit PAM by hand without locking yourself out. Whorl aims to make that a guided, reversible process.

## What it will do

1. Detect your reader and tell you whether it is supported, has a community driver, or is unsupported.
2. Install a community driver from a curated catalog, showing exactly what changes and keeping a way back.
3. Enroll your finger and verify it works.
4. Switch fingerprint login on or off for `sudo`, polkit prompts, lock screen and login, with backups and one-click rollback.
5. Guide you to biometric unlock of a password manager for the browser.

## What it will not do

- Write sensor drivers (new drivers belong upstream in libfprint or in community forks; Whorl catalogs them).
- Flash firmware. Ever.
- Promise per-website fingerprint prompts in Chrome or Firefox; Linux browsers do not support that today.

## First target

Arch/CachyOS + KDE Plasma + Goodix `27c6:5125`. Field notes from the first working install: [`docs/field-notes-27c6-5125.md`](docs/field-notes-27c6-5125.md).

## Contributing

- Have a fingerprint reader? Open a **Sensor report** issue (works / does not work, `lsusb` line, distro). That is how the catalog grows.
- Read [`CONTRIBUTING.md`](CONTRIBUTING.md) and the [plan](docs/PLAN.md).
- Security issues: see [`SECURITY.md`](SECURITY.md).

## License

GPL-3.0-or-later. See [`LICENSE`](LICENSE).
