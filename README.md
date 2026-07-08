---
title: "Lenovo ThinkBook 14-IML (Type 20RV) — OpenCore EFI for macOS"
description: "OpenCore EFI for macOS Sequoia on the Lenovo ThinkBook 14-IML (20RV) Hackintosh — kexts, drivers, ACPI patches, and hardware compatibility notes."
permalink: /
---

# Lenovo ThinkBook 14-IML (Type 20RV) — OpenCore EFI for macOS

An [OpenCore](https://github.com/acidanthera/OpenCorePkg) EFI configuration for running macOS on the **Lenovo ThinkBook 14-IML (Type 20RV)**, targeting SMBIOS `MacBookPro16,2`.

> **Status: v1.0.0 — working, with one known hardware limitation (see below).**
> A fully-working `v2.0.0` will follow once NVMe boot support for this laptop's stock Micron SSD is resolved.

## Hardware this EFI was built and tested on

| Component | Detail |
|---|---|
| Model | Lenovo ThinkBook 14-IML, Type **20RV** |
| BIOS | `CJCN42WW` |
| CPU | Intel Core i5-10210U (Comet Lake-U, 4C/8T) |
| RAM | 16GB DDR4 dual-channel (8GB @ 2667MHz + 8GB @ 3200MHz) |
| iGPU | Intel UHD Graphics 620 |
| Display | 1920×1080 @ 60Hz (internal panel) |
| Storage | Micron `MTFDHBA512TCK` 512GB NVMe SSD |
| Ethernet | Realtek RTL8111 (PCIe GbE) |
| Wi-Fi / BT | Realtek 8822CE (802.11ac) |

## Compatibility

| Feature | Status | Notes |
|---|---|---|
| Boot / Installation | ✅ Working | |
| CPU power management | ✅ Working | `SSDT-PLUG.aml`, `CpuTscSync.kext` |
| Graphics (Intel UHD 620) | ✅ Working | `WhateverGreen.kext` |
| Audio | ✅ Working | `AppleALC.kext` |
| Keyboard / Trackpad | ✅ Working | `VoodooPS2Controller.kext`, `VoodooI2C.kext` |
| Battery status | ✅ Working | `SMCBatteryManager.kext` |
| Ethernet (RTL8111) | ✅ Working | `RealtekRTL8111.kext` |
| USB mapping | ⚠️ Partial | `USBToolBox.kext`/`UTBMap.kext` included but currently disabled — needs a port map for this exact chassis |
| **NVMe SSD (Micron MTFDHBA512TCK)** | ❌ **Not working** | `NVMeFix.kext` and `NvmExpressDxe.efi` are present and enabled, but this specific Micron controller is not reliably detected/bootable on this build. This is the primary blocker for a "fully working" release. |
| Wi-Fi / Bluetooth (Realtek 8822CE) | ❌ Not working | No native macOS driver for this chip is included; typically requires a card swap to an Apple-compatible module (e.g. DW1560/BCM94360) for native Wi-Fi/BT/Handoff |
| Light sensor | ⚠️ Disabled | `SMCLightSensor.kext` included but disabled |

## Known limitations (v1.0.0)

- **NVMe SSD support is not functional for the stock Micron `MTFDHBA512TCK`.** This is the reason this is tagged v1.0.0 rather than a "fully working" release. Track/watch this repo for a `v2.0.0` once resolved.
- Wi-Fi/Bluetooth are not supported without swapping the wireless card.
- USB port mapping is not finalized for this chassis.

## ⚠️ Before you boot this on your own machine — replace these values

This EFI's `config.plist` currently contains **this author's own `PlatformInfo` identity values** (`SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`). **Do not use these as-is if you plan to sign in to iMessage, FaceTime, or iCloud** — using a duplicate/shared serial can cause Apple to flag or ban the associated services for everyone using the same values.

### What needs to be replaced

In `EFI/OC/config.plist`, under `PlatformInfo → Generic`:

| Key | What it is |
|---|---|
| `SystemSerialNumber` | The Mac's serial number |
| `MLB` | Logic board serial ("Main Logic Board") |
| `SystemUUID` | Hardware UUID |
| `ROM` | MAC address used to derive your iMessage/FaceTime device identity |

### How to replace them

1. **Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)** (works on macOS; run it from a working Hackintosh, VM, or a real Mac).
2. Run `GenSMBIOS.command` → choose **1. SMBIOS/macserial** → generate a new set for model **`MacBookPro16,2`** (must match the model already set in this config so patches stay valid).
3. It prints fresh values for `Serial Number`, `Board Serial Number (MLB)`, `SmUUID (SystemUUID)`, and a `ROM`/MAC value.
4. Open `EFI/OC/config.plist` with [ProperTree](https://github.com/corpnewt/ProperTree) (do **not** edit OpenCore plists with Xcode or a generic text editor — ProperTree understands the binary/data types OpenCore expects).
5. Navigate to `PlatformInfo → Generic` and paste in your 4 new values, replacing the existing ones.
6. Save, and copy the updated `config.plist` back onto your EFI partition.

Skip this step only if you're just testing in a VM or don't intend to use any Apple services on the install.

## Installation

### Normal install (macOS as the only/primary OS)

1. Download the latest EFI from the [Releases](../../releases) page (or use the "Code → Download ZIP" button above for the full repo).
2. Generate your own serial/MLB/UUID (see above) before first real-world use.
3. Copy the `EFI` folder to the EFI system partition of your boot USB/drive.
4. Boot from the USB via your BIOS boot menu and install macOS per standard OpenCore install guides (see [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)).

### Dual boot with Windows

This EFI is configured to support dual boot out of the box: `ScanPolicy` is set to `0` (unrestricted — scans and lists every bootable OS it finds, including Windows Boot Manager) and the OpenCore picker (`OpenCanopy`) is enabled with `HideAuxiliary: true`, which only hides internal tool entries (Reset NVRAM, UEFI Shell), never a real OS entry. In practice:

- **Windows stays exactly where it is.** OpenCore doesn't touch your existing Windows install; it just adds itself as another boot option and lists Windows Boot Manager alongside macOS in its picker at every boot.
- **Because the internal Micron NVMe SSD isn't supported yet (see [Known limitations](#known-limitations-v100)), install macOS to a separate drive** — typically an external USB3 SSD/enclosure, or an internal 2.5" SATA bay if your chassis has one free. Leave the internal NVMe drive (with Windows) untouched.
- At boot, hold the boot-menu key for your firmware (usually **F12**/**F2** on Lenovo) once, pick the EFI/USB containing this OpenCore build, and you'll get a picker listing both **Windows** and **macOS** — pick per boot, no bcdedit/rEFInd tricks needed.
- Once NVMe support lands in `v2.0.0`, true same-drive dual boot (a second partition on the internal SSD, next to Windows) becomes practical too.

## Kexts / drivers included

| Kext | Version |
|---|---|
| Lilu | 1.7.2 |
| VirtualSMC | 1.3.7 |
| WhateverGreen | 1.7.0 |
| AppleALC | 1.9.7 |
| NVMeFix | 1.1.3 |
| VoodooI2C | 2.9.1 |
| RealtekRTL8111 | 3.0.4 |
| USBToolBox | 1.2.0 |
| SMCBatteryManager | 1.3.7 |

Plus `VoodooPS2Controller`, `CpuTscSync`, `BlueToolFixup`, `BrightnessKeys`, `FeatureUnlock`, `HibernationFixup`, `RestrictEvents`, `SMCProcessor`, `SMCSuperIO`, `SMCLightSensor`.

## Credits

- [Acidanthera](https://github.com/acidanthera) for OpenCorePkg and the kexts above
- [Dortania](https://dortania.github.io/) for OpenCore documentation and guides
- The broader Hackintosh community

## Disclaimer

Provided as-is, for personal and educational use. Bundled kexts and drivers retain their original upstream licenses (see each project's repository). Use at your own risk.
