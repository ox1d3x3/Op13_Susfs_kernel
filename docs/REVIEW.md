# Project Review — Op13_Susfs_kernel

A structured review of the repository's build system, with prioritized findings
and a roadmap. Scope: GitHub Actions workflows, the composite `action.yml`, the
`repo` manifest, packaging, and documentation. The kernel itself was not
compiled as part of this review (that requires the device sources, the CCTV18
toolchain and ~30 min of runner time); findings are based on static analysis of
the build pipeline and verification of the upstream projects it depends on.

---

## 1. Architecture at a glance

There are **two independent build systems** in this repo:

| System | How it builds | Root + SUSFS source | Status |
|---|---|---|---|
| `action.yml` + `manifest.xml` | Google `repo` sync via manifest | TheWildJames AnyKernel3/`kernel_patches` + `simonpunk/susfs4ksu` + KSU/KSUN `setup.sh` | **Not referenced by any workflow** |
| `Build_ReSukiSU.yml`, `Build SukiSU_Ultra.yml`, `Build KSUN.yml` | CCTV18 source zip via `aria2` + CCTV18 Clang18 toolchain | `cctv18/susfs4oki` + ReSukiSU / SukiSU‑Ultra / KernelSU‑Next `setup.sh` | **These are what actually run** |

The three device workflows are self‑contained and **~95% identical** — only the
root‑solution integration step and a handful of string labels differ.

## 2. Findings (prioritized)

### P0 — correctness / wasted effort
1. **`action.yml` ccache was capped at 1 GB** (`CCACHE_MAXSIZE=1G`) while the log
   claimed "12G". A GKI 6.6 object set is several GB, so a 1 GB cache evicts
   almost everything between runs and gives near‑zero hit rate. **Fixed → 8 GB**
   and the log now reflects the real value.
2. **`action.yml` is orphaned.** No workflow does `uses: ./`, so the entire
   portable builder + its OnePlus performance/battery patch set never executes.
   Either wire it into a caller workflow or treat it explicitly as the
   "portable/other‑device" path. It is now documented as the latter (header
   comment + README), so it is no longer silently dead.

### P1 — maintainability
3. **Three near‑duplicate 1.1–1.3k‑line workflows.** Any fix (e.g. a toolchain
   bump, an AnyKernel3 tweak, a defconfig change) must be made three times and is
   easy to get out of sync. Recommended fix in §3.
4. **Hardcoded `v77`/`v78`/`v80` version strings** are scattered through artifact
   names, ccache keys and the ZRAM module `versionCode`. A single `env` var per
   workflow (or per the reusable workflow) removes the drift.
5. **`Clean Caches.yml`** used `actions/github-script@v6` with Chinese‑only step
   names, and didn't paginate the cache list (silently misses >100 caches).
   **Rewritten**: v7, English, paginated, prefix‑filtered, scoped reset.

### P2 — robustness / hygiene
6. No CI linting — a typo in a 1.3k‑line workflow only surfaces after a long
   build is queued. **Added** `lint.yml` (actionlint + yamllint + shellcheck).
7. Empty `.github/assets/Test.txt` committed. **Removed.**
8. No `.gitignore` / `.editorconfig` / `LICENSE` / dependabot config. **Added.**
9. README described features (always‑on ADIOS, the full OnePlus perf‑patch list,
   HMBIRD/FengChi) that come from the **orphaned** `action.yml` path, not from
   the CCTV18 workflows that actually ship releases. The README has been rewritten
   to describe what each lane truly does. The bootloop steps were also misnumbered
   (1 → 3 → 4).

### P3 — behavioral notes (intentional, documented for awareness)
10. **`susfs_patch_policy: audited_cctv18`** (default) silently drops a known
    `fs/proc/task_mmu.c.rej`. The build continues, but the pagemap‑hiding portion
    of SUSFS for that file is not applied. This is a deliberate trade‑off for a
    clean build; it is worth knowing it slightly weakens hiding.
11. **`69_hide_stuff.patch` is intentionally skipped** in the SukiSU lane and the
    SUSFS SELinux policy‑query blocks are gated behind a never‑defined symbol,
    because they failed final link against the current SukiSU builtin. Reasonable
    for a "stable daily" lane, but it disables SUSFS's SELinux‑policy hiding.
12. **Inline (`CONFIG_KSU_NONE_HOOK`) hook is forced** for the SukiSU/ReSukiSU
    lanes. Upstream's current default for GKI 2.0 is the tracepoint syscall‑redirect
    hook. The workflows expose `hook_mode`, so this is selectable — just be aware
    the default deviates from upstream.
