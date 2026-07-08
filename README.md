---
title: "Lenovo ThinkBook 14-IML (Type 20RV) — OpenCore EFI for macOS"
description: "OpenCore EFI for macOS Sequoia on the Lenovo ThinkBook 14-IML (20RV) Hackintosh — kexts, drivers, ACPI patches, and hardware compatibility notes."
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

## ⚠️ Before you boot this on your own machine

This EFI's `config.plist` currently contains **this author's own `PlatformInfo` identity values** (`SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`). **Do not use these as-is if you plan to sign in to iMessage, FaceTime, or iCloud** — using a duplicate/shared serial can cause Apple to flag or ban the associated services for everyone using the same values.

Before relying on this for daily use, generate your own unique set with [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) or `macserial`, and replace the values in `EFI/OC/config.plist` under `PlatformInfo → Generic`.

## Installation

1. Download the latest EFI from the [Releases](../../releases) page (or use the "Code → Download ZIP" button above for the full repo).
2. Generate your own serial/MLB/UUID (see warning above) before first real-world use.
3. Copy the `EFI` folder to the EFI system partition of your boot USB/drive.
4. Boot from the USB via your BIOS boot menu and install macOS per standard OpenCore install guides (see [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)).

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
