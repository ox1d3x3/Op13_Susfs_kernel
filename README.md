<!--
  README.md — Ox1d3x3 OP13 Kernel Build Pipeline
  Tip: Replace badge links/repo name if you fork.
-->

<div align="center">

# OnePlus 13 GKI Kernel Builder  
### SukiSU Ultra • KernelSU NEXT • SUSFS • LZ4 ZRAM • AnyKernel3

Build **flashable OP13 GKI kernels** via **GitHub Actions** with safe presets for **battery / balanced / performance**, plus optional feature toggles.

<br/>

<!-- Badges (edit repo path if you fork) -->
![Build](https://img.shields.io/github/actions/workflow/status/ox1d3x3/Op13_Susfs_kernel/Build%20SukiSU_Ultra.yml?style=for-the-badge)
![Stars](https://img.shields.io/github/stars/ox1d3x3/Op13_Susfs_kernel?style=for-the-badge)
![Forks](https://img.shields.io/github/forks/ox1d3x3/Op13_Susfs_kernel?style=for-the-badge)
![Last commit](https://img.shields.io/github/last-commit/ox1d3x3/Op13_Susfs_kernel?style=for-the-badge)

</div>

---

## ✅ What this repo is

This project is a **GitHub Actions kernel build pipeline** for **OnePlus 13 (PJZ110 / sm8750)** GKI kernels:

- Integrates **SukiSU Ultra** using its **official setup.sh (builtin)**  
- Optional **SUSFS** support (with **r25+ module compatibility** when using **external mode**)  
- Optional **LZ4** support + optional **flashable zram-module.zip**  
- Packages output as **AnyKernel3 flashable ZIP** + **SHA256**  
- Generates a **WildKernel-style “Build Summary”** table in the Actions run summary

---

## ⚙️ Kernel / Device Info

| Item | Value |
|---|---|
| Device | OnePlus 13 (PJZ110) |
| Chipset | sm8750 (Snapdragon 8 Elite) |
| Kernel | 6.6 (GKI 2.0) |
| Android | 15–16 |
| ROMs | OxygenOS / ColorOS |

---

## 🚀 Quick Start (Build on GitHub)

1. **Fork** this repo (or use it directly).
2. Go to **Actions** → select the workflow (e.g. **Build SukiSU_Ultra**).
3. Click **Run workflow** and choose your options.
4. When it finishes, download artifacts:
   - **Actions artifact** (wrapper ZIP) → extract → flash the `*_FLASH_THIS.zip`
   - Optional: enable **Create Release** for direct-download assets.

---

## 🧩 Workflow Options (What you can toggle)

> The workflow UI includes safe, tested defaults. **Balanced** is recommended for daily driving.

| Option | Choices | What it does |
|---|---|---|
| Toolchain | `aosp_prebuilt` / `neutron_clang` | Choose compiler toolchain |
| Profile | `battery` / `balanced` / `performance` | Safe config presets (no “random” overclocks) |
| ZRAM LZ4 perf patch | On/Off *(best-effort)* | Tries to apply the zram LZ4 perf patch; skips if not compatible |
| zram-module.zip | On/Off | Builds a flashable module (replaces `zram.ko` when built as a module) |
| SUSFS | On/Off | Enables SUSFS features |
| SUSFS source | `external` / `sukisu` | **external recommended for r25+ module compatibility** |
| Multi-manager patch | On/Off *(best-effort)* | Attempts patch; skips if not compatible |
| BBG | On/Off | Baseband Guard toggle |
| BBR | On/Off | TCP BBR congestion control toggle |
| ADIOS | On/Off | ADIOS I/O scheduler toggle |
| Re-Kernel | On/Off | Re-Kernel support toggle |
| Proxy/Net tweaks | On/Off *(best-effort)* | Applies network tweaks (skip if incompatible) |
| TTL target/match | On/Off | Enables netfilter TTL target/match |
| IP_SET | On/Off | Enables netfilter ipset |
| HMBIRD/FengChi | On/Off *(best-effort)* | Attempts to apply scheduler patch bundle (skip if incompatible) |
| Suffix / Build ID | string | Custom `LOCALVERSION`, build time/user/host spoofing |

---

## 📦 Outputs

Every run produces:

- **Flashable kernel ZIP**: `OP13_*_SUSFS_FLASH_THIS.zip`
- **SHA256 checksum**: `*.sha256`
- Optional: **zram-module.zip** (if enabled and `zram.ko` exists)

✅ The workflow also writes a **Build Summary table** to:
**Actions → Run → Summary**

---

## 🎛️ Profiles explained

These are **safe presets** intended for stability first:

- **battery**: reduces extra debug overhead and targets efficiency
- **balanced**: closest to “stock-like” behaviour (recommended)
- **performance**: focuses on responsiveness while keeping stability

> No profile should “break” anything — they mainly tune compiler/config knobs and keep the kernel sane.

---

## 🧪 What “best‑effort” means

Some patches vary by kernel tree / commit. For those options:

- **Selected = On** → workflow will **try** to apply the patch  
- If it **does not apply cleanly**, it logs **why** and **continues** the build  
- Your final Build Summary will show **Selected vs Applied/Detected**

This prevents builds from failing just because one optional patch didn’t match your source tree.

---

## 📲 Flashing Guide (Read this first)

### ⚠️ No warranty / use at your own risk
Flashing kernels/modules can cause bootloops, soft-bricks, data loss, or instability.

You are fully responsible for anything that happens to your device.

### ✅ Strongly recommended: backups in recovery (OrangeFox/OFOX)
Before flashing, backup these partitions:

1. **boot**
2. **init_boot**
3. **recovery** *(if your ROM exposes it)*

If you bootloop: restore the backup.

### Flash
1. Use Anykernel3
2. Flash the zip
3. Reboot

---

## 🩺 Troubleshooting

#### Bootloop
1. Flash the stock boot.img
2. Reboot to recovery
3. Restore the Backup

Recommended checklist:
- Build with **SUSFS = On**
- Use **SUSFS source = external** (recommended for r25+ module compatibility)
- If you previously installed another SUSFS module version, **reinstall the correct SUSFS module** after flashing

### zram.ko not found (but build succeeds)
This typically means ZRAM is built **into the kernel** (built-in) rather than as a module.
If you want `zram-module.zip`, ZRAM must be built as a **module** so `zram.ko` exists.

---

## 🙌 Credits

- Thanks: Xiaomichael, cctv18, vc-teahouse, Numbersf, mrcxlinux

---

## 📜 License

Follow upstream kernel / patch project licenses. This repo mainly provides a build pipeline and glue code.