13. The CCTV18 toolchain release is named `oneplus_sm8650_toolchain` but the device
    is sm8750. This is fine (the Clang/arm64 toolchain is SoC‑agnostic), just
    non‑obvious; a one‑line comment helps future readers.

## 3. Recommended next step — de‑duplicate via a reusable workflow

The single biggest quality win is collapsing the three build lanes into one
`workflow_call` reusable workflow plus three thin callers. The differences are
small and well‑isolated:

```
inputs to vary:
  root_name        ReSukiSU | SukiSU Ultra | KernelSU-Next
  setup_sh_url     raw setup.sh for the chosen root solution
  setup_ref        branch/tag/commit passed to setup.sh
  tree_dir         expected on-disk dir after setup (KernelSU / KernelSU-Next / drivers/kernelsu)
  ccache_ns        ccache namespace (op13-resukisu | op13-sukisu | op13-ksun)
  artifact_tag     name fragment for the Image / AK3 zip
  banner_string    AnyKernel3 kernel.string
```

Everything else — disk preflight, dependency install, CCTV18 source/toolchain
download, SUSFS apply + policy audit, optional CCTV18 patch group, defconfig
tuning, build, AnyKernel3 packaging, ZRAM tuner module, summary/upload — is
identical and moves into the reusable workflow unchanged.

This was **not** auto‑applied in this pass because the three workflows encode
real, hard‑won debugging history (the `v77 reached final link then failed on…`
comments) and a faithful extraction cannot be runtime‑tested here. Doing it
blind risks breaking a working release pipeline. The safe path: create
`_op13-build.reusable.yml`, point **one** lane at it, confirm a green run, then
migrate the other two.

## 4. Verified upstream facts (as of this review)

- `SukiSU-Ultra/SukiSU-Ultra`, `ReSukiSU/ReSukiSU` (a SukiSU fork) and
  KernelSU‑Next are all active; their `kernel/setup.sh` integration model is
  current.
- ReSukiSU/SukiSU support GKI 2.0 (5.10+); the current upstream default hook on
  GKI 2.0 is the tracepoint syscall‑redirect hook (relevant to finding #12).

## 5. What changed in this pass

- `action.yml`: ccache 1G → 8G (+ honest logging); clarifying header.
- `Clean Caches.yml`: modernized (v7, paginated, English, scoped).
- `lint.yml`: **new** CI (actionlint + yamllint + shellcheck).
- `.github/.yamllint.yml`: **new** relaxed lint profile.
- `.gitignore`, `.editorconfig`, `.github/dependabot.yml`, `LICENSE`: **new**.
- `.github/assets/Test.txt`: removed.
- `README.md`: rewritten to match real behavior.
- The three build workflows: **left intact** (see §3 for why).

---

## 6. Release v81 — feature decisions (second pass)

All three build lanes were carefully re‑read end‑to‑end and every upstream
resource was re‑verified resolvable (`git ls-remote` + raw `setup.sh` 200s +
toolchain release assets 302 + SUSFS patch 200). The lanes were then updated to a
unified **v81** with a feature‑rich‑but‑stable daily‑driver default set.

| Toggle | Old default | v81 default | Why |
|---|---|---|---|
| BBR | Off | **On / enabled** | Compiled in, runtime‑selectable, not forced as system default → safe throughput option |
| Re:Kernel | Off | **On** | Best‑effort + dry‑run gated (skips on reject), commonly used for background management |
| KSUN ccache | 3 G | **8 G** | Matches the other lanes; fewer cold rebuilds |
| Version string | v77 / v78 / v80 | **v81** (unified) | Removes the cross‑lane drift |
| FengChi / HMBIRD | Off | Off (unchanged) | Maintainer flags it experimental / battery‑negative — stability wins |
| Droidspaces | Off | Off (unchanged) | Container namespaces add overhead, not needed for a phone |

Kept at their already‑good values: SUSFS inline, KPM, multi‑manager, ADIOS,
Baseband‑Guard, netfilter/IP_SET, ZRAM LZ4 + writeback + multi‑comp, NTSYNC,
unicode fix, O2, ThinLTO, `audited_cctv18` SUSFS policy, stock‑mimic `uname`.

Validation: `yamllint` clean and **zero** `bash -n` syntax errors across all
three edited workflows. The three workflows' structure/logic was **not**
restructured — only input defaults, the ccache size and the version strings were
changed; the SukiSU historical debugging notes (the `v77 reached final link…`
comments) were deliberately preserved.

Note: the first v81 **KSUN** run will be a full (uncached) compile because the
ccache namespace moved from `…_v77` to `…_v81`. SukiSU/ReSukiSU caches are
unaffected (their ccache dirs are unversioned).
