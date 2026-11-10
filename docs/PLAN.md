# Whorl: plan

Status: draft v0.1 (2026-11-10). Feedback welcome through issues.

## 1. Problem

Getting a fingerprint reader to work on Linux is a research project, not a setting:

- Many readers are not supported by upstream libfprint. Working drivers exist only as scattered community forks, merge requests and PKGBUILDs.
- Nothing tells a user which fork matches their sensor, what installing it replaces, or what it writes to the device.
- Once the device works, wiring it into `sudo`, polkit prompts, the lock screen and login is manual PAM editing, where one mistake can lock you out.
- Desktops ship an enrollment UI (KDE/GNOME settings) but nothing for steps before and after it.

The motivating case: a Goodix `27c6:5125` reader on an HONOR MagicBook 14. Finding the right driver, reading its code, building it, resolving an unknown pairing key and adding PAM lines took hours of expert work. See `docs/field-notes-27c6-5125.md`.

## 2. Goal and non-goals

**Goal:** install the app, open it, enroll a finger, and be able to use it in the terminal (`sudo`), polkit dialogs, the lock screen and, where possible, the browser, with every change reversible.

**Non-goals:**

- Writing sensor drivers or reverse engineering protocols. Whorl catalogs drivers, it does not author them.
- Firmware flashing of any kind.
- Replacing fprintd, libfprint or the desktop's own settings.
- Promising per-website fingerprint prompts in Chrome or Firefox (not possible today; see section 6).

## 3. User flow (wizard)

1. **Detect.** List USB (and SPI) fingerprint devices by vendor:product id.
2. **Status.** `supported` (libfprint already handles it), `community` (a catalog driver exists), or `unsupported` (link to the sensor-request issue).
3. **Driver.** For `community`, show exactly what will happen: source, package that gets replaced, whether the device is written to. Install only after explicit consent. Always keep the previous package for rollback.
4. **Enroll.** Choose a finger; progress ring with per-stage hints ("not centered", "lift and retry"); mandatory verification test at the end.
5. **Use it.** Per-place switches with rollback: terminal (`sudo`), polkit prompts, lock screen, login screen. Each shows the exact PAM change before applying.
6. **Browser.** Guide to password-manager biometric unlock (Bitwarden desktop + extension), with an honest explanation of the limits.

## 4. Architecture

```
GUI (PySide6/QML)      CLI (`whorl`)
        \                /
         whorl-core (Python library)
   detect | catalog | fprintd client | pam manager | installer
                 |                      |
         fprintd (D-Bus)        whorl-helper (privileged, via polkit)
```

- **detect:** reads sysfs/udev, compares ids with the installed libfprint hwdb (`60-autosuspend-libfprint-2.hwdb` lists supported devices and a "Known unsupported devices" section).
- **catalog:** `data/sensors.yaml`, schema in the file header. Community-editable; validated in CI.
- **fprintd client:** D-Bus `net.reactivated.Fprint` (Enroll, Verify, ListEnrolledFingers, DeleteEnrolledFingers).
- **pam manager:** templates for `sudo`, `polkit-1`, `kde` (lock screen uses `kde-fingerprint`), `sddm`; timestamped backups; dry-run diff; rollback.
- **installer:** per-distro recipes (first: Arch `makepkg` + `pacman -U`). Runs in the helper.
- **helper:** the only code that runs as root; small, audited, takes a fixed set of verbs, validates every argument, no shell strings.

## 5. Safety model

- Sensor writes are irreversible and per-device. They require a specific confirmation, and are logged.
- No firmware flashing, no telemetry, never touch enrolled fingerprint data or key files beyond what the driver itself does.
- PAM edits always use `sufficient`, a backup, and a verified rollback; the GUI warns to keep another privileged session open.
- Catalog `risk` flags are mandatory and shown before install.
- Reproducible installs: recipes pin source URL and commit; the user is shown what will be built.

## 6. Browser reality

Chrome and Firefox on Linux do not call fprintd for password fields, and WebAuthn platform authenticators with fingerprint are not available. The workable route is a password manager with system authentication, e.g. Bitwarden desktop "unlock with system authentication" (polkit), connected to its browser extension. Whorl will configure the polkit side and link the guide; it will not claim more.

## 7. Milestones

| # | Milestone | Output |
|---|---|---|
| M0 | Skeleton | pyproject, package, lint/test CI, release notes template |
| M1 | Detect + catalog | `whorl detect`, schema validation, fixtures from real `lsusb` dumps |
| M2 | Enroll + verify | fprintd client, minimal GUI with enroll/verify |
| M3 | PAM manager | `sudo`/polkit/lock-screen toggles with backup and rollback |
| M4 | Driver recipes | Arch recipe for Goodix `27c6:5125`, helper + polkit action |
| M5 | Packaging | AUR package, screenshots, docs site |
| M6 | Breadth | more sensors and distros through contributor-added catalog entries |

## 8. Testing

- Unit tests for detect/catalog/PAM manager using fixtures; PAM changes tested against copies in a temp root.
- libfprint ships a virtual image device for tests: use it for the fprintd client in CI.
- Real-hardware results are collected through the sensor-report issue template, which is how the catalog grows.

## 9. Risks

| Risk | Mitigation |
|---|---|
| Community drivers disappear or change | Pin commits; catalog entry records last-verified date; keep the built package for rollback |
| PAM edit locks the user out | `sufficient` only, backups, `pkexec` recovery path documented |
| Irreversible key write on a sensor | Explicit confirmation, catalog risk flag, log |
| Hardware coverage is tiny | Make adding a sensor a one-file contribution |
| Maintainers cannot test every sensor | Entries carry "tested on" evidence; unverified entries are marked |
