# Why the KernelSU‑Next (KSUN) lane was deprecated

**Status:** deprecated as of release line `v95`.
**Archived workflow:** `docs/deprecated/Build_KSUN.yml.deprecated`
(moved out of `.github/workflows/` so GitHub neither runs nor validates it).

## Short version

pershoot's `KernelSU-Next` `dev-susfs` branch now requires SUSFS helper
functions that CCTV18's `susfs4oki` does not provide. The lane cannot be built
against this project's SUSFS tree, and there is no commit left to pin to that
avoids the problem.

## The evidence

### 1. The build failure

```
drivers/kernelsu/feature/sucompat.c:197: error: call to undeclared function
    'susfs_is_current_proc_no_su'
drivers/kernelsu/feature/sucompat.c:200: error: call to undeclared function
    'susfs_set_current_proc_no_su'
note: did you mean 'susfs_is_current_proc_umounted'?
include/linux/susfs_def.h:96: 'susfs_is_current_proc_umounted' declared here
```

The compiler's own suggestion is the tell: our `susfs_def.h` (from
`cctv18/susfs4oki`) has `susfs_is_current_proc_umounted` but **not**
`susfs_is_current_proc_no_su`.

### 2. The symbols do not exist in CCTV18's SUSFS

Searched the entire `cctv18/susfs4oki` repository on **both** the stable
`oki-android15-6.6` branch and the `oki-android15-6.6-dev` branch:
zero matches for `no_su`. Both branches declare `SUSFS_VERSION "v2.2.0"`.

### 3. KSUN expects a *different* SUSFS

The commit that introduced the dependency —
`3dba1bd "kernel: susfs (v2.2.0): Introduce SuSFS"` — names its source in the
commit message:

```
https://gitlab.com/pershoot/susfs4ksu
```

pershoot maintains their own SUSFS fork. Both forks call themselves "v2.2.0",
but they are not the same code. We build with CCTV18's kernel source, so
CCTV18's matching SUSFS fork is the correct pairing for the other lanes.

### 4. Pinning cannot fix it

`dev-susfs` was **force‑pushed**. The commits this project previously used
(`1200c466`, `70fbe0a0`) no longer exist in the branch history. Walking what
remains:

| Commit | SUSFS support | `no_su` dependency |
|---|---|---|
| `8f77a40` (before SuSFS introduce) | **none** (0 files) | none |
| `3dba1bd` (SuSFS introduce) | yes (23 files) | **yes** |
| `841686e` (HEAD) | yes (23 files) | **yes** |

SUSFS support and the incompatible dependency arrived in the *same commit*.
So the options are "SUSFS with a dependency we can't satisfy" or "no SUSFS at
all" — neither is a usable daily driver for this project.

## If you want to revive it

The archived workflow has `SUSFS_REPO_URL` as an env var at the top:

```yaml
SUSFS_REPO_URL: https://github.com/cctv18/susfs4oki.git
```

Point it at `https://gitlab.com/pershoot/susfs4ksu.git` and set `susfs_ref` to
that fork's matching branch. GitHub runners can reach gitlab.com.

**This pairing is UNVERIFIED.** It was never tested, because gitlab was not
reachable from the environment these workflows were authored in. Treat it as
an experiment, not a supported path.

## What to use instead

The **SukiSU Ultra** and **ReSukiSU** lanes both build and are the supported
paths. Both are pinned to known‑good commits and use CCTV18's SUSFS —
the same SUSFS the popular OPlus Kernel Lab builds use for these devices.
