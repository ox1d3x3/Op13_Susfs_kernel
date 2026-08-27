# Known limitation: SUSFS SELinux policy hiding is NOT active

**Applies to:** both lanes (SukiSU Ultra, ReSukiSU), all versions.
**Cause:** upstream incompleteness in `cctv18/susfs4oki`, not a bug in this project.

## Symptom

Root-detection scanners may report findings such as:

- *"Dirty SELinux policy exposes LSPosed rule"*
- *"Enforcing with KSU context materialized"*

These are exactly the artefacts SUSFS's SELinux **policy-query hiding** is meant
to mask. That feature is compiled out of these builds.

## Why (verified, not assumed)

CCTV18's SUSFS patch adds calls into `security/selinux/hooks.c` and
`security/selinux/selinuxfs.c` that use three symbols:

| Symbol | Declared `extern` by SUSFS | Actually defined anywhere? |
|---|---|---|
| `backup_sepolicy` | yes | **no** |
| `security_context_to_sid_with_policy` | yes | **no** |
| `security_sid_to_context_with_policy` | yes | **no** |

Checked:

1. The SUSFS patch's own `security/selinux/ss/services.c` hunk — it adds only
   two unrelated wrappers (`security_dump_masked_av_fn`,
   `context_struct_compute_av_fn`). It does **not** define the three above.
2. The 6.6.118 kernel source (`security/selinux/ss/services.c`,
   `security/selinux/include/security.h`) — **zero** occurrences of any of them.

So they are declared and called but never defined. Building as-is fails at link:

```
ld.lld: error: undefined symbol: security_context_to_sid_with_policy
ld.lld: error: undefined symbol: backup_sepolicy
```

## What this project does about it

Both lanes run a **SELinux policy-query guard** that rewrites

```c
#ifdef CONFIG_KSU_SUSFS
```

to

```c
#if defined(CONFIG_KSU_SUSFS) && defined(CONFIG_KSU_SUSFS_SELINUX_POLICY_HIDE)
```

in those two files. `CONFIG_KSU_SUSFS_SELINUX_POLICY_HIDE` is not defined by any
root project, so those blocks compile out and the link succeeds.

**This guard is required.** Removing it does not enable the feature — it simply
breaks the build.

## What is NOT affected

- All **10** SUSFS options the root drivers actually define are set and active:
  `SUS_PATH`, `SUS_MOUNT`, `SUS_KSTAT`, `SUS_MAP`, `SPOOF_UNAME`,
  `SPOOF_CMDLINE_OR_BOOTCONFIG`, `HIDE_KSU_SUSFS_SYMBOLS`, `OPEN_REDIRECT`,
  plus `CONFIG_KSU_SUSFS` itself (`ENABLE_LOG` is deliberately off for battery).
  Confirm per build in `_ox_debug/susfs-config-audit.txt`.
- Both roots ship their own kernel-side `kernel/feature/selinux_hide.c`, which is
  independent of the SUSFS policy-query path and is compiled in.

## Resolution path

This needs CCTV18 to ship the missing definitions (or a companion patch that
provides `backup_sepolicy` and the `*_with_policy` helpers). When that lands,
the guard can be dropped and SELinux policy hiding will work.

Until then, hiding SELinux-policy artefacts is best handled in userspace
(module-level hiding / detection-specific mitigations), not by the kernel.
