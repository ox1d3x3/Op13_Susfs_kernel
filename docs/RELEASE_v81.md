# OP13 GKI Kernel — Release v81

Daily‑driver focused **Linux 6.6.89 GKI 2.0** kernel for the **OnePlus 13**
(`PJZ110` / `sun` / `sm8750`). Three lanes, one shared CCTV18 source/toolchain/SUSFS base.

> ⚠️ Flashing kernels can bootloop or soft‑brick a device. Back up `boot` /
> `init_boot` first. A new release line **may or may not be perfectly stable** —
> please report results. See the README "Safety" section before flashing.

## Pick your build

| Lane | Root | Flashable ZIP |
|---|---|---|
| **ReSukiSU** | ReSukiSU (SukiSU fork) | `AK3_OP13_ReSukiSU_..._v81_..._FLASH_THIS.zip` |
| **SukiSU Ultra** | SukiSU Ultra + KPM | `AK3_OP13_SukiSUUltra_..._v81_..._FLASH_THIS.zip` |
| **KernelSU‑Next** | KernelSU‑Next (pershoot dev‑susfs) | `AK3_OP13_..._v81_..._FLASH_THIS.zip` |

Optional add‑on (any lane): `OP13_ZRAM_LZ4_Tuner_v81_*.zip` — prefers LZ4 ZRAM,
safe (won't reset active ZRAM).

## What's enabled by default (v81)

**Feature‑rich daily‑driver profile** (`stock_daily` + `O2` + ThinLTO):

- **SUSFS** (CCTV18 `susfs4oki @ oki-android15-6.6`) — inline hook, SUS path /
  mount / kstat, try‑umount, `uname` + cmdline/bootconfig spoof, open‑redirect, sus‑map
- **Root:** ReSukiSU / SukiSU Ultra / KernelSU‑Next (one per lane)
- **KPM** support (SukiSU/ReSukiSU lanes)
- **ZRAM** LZ4/LZ4HC/ZSTD + writeback + multi‑comp, default compressor LZ4
- **ADIOS** I/O scheduler *(best‑effort)*
- **Baseband‑Guard** *(best‑effort)*
- **Netfilter extras:** TTL/HL target + match, IP_SET
- **NTSYNC** (Wine/proton‑style sync compatibility)
- **TCP BBR** — *new in v81:* compiled in (runtime‑selectable, not forced default)
- **Re:Kernel** — *new in v81:* enabled best‑effort (skips if the patch doesn't apply)
- **Stock‑mimic `uname -r`** so the build looks stock to apps

**Intentionally off:** FengChi/HMBIRD scheduler (experimental — kept off for
battery/stability), Droidspaces containers.

## Changes vs the previous line

- Unified all three lanes onto a single **v81** version string (was v77/v78/v80).
- **BBR** default `Off → On` (compiled in, not forced as system default).
- **Re:Kernel** default `Off → On` (best‑effort, dry‑run gated — safe to leave on).
- **KSUN** ccache raised `3G → 8G` (fewer cold rebuilds; first v81 KSUN run is a
  full rebuild because the cache namespace moved to v81).
- All upstream resources re‑verified resolvable at release time (source, CCTV18
  Clang 18 toolchain, SUSFS branch + patch, all three `setup.sh` scripts).

## Verify after flashing

```sh
uname -a
su -c 'cat /sys/block/zram0/comp_algorithm 2>/dev/null; cat /proc/swaps'
su -c 'zcat /proc/config.gz | grep -E "CONFIG_KSU(_SUSFS)?(_SUS_PATH|_SUS_MOUNT|_TRY_UMOUNT|_SPOOF_UNAME)?="'
su -c 'zcat /proc/config.gz | grep -E "CONFIG_TCP_CONG_BBR|CONFIG_NTSYNC"'
```

## If a toggle didn't take effect

Best‑effort patches (ADIOS, Baseband‑Guard, Re:Kernel, FengChi) apply **only if**
the matching CCTV18 patch dry‑runs cleanly. Check the run's
`_ox_debug/optional-patches.log` and `_ox_debug/config-scan.txt` to see exactly
what landed in the final `.config`.

## Credits

@cctv18 · SukiSU‑Ultra · ReSukiSU · KernelSU‑Next · @simonpunk (SUSFS) ·
@osm0sis (AnyKernel3) · @mrcxlinux · @WildKernels / @TheWildJames
