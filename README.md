# KernelSU Next Module Template

Bare-bones starter for [KernelSU Next](https://github.com/KernelSU-Next/KernelSU-Next) modules.

## Quick start

1. Fork this repo (prefer a **lowercase** repo name, or set a custom `id=` in `module.prop`)
2. Edit `name` and `description` in `module.prop`
3. Edit `customize.sh` and/or `service.sh` if needed
4. Add your module files (`system/`, `bin/`, `webroot/`, etc.)
5. Add a `## [vX.Y.Z]` section to `CHANGELOG.md` (required for release notes)
6. Push to the default branch, then tag that commit and push: `git tag vX.Y.Z && git push origin vX.Y.Z`

Or run the **Release** workflow manually on the default branch and enter the version.

Tags must point to commits on the default branch. The release workflow rejects tags from side branches.

## Layout

```text
.github/workflows/
  ci.yml             # validation
  release.yml        # build + GitHub Release
build/               # release ZIP output (gitignored)
CHANGELOG.md         # release notes source
customize.sh         # install-time script (optional)
LICENSE              # MIT license
module.prop          # module metadata
scripts/
  validate.sh
service.sh           # boot-time script (optional)
update.json          # OTA metadata (updated on release)
```

## module.prop

Edit `name` and `description`. Other fields use placeholders:

| Field | Placeholder | At release |
|-------|-------------|------------|
| `id` | `INJECTED` | Repo name — skipped if you set your own value |
| `author` | `INJECTED` | Repo owner — skipped if you set your own value |
| `version` | `PIPELINE` | Tag or manual release input |
| `versionCode` | `PIPELINE` | `YYYYMMDD` + same-day patch digit (`0`–`9`), e.g. `202606241` |
| `updateJson` | `PIPELINE` | Raw URL to this repo's `update.json` on the default branch |

Release patches `module.prop` in the CI workspace only (for the ZIP). Nothing in `module.prop` is committed.

If `id` is `INJECTED`, the repo name must match `^[a-zA-Z][a-zA-Z0-9._-]+$`. CI validates this automatically. Lowercase names are recommended.

## Scripts

Both are optional — delete either one if you do not need it.

- **customize.sh** — sourced during install. Set `SKIPUNZIP=0` unless you handle extraction yourself.
- **service.sh** — runs at `late_start`. Use `MODDIR=${0%/*}` for paths. Must not block boot.

## update.json

Used by the KernelSU manager for OTA updates. The release workflow updates and commits this file after the GitHub Release is created. `versionCode: 0` means no release yet. Same-day releases increment the trailing patch digit (`202606240` → `202606241`).

## CI/CD

| Workflow | Trigger | What it does |
|----------|---------|--------------|
| `ci.yml` | Push/PR to `master` or `main` | Runs `scripts/validate.sh` |
| `release.yml` | Tag `v*` or manual | Patches `module.prop`, builds ZIP, creates GitHub Release, then commits `update.json` |

CI ignores pushes that only change `update.json`.


If your default branch is not `master` or `main`, edit the `branches` list in `.github/workflows/ci.yml`.

### Branch protection

If the default branch is protected, allow `github-actions[bot]` to bypass branch protection or push to the default branch; otherwise the release workflow cannot commit `update.json`.

Local validation (requires [shellcheck](https://www.shellcheck.net/) and [jq](https://jqlang.org/)):

```bash
bash scripts/validate.sh
```

Check release notes before tagging:

```bash
RELEASE_VERSION=v1.0.0 bash scripts/validate.sh
```

## License

MIT — see [LICENSE](LICENSE).
