# Contributing to Whorl

Thanks for helping. The most valuable contribution right now is **hardware evidence**.

## The easiest way to help: report your sensor

Open a **Sensor report** issue with:

- the `lsusb` line of your reader (or `lsusb | grep -i -E "fingerprint|goodix|synaptics|elan|validity|fpc"`),
- laptop model and distribution,
- what you tried and whether it works (`fprintd-list $USER`, `fprintd-enroll`, `fprintd-verify`).

Please do not paste fingerprint data, key files or anything from `/var/lib/fprint`.

## Adding a sensor to the catalog

Catalog entries live in `data/sensors.yaml` (schema in the file header). A good entry has:

- the id, a status (`supported`, `community`, `unsupported`),
- links to the driver source, pinned to a commit or tag where possible,
- honest `risk` flags (`writes_key`, `replaces_system_package`); entries needing firmware flashing are not accepted,
- at least one `tested_on` record with a date and result.

## Code contributions

The project is at the planning stage; see `docs/PLAN.md` for milestones. Before large work, open an issue or comment on an existing one so we can agree on the approach.

Ground rules:

- Safety invariants in `docs/PLAN.md` section 5 are not negotiable: no implicit writes to sensors, no firmware flashing, no telemetry, PAM edits are `sufficient` with a backup and rollback.
- Privileged code (the helper) stays small: fixed verbs, validated arguments, no shell strings.
- Keep pull requests focused, describe how you tested, and say which hardware/distro you ran on.
- Language: English for code, docs, issues and commits.

## Conduct

Be kind and assume good faith. See `CODE_OF_CONDUCT.md`.
