<div align="center">

# ASUS ROG Zephyrus G14 (GA401IHR) Hackintosh

OpenCore EFI for the 2020 ASUS ROG Zephyrus G14 (GA401IHR) running macOS Sequoia

![macOS](https://img.shields.io/badge/macOS-Sequoia%2015.8.1-blue?logo=apple&logoColor=white)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.4-brightgreen)
![CPU](https://img.shields.io/badge/CPU-Ryzen%207%204800HS-ED1C24?logo=amd&logoColor=white)
![SMBIOS](https://img.shields.io/badge/SMBIOS-MacBookPro16%2C3-lightgrey)

</div>

---

## Contents

- [Overview](#overview)
- [Hardware](#hardware)
- [Software](#software)
- [What works](#what-works)
- [Installation](#installation)
- [BIOS configuration](#bios-configuration)
- [Changes from upstream](#changes-from-upstream)
- [Known issues](#known-issues)
- [Credits](#credits)
- [Disclaimer](#disclaimer)

## Overview

This repository contains an OpenCore EFI that boots macOS **Sequoia 15.8.1** (also tested on 15.1) on the **ASUS ROG Zephyrus G14 2020, model GA401IHR**.

It is based on [PIut02/ROG-Zephyrus-G14-GA401-Hackintosh](https://github.com/PIut02/ROG-Zephyrus-G14-GA401-Hackintosh) (`2020/EFI`, commit `37edc25`). All of the original work belongs to PIut02 and to the authors of OpenCore and the bundled kexts. This repository only adds the [six changes](#changes-from-upstream) that were required to boot Sequoia on a GA401IHR; every other file is byte-for-byte identical to upstream.

## Hardware

| Component | Model |
| :-- | :-- |
| Laptop | ASUS ROG Zephyrus G14 GA401IHR (BIOS 416) |
| CPU | AMD Ryzen 7 4800HS (Renoir, 8C/16T) |
| iGPU | AMD Radeon Vega (Renoir), 2 GB UMA, via NootedRed |
| dGPU | NVIDIA GeForce GTX 1650 (unsupported in macOS) |
| Audio | Realtek ALC285 (`layout-id` 33) |
| Wi-Fi / Bluetooth | Intel Wi-Fi 6E AX210 (upgraded) |
| Storage | Micron 3400 512 GB NVMe |
| Touchpad | ELAN1201 (I2C HID) |

## Software

| Item | Version |
| :-- | :-- |
| macOS | Sequoia 15.8.1 (`24H32`) |
| OpenCore | 1.0.4 |
| SMBIOS | `MacBookPro16,3` |
| NootedRed | 0.8.10 (ChefKissInc, RELEASE) |
| itlwm | 2.3.0 |
| AMD kernel patches | [AMD_Vanilla](https://github.com/AMD-OSX/AMD_Vanilla) `eaf52ef` |

## What works

| Feature | Status | Notes |
| :-- | :--: | :-- |
| Boot (Sequoia 15.1 / 15.8.1) | ✅ | |
| iGPU acceleration | ✅ | NootedRed 0.8.10 |
| Internal display | ✅ | |
| HDMI output | ⚠️ | Not fully verified |
| NVMe storage | ✅ | |
| Wi-Fi | ⚠️ | `itlwm`, appears as Ethernet; not fully verified |
| Bluetooth | ⚠️ | Not fully verified |
| Audio (ALC285) | ⚠️ | Not fully verified |
| Sleep / wake | ⚠️ | Not fully verified |
| USB-C video output | ❌ | Wired to the dGPU |
| NVIDIA GTX 1650 | ❌ | Not supported by macOS |

## Installation

1. Configure the BIOS as described in [BIOS configuration](#bios-configuration).
2. Copy the `EFI` folder to the EFI partition of your USB installer or internal drive.
3. **Generate your own SMBIOS.** Use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) or `macserial` with model `MacBookPro16,3`, then replace `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM` under `PlatformInfo → Generic` in `EFI/OC/config.plist`.
4. **Set your own Wi-Fi network.** Edit `WiFiConfig` in `EFI/OC/Kexts/itlwm.kext/Contents/Info.plist`, or remove the entries and use [HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases).
5. Boot, then select **Reset NVRAM** in the OpenCore picker once.

> [!WARNING]
> The SMBIOS values in `config.plist` belong to the author's machine. Never reuse them: a shared serial number causes iCloud and iMessage problems on both machines.

## BIOS configuration

Hidden options are reached with [Smokeless UMAF](https://github.com/DavidS95/Smokeless_UMAF).

| Setting | Value |
| :-- | :-- |
| `AMD CBS → NBIO → GFX Configuration → iGPU Configuration` | `UMA_SPECIFIED` |
| `AMD CBS → NBIO → GFX Configuration → UMA Frame Buffer Size` | `2G` |
| `PCI Subsystem Settings → Above 4G Decoding` | Enabled (do not combine with `npci=` in boot-args) |
| Secure Boot | Disabled |
| Fast Boot | Disabled |

## Changes from upstream

Differences from PIut02 `2020/EFI`:

| # | Section | Change | Reason |
| :-- | :-- | :-- | :-- |
| 1 | `Kernel → Patch` | Replaced all AMD patches with [AMD_Vanilla](https://github.com/AMD-OSX/AMD_Vanilla) (`eaf52ef`), core count `8`, kept upstream's PAT choice (Shaneee) | Upstream patches have `MaxKernel 23.99.99` (Sonoma only), so Sequoia (Darwin 24) hangs at `EB\|LOG:EXITBS:START` |
| 2 | `ACPI → Add` | Disabled `SSDT-NoHybGfx.aml` (file kept) | Boot hangs at `[ PCI configuration begin ]` |
| 3 | `Kernel → Add` | Disabled `AirportItlwm.kext` (file kept) | `Prelinked injection AirportItlwm.kext - Invalid Parameter`; v2.3.0 has no Sequoia build |
| 4 | `Kernel → Add` | Added `itlwm.kext` 2.3.0 with a preconfigured `WiFiConfig` | Wi-Fi on Sequoia; joins the saved network at boot without HeliPort |
| 5 | `PlatformInfo → Generic` | Filled in SMBIOS | Upstream leaves it empty for users to generate |
| 6 | `Kernel → Add` | Replaced the htmambo NootedRed fork (1.0.0) with official [ChefKissInc NootedRed](https://github.com/ChefKissInc/NootedRed) 0.8.10 | The fork kernel-panics about 8 s into boot on 15.8.1 (`24H32`); 0.8.10 boots with full acceleration. NootedRed stays ordered before AppleALC, as required by NootedRed discussion #422 |

## Known issues

- Wi-Fi via `itlwm` appears in macOS as an Ethernet interface. Use HeliPort to join networks other than the ones in `WiFiConfig`.
- The GTX 1650 stays powered on because `SSDT-NoHybGfx` is disabled, which reduces battery life.
- The USB-C port cannot output video because it is wired to the dGPU. Use HDMI.
- Wi-Fi, Bluetooth, ALC285 audio and sleep/wake have not been fully verified on this machine.

## Credits

- [PIut02](https://github.com/PIut02/ROG-Zephyrus-G14-GA401-Hackintosh): original EFI for the G14 GA401
- [Acidanthera](https://github.com/acidanthera): OpenCore, Lilu, VirtualSMC, AppleALC and more
- [AMD-OSX](https://github.com/AMD-OSX/AMD_Vanilla): AMD kernel patches
- [ChefKissInc](https://github.com/ChefKissInc): NootedRed
- [OpenIntelWireless](https://github.com/OpenIntelWireless): itlwm, HeliPort, IntelBluetoothFirmware
- [Dortania](https://dortania.github.io/): OpenCore Install Guide

## Disclaimer

This EFI is provided as is, without warranty of any kind. It is tuned for one specific machine (GA401IHR, BIOS 416); other G14 variants may need changes. You are responsible for any damage to your hardware or data. macOS is a trademark of Apple Inc.; this project is not affiliated with Apple, ASUS, AMD or NVIDIA.
