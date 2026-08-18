# OpenHarmony Port — status & record

This branch records source-level adaptations that let DeepSeek Harness run on a
HarmonyOS PC (OpenHarmony, aarch64, musl libc, Node v26.5.0, `process.platform === 'openharmony'`).

> **Upstream status (2026-08-18)**: the official repo does not accept external
> PRs yet (see CONTRIBUTING) and has Issues disabled; GitHub Discussions is the
> sanctioned channel. Community feedback draft:
> https://github.com/shizhonggang/dsh-harmonyos/blob/main/docs/upstream-feedback.md

## Commits on this branch

1. **subprocess-local**: `createProcessInspector` now recognizes `openharmony`
   (treated like linux) instead of throwing — previously the whole agent command
   channel failed to activate.
2. **terminal-bash**: default `shellPath` points at the HNP bash
   (`/data/service/hnp/bin/bash`; HarmonyOS has no `/bin/bash`). The value is
   already on `PATH` on these devices.
3. **session-persistence-jsonl**: session publish switched from `link()` to
   `rename()`. OpenHarmony forbids `linkat` (EPERM) — the platform-wide ban that
   affects every atomic-publish via hard link. `rejectExistingLog` still guards
   the EEXIST race.
4. **attachment-local**: same `link()` → `rename()` switch for attachment
   staging.

## Adaptations that cannot be source commits (npm-tree / machine level)

- **koffi** win32-ABI stub (`koffi/src/koffi/index.js`): `dsh-sandbox-windows-acl`
  is statically imported by `dsh-sandbox-local` on every platform, forcing koffi
  native load; no openharmony binding exists. Upstream fix would be a
  platform-conditional dynamic import.
- **node-pty**: no openharmony-arm64 prebuild — compile locally with clang.
- **sharp**: no openharmony runtime — use `@img/sharp-wasm32` WASM fallback.
- **Prebuilt `.node`** (rolldown/tsdown) fail `dlopen` without a `.codesign`
  signature — `binary-sign-tool sign -selfSign 1` (OpenHarmony SDK).
- `--expose-internals` required for the HMR service (cannot be passed via
  `NODE_OPTIONS`); launchers pass it on the command line.

## Reproducible install

Full install/upgrade/patch suite: https://github.com/shizhonggang/dsh-harmonyos
