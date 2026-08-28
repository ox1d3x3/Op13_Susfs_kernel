# SELinux policy hiding — how it works and how to turn it on

> **This supersedes the earlier `KNOWN_SUSFS_SELINUX_LIMITATION.md`, which was
> wrong.** That document concluded the required symbols existed nowhere. They do
> exist — in the root driver, not in the SUSFS patch. Corrected below.

## Symptom

Root-detection scanners report things like:

- *"Dirty SELinux policy exposes LSPosed rule"*
- *"Enforcing with KSU context materialized"* — KSU SELinux contexts visible in
  the live policy

## How the feature actually works

CCTV18's SUSFS patch adds calls into `security/selinux/hooks.c` and
`security/selinux/selinuxfs.c` that answer policy queries from a *pristine
backup* of the policy instead of the live (KSU-modified) one. It declares three
symbols `extern` but does **not** define them:

- `backup_sepolicy`
- `security_context_to_sid_with_policy`
- `security_sid_to_context_with_policy`

The definitions come from the **root driver**, not from SUSFS:

| Symbol | Defined in |
|---|---|
| `backup_sepolicy` | `drivers/kernelsu/selinux/rules.c` (line 3, global) |
| `security_context_to_sid_with_policy` | `drivers/kernelsu/feature/selinux_hide.c` |
| `security_sid_to_context_with_policy` | `drivers/kernelsu/feature/selinux_hide.c` |
| `security_compute_av_user_with_policy` | `drivers/kernelsu/feature/selinux_hide.c` |

Both files are compiled unconditionally — they appear in the build log as
`CC drivers/kernelsu/feature/selinux_hide.o` and
`CC drivers/kernelsu/selinux/rules.o`.

## What went wrong here (build side)

At the **v77-era** root revision the driver did **not** yet define those symbols,
so the link failed:

```
ld.lld: error: undefined symbol: security_context_to_sid_with_policy
ld.lld: error: undefined symbol: backup_sepolicy
```

A guard was added to the **SukiSU lane** that rewrote

```c
#ifdef CONFIG_KSU_SUSFS
```

to

```c
#if defined(CONFIG_KSU_SUSFS) && defined(CONFIG_KSU_SUSFS_SELINUX_POLICY_HIDE)
```

in those two files, compiling the blocks out. Since no root project defines
`CONFIG_KSU_SUSFS_SELINUX_POLICY_HIDE`, that permanently disabled the feature.

The driver has since gained the definitions, but the guard kept firing
**unconditionally** — so SELinux policy hiding stayed off long after the reason
for it had gone away.

### Fixed in v99

The guard is now **auto-detecting**. Before applying, it checks whether the root
driver supplies the symbols:

- **Symbols present** → guard is **skipped**, SELinux policy hiding stays
  **enabled**.
- **Symbols absent** → guard applies, preserving the original link-failure
  protection.

Each build records the outcome in `_ox_debug/selinux-hide-status.txt`.

Note: the **ReSukiSU lane never had this guard**, so its SELinux policy-query
code has always been compiled in.

## What you must still do (runtime side)

Compiling the code in is only half of it. In `selinux_hide.c`:

```c
bool ksu_selinux_hide_running __read_mostly = false;
```

The feature is **off by default** and is toggled at runtime through the
manager's feature system (`selinux_hide_feature_set`). So after flashing a v99+
build you must **enable the SELinux hide feature in the SukiSU manager**.
If it stays off, scanners will keep seeing KSU contexts even though the kernel
supports hiding them.

## What this cannot fix

The *"Dirty SELinux policy exposes LSPosed rule"* finding is largely about
**LSPosed itself** — loaded Xposed classes, `XposedBridge` fields, binder bridge
replies, stack-trace signatures in your app's runtime. Those are userspace
artefacts of an active hook framework. No kernel feature can mask them; that is
LSPosed/module-level hiding territory.

SELinux policy hiding addresses the *policy-surface* half (KSU contexts, dirty
policy rules), not the presence of a hook framework inside a process.
