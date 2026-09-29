# AGENTS.md — pod-sway

Standalone candy repo for the `sway` candy — the headless Sway Wayland
compositor, staged with a launcher wrapper and config and run as a supervised
service. The candy lives in `charly.yml` at the repo root plus its config and
wrapper.

Canonical files:

- `charly.yml` — the `sway:` candy entity (description, `require`, `env`,
  `env_accept`, `distro`, `service`, `plan`) and its `skill:` entity.
- `config` — the staged `~/.config/sway/config`.
- `sway-wrapper` — the launcher copied into `~/.local/bin`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-selkies:sway` — the owning skill: the candy properties, XWayland, the
  stale-IPC-socket cleanup, and the NVIDIA renderer notes. Load before editing,
  building, deploying, or troubleshooting this candy.
- `/charly-selkies:sway-desktop` — the composition that includes sway.
- `/charly-infrastructure:dbus-layer` — the D-Bus session-bus dependency.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `setcap:` / `copy:` / `check:`, service declarations).
- `/charly-check:check` — the check/R10 framework (`charly check box`,
  `charly check run <bed>`).
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; services, env).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert `/usr/bin/sway`, the `sway` package, the
  `sway-wrapper` launcher, the staged config, and — at deploy scope — the running
  `sway` service.

## Modify this repo

- Edit the `sway:` candy entity in `charly.yml`; the `skill:` entity in the same
  file is the owning skill's source — a candy change and its skill change land
  together.
- The `setcap` step strips `cap_sys_nice` from `/usr/bin/sway`; keep it, or the
  binary cannot exec inside the rootless user namespace.
- `config` and `sway-wrapper` are copied by `copy:` steps; keep their
  destinations and modes in step with the service `exec` path.
- Keep the headless env (`WLR_BACKENDS=headless`, `XDG_RUNTIME_DIR=/tmp`) in step
  with the supervisord `[program:sway]` block.
- The `skill:` entity is the source for `/charly-selkies:sway`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
