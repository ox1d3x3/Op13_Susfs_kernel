<!--
  Op13_Susfs_kernel — README
  Repo: ox1d3x3/Op13_Susfs_kernel
-->

<h1 align="center">If This Kernel is useful to you, a ⭐ helps others find it.</h1>

<div align="center">

<img src="./.github/assets/hero.svg" alt="OP13 GKI Kernel banner" width="100%" />

<br/>

<!-- Build lanes -->
<a href="../../actions/workflows/Build_ReSukiSU.yml">
  <img alt="ReSukiSU build" src="https://img.shields.io/badge/build-ReSukiSU-0A84FF?style=flat-square&logo=github" />
</a>
<a href="../../actions/workflows/Build%20SukiSU_Ultra.yml">
  <img alt="SukiSU Ultra build" src="https://img.shields.io/badge/build-SukiSU%20Ultra-0A84FF?style=flat-square&logo=github" />
</a>

<br/>

<a href="../../releases">
  <img alt="Release" src="https://img.shields.io/github/v/release/ox1d3x3/Op13_Susfs_kernel?display_name=tag&sort=date&style=flat-square&label=Release&logo=github" />
</a>
<a href="../../releases">
  <img alt="Downloads" src="https://img.shields.io/github/downloads/ox1d3x3/Op13_Susfs_kernel/total?style=flat-square&label=Downloads&logo=github" />
</a>
<a href="../../commits">
  <img alt="Last Commit" src="https://img.shields.io/github/last-commit/ox1d3x3/Op13_Susfs_kernel?style=flat-square&label=Last%20Commit&logo=git" />
</a>
<img alt="License" src="https://img.shields.io/badge/License-GPL--2.0--only-111111?style=flat-square" />

<br/>

<img alt="Device" src="https://img.shields.io/badge/Device-OnePlus%2013%20(PJZ110%20%C2%B7%20sun)-111111?style=flat-square" />
<img alt="Kernel" src="https://img.shields.io/badge/Kernel-6.6.118%20(GKI%202.0)-111111?style=flat-square" />
<img alt="Android" src="https://img.shields.io/badge/Android-15%20%2F%2016-111111?style=flat-square" />

</div>

---

## Overview

A daily‑driver focused **Linux 6.6 GKI 2.0** kernel for the **OnePlus 13**
(`PJZ110` / `sun` / `sm8750`), built entirely from GitHub Actions. The goal is a
**stock‑like, stable baseline** with modern root + hiding support and a few
opt‑in performance and battery tweaks — not a maxed‑out benchmark build.

Two root solutions are offered as separate build **lanes**, both sharing the
same CCTV18 OP13 source, toolchain and SUSFS tree:

