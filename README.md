# pod-sway

The `sway` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It provides a headless Sway
Wayland compositor, staged with a launcher wrapper and config and run as a
supervised service.

## What it provides

Installs the Sway compositor plus Xwayland, fuzzel, and the Mesa GL/Vulkan
drivers (Fedora and Arch/CachyOS), strips the `cap_sys_nice` file capability so
the binary can exec inside a rootless user namespace, and stages a `sway-wrapper`
launcher plus a config under the user's home. A supervisord service runs the
wrapper headless (`WLR_BACKENDS=headless`).

| Property | Value |
|---|---|
| Service | `sway` (`~/.local/bin/sway-wrapper`, `restart: always`, priority 10) |
| Requires | `pod-dbus` |
| Install files | `config`, `sway-wrapper` (copied into `~/.config/sway` / `~/.local/bin`) |
| Env | `WLR_BACKENDS=headless`, `WLR_HEADLESS_OUTPUTS=1`, `WLR_LIBINPUT_NO_DEVICES=1`, `XDG_RUNTIME_DIR=/tmp`, `WAYLAND_DISPLAY=wayland-0` |
| Env (accepted) | `XKB_DEFAULT_LAYOUT` (default `us`), `XKB_DEFAULT_VARIANT` |

The `libinput` backend is deliberately excluded: it needs a libseat session that
fails in rootless containers.

Every claim is observable: the binary, the package, the staged files, and the
live service.

## How to use it

Typically pulled in transitively via `chrome-sway` / `sway-desktop` rather than
used directly:

```yaml
my-desktop:
  candy:
    - '@github.com/opencharly/pod-sway:<tag>'
```

Override the keyboard layout:

```bash
charly config my-desktop -e XKB_DEFAULT_LAYOUT=de
```

## Verification

The candy's `check:` plan asserts `/usr/bin/sway`, the `sway` package, the
`sway-wrapper` launcher, the staged `~/.config/sway/config`, and — at deploy
scope — the running `sway` service.

## Layout

- `charly.yml` — the `sway:` candy entity (description, `require`, `env`,
  `env_accept`, `distro`, `service`, `plan`) plus its `skill:` entity.
- `config`, `sway-wrapper` — the staged compositor config and launcher.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-selkies:sway` — the candy properties, XWayland, the
  stale-IPC-socket cleanup, and the NVIDIA renderer notes.
- `/charly-infrastructure:dbus-layer` — the D-Bus session-bus dependency.
- `/charly-selkies:sway-desktop` — the full desktop composition (VNC).
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
