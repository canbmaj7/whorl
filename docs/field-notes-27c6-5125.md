# Field notes: Goodix 27c6:5125 on Arch/CachyOS

First working install, 2026-11-10. Laptop: HONOR MagicBook 14 (DMI product `NMH-WCX9`), CachyOS, KDE Plasma, fprintd 1.94.5. These notes are the raw material for the future catalog recipe; they are evidence from one machine, not a guarantee.

## Starting state

- `lsusb`: `27c6:5125 Shenzhen Goodix Technology Co.,Ltd. Goodix Fingerprint Device`.
- libfprint 1.94.100 lists `27C6p5125` under "Known unsupported devices" in `60-autosuspend-libfprint-2.hwdb`. `fprintd-list` says "No devices available".
- The sensor is a Goodix "TLS" type: a TLS-PSK channel to the host, host-side image matching.

## Driver used

`goodix5125` by RuVl, submitted upstream as libfprint merge request !669 (not merged at the time). Source branch `goodix5125-mr` of `https://gitlab.freedesktop.org/RuVl/libfprint.git`; packaging in `https://github.com/RuVl/FingerprintDriver_27c6_5125` (`packaging/arch/PKGBUILD`, package `libfprint-goodix5125-git`, replaces `libfprint`). Matching is done on the host by a free C implementation (openchicago). Reading the code showed: no firmware update or erase commands, and the pairing key is never written unless `GOODIX5125_PROVISION_PSK=random` is set.

## Build and install notes

- Needs `meson` and `ninja` (not in the base install) plus `git`, `glib2-devel`, `systemd`.
- On a 7 GB machine a default parallel build crashed the terminal; `meson compile -C build -j2` worked.
- The package conflicts with `libfprint`; `pacman -U` needs `--ask 4` (answer yes to the conflict) when run non-interactively.
- Rollback: `sudo pacman -S libfprint`. Keep the built `.pkg.tar.zst`.
- Unattended installs need a polkit agent: `pkexec /usr/bin/pacman ...` shows a graphical prompt.

## Pairing key

- The driver reads the host key from `/var/lib/fprint/goodix5125/psk` (root only, mode 0600, 32 bytes or 64 hex digits).
- On this unit the sensor already held an unknown key. The all-zero key (which the Windows driver is reported to use) did not match; the driver then reports `hash differs` and writes nothing.
- Pairing with a new key is an **irreversible overwrite** of the sensor's write-only key slot. It was done once, deliberately, with the zero key in the file:
  `systemctl set-environment GOODIX5125_PROVISION_PSK=random`, restart fprintd, start `fprintd-enroll`, then `systemctl unset-environment ...` and restart fprintd again. With a key file already present the driver writes that key, not a random one.
- Consequence to tell users: other hosts paired with the old key (e.g. Windows Hello) may need to re-enroll.

## Enrollment and accuracy

- `fprintd-enroll -f right-index-finger` needed about 13 good stages; many `enroll-finger-not-centered` results are normal. Place the finger centered and hold it.
- Verification matched; roughly half of quick single touches miss, so allow a second try.

## PAM

- KDE lock screen: works out of the box via the shipped `/usr/lib/pam.d/kde-fingerprint`.
- `sudo` (`/etc/pam.d/sudo`) and polkit (`/etc/pam.d/polkit-1`, copy of `/usr/lib/pam.d/polkit-1` into `/etc`): add `auth sufficient pam_fprintd.so max-tries=2 timeout=10` as the first auth line. Back up first.
- Showing "Fingerprint matched" in `sudo`: `pam_echo` printed nothing in sudo. This stack worked:
  ```
  auth [success=1 default=ignore] pam_fprintd.so max-tries=2 timeout=10
  auth [success=2 default=ignore] pam_permit.so
  auth optional                   pam_exec.so quiet /etc/security/fprint-matched.sh
  auth sufficient                 pam_permit.so
  auth include system-auth
  ```
  where the script writes the message to `"$PAM_TTY"`. On failure the jump reaches `system-auth` and the password prompt.
- Power-button unlock offered by the driver is off by default and has a documented security caveat; leave it off.

## Language

pam_fprintd prompts follow `LC_MESSAGES`; a Turkish translation exists, the system was `en_US`.