| Lane | Workflow | Root solution | Pick it if… |
|---|---|---|---|
| **SukiSU Ultra** | `Build SukiSU_Ultra.yml` | [SukiSU‑Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra) | you want SukiSU Ultra with KPM + inline SUSFS |
| **ReSukiSU** | `Build_ReSukiSU.yml` | [ReSukiSU](https://github.com/ReSukiSU/ReSukiSU) (SukiSU fork) | you want the SukiSU feature set with the ReSukiSU manager stack |

> **KernelSU‑Next lane is DEPRECATED** (archived at
> [`docs/deprecated/Build_KSUN.yml.deprecated`](./docs/deprecated/Build_KSUN.yml.deprecated)).
> pershoot's `dev-susfs` branch was force‑pushed and now requires SUSFS helpers
> (`susfs_is/set/clear_current_proc_no_su`) that CCTV18's `susfs4oki` does not
> provide, so it cannot build against this project's SUSFS tree. No commit
> remains that has SUSFS support *without* that dependency. See
> [`docs/deprecated/WHY_KSUN_DEPRECATED.md`](./docs/deprecated/WHY_KSUN_DEPRECATED.md).

> **Builds are produced as GitHub Actions artifacts** on each manual run. If a
> tagged release exists, grab the ZIP from **[Releases](../../releases)**;
> otherwise open the relevant **Actions** run and download the artifact.

> **Current release line: `v81`** — feature‑rich daily‑driver defaults
> (SUSFS inline + KPM + ADIOS + Baseband‑Guard + ZRAM LZ4 + NTSYNC + netfilter,
> with BBR and Re:Kernel now on by default). FengChi/HMBIRD stays off for
> stability. See [`docs/RELEASE_v81.md`](./docs/RELEASE_v81.md).

---

## Build matrix

Both lanes are **manually triggered** (`workflow_dispatch`) and share:

- **Source:** `cctv18/android_kernel_common_oneplus_sm8750` @ `oneplus/sm8750_b_16.0.0_oneplus_13_6.6.118`
- **Toolchain:** CCTV18 prebuilt **LLVM/Clang 18** (`LLVM-Clang18-r510928`) + build‑tools
- **SUSFS:** `cctv18/susfs4oki` @ `oki-android15-6.6` (single source — no mixed SUSFS trees)
- **Packaging:** [osm0sis/AnyKernel3](https://github.com/osm0sis/AnyKernel3), boot detection fixed for the A/B `sun` layout
- **Caching:** per‑lane `ccache` (zstd), restored across runs

### Key inputs (set them when you press *Run workflow*)

| Input | What it does | Sensible default |
|---|---|---|
| `profile` | `stock_daily` / `stock_plus` / `minimal_safe` / `debug_only` | `stock_daily` |
| root ref | `sukisu_ref` / `resukisu_ref` | **pinned commits** (see “Why the root refs are pinned”) |
| `SUSFS` / `susfs_enable` | Enable CCTV18 SUSFS | On |
| `susfs_patch_policy` | `strict` / `audited_cctv18` / `native_cctv18` | `audited_cctv18` |
| `hook_mode` | Hook target (inline SUSFS vs. auto/kprobe) | `inline_susfs` / `cctv18_susfs_auto` |
| `kernel_suffix` / `kernel_name` | Stock‑mimic `LOCALVERSION` (controls `uname -r`) | stock‑like string |
| `optimise` | `O2` (stable) or `O3` (test‑only) | `O2` |
| `zram_comp` | Preferred ZRAM compressor | `lz4` |
| `bbr` / `bbr_mode` | Compile in TCP BBR (selectable at runtime, not forced default) | **On / enabled** |
| `proxy` / `better_net` | Netfilter / TTL / IP_SET options | On |
| `adios`, `bbg`, `rekernel` | Optional CCTV18 patches (best‑effort, see note) | **On** |
| `fengchi` / HMBIRD | Experimental scheduler | Off (stability) |
| `clean_ccache` | Wipe ccache before building | false |

> **Optional patches are best‑effort.** `adios`, `baseband_guard`, `rekernel`,
> `fengchi`/HMBIRD etc. are applied **only if** a matching CCTV18 patch exists
> and dry‑runs cleanly; otherwise the step logs a warning and skips, so the build
> still completes. Don't assume a toggle is "on" in the final Image — confirm in
> the run's `_ox_debug/optional-patches.log` and `config-scan.txt`.

### Outputs

Each run uploads one artifact containing:

- `Image_OP13_…` — raw kernel image (+ `.sha256`)
- `AK3_OP13_…_FLASH_THIS.zip` — AnyKernel3 flashable ZIP (+ `.sha256`)
- `OP13_ZRAM_LZ4_Tuner_…zip` — optional, separate ZRAM/LZ4 tuner module
- `_ox_debug/` — logs, final `.config`, rejects, UTS release, patch audit

---

## Features

What each lane reliably provides:

- **GKI 2.0** packaging as an AnyKernel3 flashable ZIP
- **Root:** ReSukiSU **or** SukiSU Ultra **or** KernelSU‑Next (one per lane)
- **SUSFS** (CCTV18 `susfs4oki`) with inline hook by default — SUS path / mount /
  kstat, try‑umount, `uname` spoof, cmdline/bootconfig spoof, open‑redirect, sus‑map
- **ZRAM** with LZ4 / LZ4HC / ZSTD kernel support and a preferred compressor
- **Netfilter extras** when `proxy`/`better_net` is on: TTL/HL target + match, IP_SET
- **KPM** support on the SukiSU/ReSukiSU lanes (kernel support only — no modules bundled)
- **Stock‑mimic `uname -r`** via `kernel_suffix`, so the build looks stock to apps

Opt‑in / best‑effort (only if the CCTV18 patch applies cleanly): **ADIOS** I/O
scheduler, **Baseband‑Guard**, **Re:Kernel**, **FengChi/HMBIRD** scheduler,
**NTSYNC**, **BBR**.

> **Hiding note:** the SukiSU stable lane intentionally **skips** the SUSFS
> SELinux‑policy hiding blocks (`69_hide_stuff` and the policy‑query wrappers)
> because they failed final link against the current SukiSU builtin. Core SUSFS +
> inline hook stay enabled; SELinux‑policy hiding is reduced. See `docs/REVIEW.md`.

---

## Safety first

> ⚠️ Flashing kernels can cause **bootloops**, **soft‑bricks**, **data loss** and
> **instability**. You are responsible for what you flash. If you can't recover
> from a bootloop, **don't flash**.

**Back up before flashing.** In recovery, back up `boot` and `init_boot`
(and `vbmeta` if your setup uses it).

**If you bootloop:**
1. Reboot to recovery.
2. Restore your `boot` / `init_boot` backup. (If recovery is broken, flash the
   stock `boot.img`, then reboot to recovery.)
3. Reboot.

---

## Install

1. Download the correct `AK3_OP13_…_FLASH_THIS.zip` from **Releases** (or the
   **Actions** run artifact).
2. Flash with a kernel flasher or your KernelSU/SukiSU/Magisk manager's
   AnyKernel3 flasher.
3. Reboot.
4. Install/update the matching SUSFS module for your root manager.

**Force LZ4 ZRAM (optional):** flash the separate `OP13_ZRAM_LZ4_Tuner` module,
reboot, and it prefers LZ4 when supported. It is safe by design — it will **not**
reset ZRAM if it's already active.

> Switching lanes (e.g. SukiSU → ReSukiSU)? Reinstall the SUSFS module afterwards to
> clear stale module state.

---

## Verify after flashing

```sh
# Kernel version / stock-mimic uname
uname -a

# ZRAM + active compressor
su -c 'cat /sys/block/zram0/comp_algorithm 2>/dev/null; cat /proc/swaps'

# SUSFS kernel flags
su -c 'zcat /proc/config.gz | grep -E "CONFIG_KSU(_SUSFS)?(_SUS_PATH|_SUS_MOUNT|_TRY_UMOUNT|_SPOOF_UNAME)?="'
```

---

## Troubleshooting

- **SUSFS module "operation not supported" / `0x555e1`** — reinstall the SUSFS
  module after flashing; if you had a different one before, uninstall it, reboot,
  then install the correct one.
- **No `zram.ko`** — ZRAM is built‑in here; check `/proc/swaps` and
  `/sys/block/zram0/comp_algorithm` instead of looking for a `.ko`.
- **Bootloop** — restore `boot` / `init_boot`, then return to the `stock_daily`
  profile and re‑introduce options one at a time.
- **A toggle didn't take effect** — open the run's `_ox_debug/config-scan.txt`
  and `optional-patches.log`; best‑effort patches skip if they don't apply.

---

## Build it yourself

Open **Actions → pick a lane → Run workflow**, choose your inputs, and download
the artifact when it finishes. Start from `stock_daily` + `O2` + inline SUSFS for
a stable result, then experiment from a fork.

### Maintenance utilities
- **Clean Caches** (`Clean Caches.yml`) — purge `ccache-*` Actions caches.
- **Cleanup CCache** (`clean-up.yml`) — bulk cache/run cleanup with filters.
- **Lint Workflows** (`lint.yml`) — actionlint + yamllint + shellcheck on every
  workflow change, so pipeline typos are caught before a build is queued.

### Portable composite action (advanced)
`action.yml` is a separate, **device‑agnostic** builder that syncs sources with
Google's `repo` tool (`manifest.xml`), pulls AnyKernel3/`kernel_patches` from
TheWildJames + SUSFS from `simonpunk/susfs4ksu`, and applies a broad OnePlus
performance/battery patch set. It is intended for forks/other devices and is
**not** what the OP13 release lanes above use. See `docs/REVIEW.md` for details
and a roadmap to unify the lanes.

---

## Credits

[@cctv18](https://github.com/cctv18) ·
[SukiSU‑Ultra](https://github.com/SukiSU-Ultra) ·
[ReSukiSU](https://github.com/ReSukiSU) ·
[KernelSU‑Next](https://github.com/KernelSU-Next) ·
[@simonpunk (SUSFS)](https://gitlab.com/simonpunk/susfs4ksu) ·
[@osm0sis (AnyKernel3)](https://github.com/osm0sis) ·
[@mrcxlinux](https://github.com/mrcxlinux) ·
[@WildKernels / @TheWildJames](https://github.com/WildKernels)

---

## License

Distributed under **GPL‑2.0‑only** (see [`LICENSE`](./LICENSE)), matching the
upstream Linux kernel and SUSFS sources this project builds on. Build glue and
workflows are covered by the same license. Bundled third‑party components
(AnyKernel3, root solutions, CCTV18 patches) remain under their own upstream
licenses — follow those where applicable.
