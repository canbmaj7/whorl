# Security policy

Whorl edits authentication configuration (PAM) and will run a small privileged helper, so security reports are taken seriously.

## Reporting

Please do **not** open a public issue for a vulnerability. Use GitHub's private vulnerability reporting ("Security" tab, "Report a vulnerability") on this repository. Include the affected version or commit, what an attacker can do, and steps to reproduce.

## In scope (examples)

- A way to make the helper write outside its allowed files or run unexpected commands.
- PAM changes that can bypass authentication or lock a user out unrecoverably.
- Any path that exposes fingerprint templates, pairing keys or other secrets.
- Catalog entries that point to malicious sources.

## Out of scope

- Weaknesses of third-party drivers or of fingerprint authentication in general (report those upstream).
- Issues that need an already-root attacker.

There is no bug bounty. We will acknowledge reports as soon as we can and credit reporters who want it.
