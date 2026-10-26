# USB-Arsenal Research — android14-6.1 (6.1.162, 2026-03)

GO/NO-GO research for adding USB-Arsenal (ravindu644/USB-Arsenal, `kernel-patches/`) to the android14-6.1 GKI tree at sublevel 162 (2026-03), per matrix row `.github/config/android14-6.1.json` → `{sublevel: "162", date: "2026-03"}`.

Scope: research only, 6.1.162 ONLY. No patches applied, no local build (builds run in GitHub Actions only: `Build Kernels`, `release_type=CI`, `kernel_build_version=6.1.x-android14`, `os_patch_level=162`, one root flavor).

## 1. What USB-Arsenal patches

From `kernel-patches/README.md` (https://github.com/ravindu644/USB-Arsenal/blob/main/kernel-patches/README.md):

**Config options required**
| Option | Needed for |
|---|---|
| `CONFIG_USB_CONFIGFS` | everything (gadget interface) |
| peripheral-capable UDC driver | everything |
| `CONFIG_USB_CONFIGFS_MASS_STORAGE` | `storage` |
| `CONFIG_USB_CONFIGFS_F_HID` | `hid` |
| `CONFIG_USB_CONFIGFS_NCM` | `tether` |
| `CONFIG_USB_CONFIGFS_RNDIS` | `tether --rndis` (gone from GKI after 5.10) |
| `CONFIG_IP_NF_NAT`, `CONFIG_IP_NF_TARGET_MASQUERADE` | `tether` NAT |

**Patches (each `<version>/` folder holds `git format-patch` exports; apply a folder in order via `git am`)**

| Patch | Kernels with a patch folder |
|---|---|
| configfs: allow creating more than one gadget | 4.14, 4.19 |
| configfs: support mainline gadget bring-up alongside Android | 4.14, 4.19 |
| configfs: unregister the gadget's own android device | 4.9, 5.10, 5.15, 6.6 (part of first patch on 4.14/4.19) |
| f_ncm: fix Samsung's changes for a second instance | 4.9, 4.14, 4.19 |
| f_mass_storage: CD-ROM emulation | 4.9, 4.14, 4.19 ("5.10 and up have them" upstream) |
| f_mass_storage: DVD-ROM emulation (>2.36G images) | 4.9, 4.14, 4.19, 5.10, 5.15, 6.6, 6.12 |
| f_hid: free HID state only on /dev/hidgN close | 4.9, 4.14, 4.19, 5.10, 5.15, 6.6 ("5.10 and up have them") |
| f_hid: keep queues/cdev across rebind | 4.9, 4.14, 4.19, 5.10, 5.15, 6.6 (6.12 has them) |
| f_hid: writable bInterval configfs attribute | 4.14, 5.10, 5.15 |

Behavior note: `usb-arsenal check` reads whether a patch is present and never probes; `tether` picks RNDIS when Samsung's NCM extras exist without the f_ncm fix; `hid --interval` warns when the bInterval attribute is missing.

## 2. Kernels USB-Arsenal supports

Patch folders in `kernel-patches/` (GitHub API listing of https://api.github.com/repos/ravindu644/USB-Arsenal/contents/kernel-patches):

**4.9, 4.14, 4.19, 5.10, 5.15, 6.6, 6.12** + README.md

There is **no `6.1` folder** — nothing to `git am` for android14-6.1. Coverage jumps from 5.15 to 6.6.

## 3. Matrix row (source of truth)

`.github/config/android14-6.1.json`:
```json
{ "sublevel": "162", "date": "2026-03" }
```
Exactly one row matches sublevel 162 → target is android14-6.1 @ 6.1.162 (2026-03).

## 4. 6.1.162 base-config state (git-ignored local scratch, kernels/android14-6.1/)

Both root flavors, `kernel/configs/{u,v}/android-6.1/android-base.config`:

| Config | u/android-6.1 | v/android-6.1 |
|---|---|---|
| `CONFIG_USB_CONFIGFS` | y | y |
| `CONFIG_USB_CONFIGFS_MASS_STORAGE` | — | — |
| `CONFIG_USB_CONFIGFS_F_HID` | — | — |
| `CONFIG_USB_CONFIGFS_NCM` | — | — |
| `CONFIG_USB_CONFIGFS_RNDIS` | — | — |
| `CONFIG_USB_CONFIGFS_F_ACC` / `F_AUDIO_SRC` / `F_FS` / `F_MIDI` | y / y / y / y | — / — / y / y |
| `CONFIG_IP_NF_NAT` | y | y |
| `CONFIG_IP_NF_TARGET_MASQUERADE` | y | y |
| `CONFIG_USB_GADGET` | y | y |
| UDC driver (`CONFIG_USB_*_UDC`) | none enabled | none enabled |

Kernel driver sources are not synced in the scratch tree, so UDC availability and 6.1 mainline configfs behavior (multi-gadget, mainline bring-up alongside Android) could not be verified locally.

## 5. Per-area verdicts for 6.1.162

| Area | Requirement | 6.1.162 state | Verdict |
|---|---|---|---|
| configfs core | `CONFIG_USB_CONFIGFS` | y (both flavors) | GO |
| multi-gadget / mainline bring-up | USB-Arsenal patch (4.14/4.19 only); no 6.1 patch | unconfirmed locally; no patch available | BLOCKED |
| storage | `CONFIG_USB_CONFIGFS_MASS_STORAGE` | not set (u, v); mass-storage patches exist only through 6.12, none for 6.1 | NO-GO |
| hid | `CONFIG_USB_CONFIGFS_F_HID` | not set (u, v); bInterval writable-attribute patch only through 5.15, none for 6.1 | NO-GO |
| tether (NCM) | `CONFIG_USB_CONFIGFS_NCM` | not set (u, v); f_ncm Samsung fix only through 4.19 | NO-GO |
| tether (--rndis) | `CONFIG_USB_CONFIGFS_RNDIS` | not set (u, v); README: RNDIS gone from GKI after 5.10 | NO-GO |
| tether (NAT) | `CONFIG_IP_NF_NAT` + `CONFIG_IP_NF_TARGET_MASQUERADE` | y (both flavors) | GO |
| UDC driver | peripheral-capable UDC | none enabled in base config; unverified locally | BLOCKED |
| patch availability | `kernel-patches/6.1/` folder | does not exist | NO-GO |

## 6. Verdict: NO-GO for 6.1.162

USB-Arsenal has **no patch set for kernel 6.1** (folders jump 5.15 → 6.6), so there is nothing to apply to android14-6.1 @ 162. Additionally both 6.1 root flavors lack `CONFIG_USB_CONFIGFS_MASS_STORAGE`, `CONFIG_USB_CONFIGFS_F_HID`, `CONFIG_USB_CONFIGFS_NCM`, and `CONFIG_USB_CONFIGFS_RNDIS` — the configfs functions `storage`, `hid`, and `tether` require — with no USB-Arsenal patches available for 6.1 to add them. Only `tether` NAT prerequisites (`CONFIG_IP_NF_NAT`, `CONFIG_IP_NF_TARGET_MASQUERADE`) and configfs core are already present.

Recommendation: the dependent target task (`Add usb-arsenal Target a14-6.1-162`) should complete as **SKIPPED-NA** (pre-check Step 1: "N/A -> complete as SKIPPED-NA"). Re-evaluate if USB-Arsenal publishes a `kernel-patches/6.1/` set or the 6.1 base config enables the missing configfs functions plus a UDC driver.

## Refs

1. USB-Arsenal kernel-patches README — https://github.com/ravindu644/USB-Arsenal/blob/main/kernel-patches/README.md
2. USB-Arsenal `kernel-patches/` folder listing (no 6.1) — https://api.github.com/repos/ravindu644/USB-Arsenal/contents/kernel-patches
3. Matrix row — `.github/config/android14-6.1.json` → sublevel 162, date 2026-03
4. 6.1 base configs — `kernels/android14-6.1/kernel/configs/{u,v}/android-6.1/android-base.config` (git-ignored local scratch)
