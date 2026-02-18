<!--
  Kernel-focused README for ox1d3x3/Op13_Susfs_kernel
  Keep URLs minimal & badges live.
-->

<div align="center">

# OP13 GKI Kernel
### KernelSU-Next • SukiSU Ultra • SUSFS • LZ4 ZRAM

**Daily-driver GKI kernel builds for OnePlus 13** with clean feature set, stability-first defaults, and optional performance goodies.

<br/>

[![Release](https://img.shields.io/github/v/release/ox1d3x3/Op13_Susfs_kernel?style=for-the-badge&logo=github&label=Release&color=2ea043)](https://github.com/ox1d3x3/Op13_Susfs_kernel/releases)
[![Downloads](https://img.shields.io/github/downloads/ox1d3x3/Op13_Susfs_kernel/total?style=for-the-badge&label=Downloads&color=8250df)](https://github.com/ox1d3x3/Op13_Susfs_kernel/releases)
[![Last Commit](https://img.shields.io/github/last-commit/ox1d3x3/Op13_Susfs_kernel?style=for-the-badge&label=Last%20Commit&logo=git&logoColor=white&color=0e8a16)](https://github.com/ox1d3x3/Op13_Susfs_kernel/commits)

![Device](https://img.shields.io/badge/Device-OnePlus%2013%20(PJZ110)-1f6feb?style=for-the-badge)
![Kernel](https://img.shields.io/badge/Kernel-6.6%20(GKI%202.0)-1f6feb?style=for-the-badge)
![Android](https://img.shields.io/badge/Android-15%20/%2016-1f6feb?style=for-the-badge)

</div>

---

## ✨ What is this?

This repo publishes **GKI+SUSFS+LZ4** for **OnePlus 13 (SUN / sm8750)** based on **Android 6.6 GKI**.

You get two release variants:

- **KernelSU-Next builds** (KSU Next)
- **SukiSU Ultra builds** (Ultra / builtin)

> Both variants can include **SUSFS**, and you can choose **battery / balanced / performance** style presets in releases.

---

## 🔥 Features & goodies

Core:
- **Root framework**: KernelSU-Next or SukiSU Ultra (depending on release)
- **SUSFS v2 support** *(when enabled in a release)*
- **GKI-compatible packaging** (AnyKernel3 flashable zip)
- **Clean, stability-first defaults** (daily driver focus)

Optional toggles that may be present in a given build (check release notes):
- **ZRAM + LZ4** (including optional “ZRAM LZ4 perf” patch)
- **ADIOS I/O scheduler**
- **Baseband Guard (BBG)**
- **Netfilter extras**: TTL target/match, IP_SET
- **TCP congestion control**: optional BBR
- **HMBIRD / FengChi scheduler bundle** *(best-effort; may be already present or skipped depending on kernel base)*

✅ **Release notes / changelog** will always be the source of truth for what’s enabled.

NOTE: All the releases are "STOCK/Balanced" profile based. If you want to experiepment with different profile, please fork it and run it. Fork Guide below.

---

## 📥 Downloads

➡️ Grab the latest builds here:  
**Releases:** https://github.com/ox1d3x3/Op13_Susfs_kernel/releases
---

## ⚠️ Warning / Disclaimer

Flashing kernels can cause **bootloops**, **soft-bricks**, **data loss**, and **instability**.

You are responsible for what you flash.

If you don’t know how to recover from a bootloop, **do not flash**.

---

## 🛟 Backup (do this first)

Before flashing any kernel, make backups in recovery (recommended):

Backup partitions:
- **boot**
- **init_boot**

If your recovery supports it, also back up:
- **vbmeta** *(some setups)*

If you bootloop:
1. Flash sotck boot.img
2. Reboot to recovery
3. Restore the backup

---

## 📲 Install guide (AnyKernel3 ZIP)

### Recommended (Recovery)
1. Download the latest Releases zip
2. Flash through Anykernel3 or Kernel flasher
3. Reboot
4. ENJOY!!


---

## ✅ Post-flash verification (quick checks)

### Kernel version
```sh
uname -a
```

### Confirm ZRAM + compressor availability
```sh
su -c 'ls -l /sys/block | grep zram || true'
su -c 'cat /sys/block/zram0/comp_algorithm 2>/dev/null || true'
su -c 'cat /proc/swaps'
```

### Confirm SUSFS flags (kernel-side)
```sh
su -c 'zcat /proc/config.gz | grep -E "CONFIG_KSU_SUSFS|TRY_UMOUNT|SPOOF_UNAME|CMDLINE_OR_BOOTCONFIG"'
```

---

## 🩺 Troubleshooting

Fix:
- Reinstall the SUSFS module after flashing a new kernel variant/version.
- If you previously had a different SUSFS module, remove it and install the correct version again.

### Bootloop
- Restore **boot/init_boot** backups
- If you changed many features at once, go back to a **balanced baseline** release and add changes one-by-one

### zram.ko not found
Sometimes ZRAM is compiled **built-in** instead of module, so there is no `zram.ko`.  
Use the runtime checks above (`/sys/block/zram0/...`) instead of relying on `zram.ko`.


## Build Yours OWN - BYO
1. Fork it
2. Actions
3. Select the Build
4. Select the Features of your choice
5. Start and wait


---

## 🙌 Credits

[Xiaomichael](https://github.com/xiaomichael) | [cctv18](https://github.com/cctv18) | [vc-teahouse](https://github.com/vc-teahouse) | [Numbersf](https://github.com/Numbersf) | [mrcxlinux](https://github.com/mrcxlinux)

---

## 📜 License

This repo distributes kernel builds and build glue.  
Follow upstream kernel sources and patch project licenses where applicable.
