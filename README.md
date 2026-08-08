# X570 AORUS MASTER + Ryzen 9 3900XT — macOS Tahoe OpenCore EFI

A tested OpenCore EFI for the **Gigabyte X570 AORUS MASTER** and **AMD Ryzen 9 3900XT**, validated on **macOS Tahoe 26.6.1 (`25G76`)**.

The system uses an **AMD Radeon RX 580** for macOS while an **NVIDIA GeForce RTX 3080** remains installed for Windows and is disabled only in macOS.

> [!CAUTION]
> **This EFI is hardware-specific.** The public configuration is intentionally sanitized and does not contain usable Apple identifiers. Generate your own `MacPro7,1` SMBIOS values before booting, test from removable media first, and keep a known-good EFI backup.

## Validated configuration

| Component | Configuration |
|---|---|
| Motherboard | Gigabyte X570 AORUS MASTER |
| CPU | AMD Ryzen 9 3900XT — 12 cores / 24 threads |
| macOS GPU | AMD Radeon RX 580 8 GB |
| Windows GPU | NVIDIA GeForce RTX 3080 |
| Wi-Fi / Bluetooth | Intel Wi-Fi 6 AX200 |
| Ethernet | Realtek RTL8125 2.5 GbE; Intel I211 also present |
| Audio | Realtek ALC1220 |
| Storage | WD_BLACK SN850X 1 TB + Samsung 970 EVO Plus 500 GB |
| Case / front I/O | Cooler Master H500M — 4× front USB-A 3.x + front USB-C |
| SMBIOS | `MacPro7,1` |
| macOS | Tahoe 26.6.1 (`25G76`) |

## Feature status

| Feature | Status | Notes |
|---|---|---|
| macOS boot | ✅ Working | Stable on Tahoe 26.6.1 / `25G76` |
| CPU topology | ✅ Working | 12 physical / 24 logical threads confirmed |
| RX 580 acceleration | ✅ Working | Ellesmere graphics stack and Metal acceleration confirmed |
| RTX 3080 retained | ✅ Working | Disabled only in macOS through PCI device properties |
| Intel AX200 Wi-Fi | ✅ Working | Native macOS Wi-Fi UI via AirportItlwm + OCLP Modern Wireless root patch |
| Bluetooth | ✅ Working | IntelBluetoothFirmware + IntelBTPatcher + BlueToolFixup |
| Ethernet | ✅ Working | Realtek RTL8125 via `RTL812xLucy.kext`; 1 Gbps link confirmed |
| ALC1220 audio | ✅ Working | AppleALC layout 5 + OCLP Modern Audio root patch |
| USB-A | ✅ Working | Custom USBToolBox map |
| H500M front USB-C | ✅ Working | iPhone enumeration, pairing and data communication confirmed |
| Rear USB-C | ✅ Working | Verified during physical USB mapping |
| iCloud / iMessage / FaceTime | ✅ Working | APNs client identity and IDS registration confirmed |
| iPhone camera over USB | ✅ Working | Available to QuickTime when connected by cable |
| Wireless Continuity Camera | ❌ Unavailable | Current Intel Wi-Fi stack does not create `awdl0` |
| AirDrop / AWDL-dependent features | ❌ Unavailable | Limited by the same missing-AWDL path |
| iPhone Mirroring | ❌ Unsupported on this build | No physical Apple T2 / Apple silicon security hardware |
| iOS 27 DeviceSupport package | ⚠️ Pending | USB pairing works; Apple currently provides no matching `DEVICESUPPORT` product to this system |

## Before you boot

### 1. Generate your own Apple identifiers

The repository contains placeholders under `PlatformInfo -> Generic`:

| Key | Repository value |
|---|---|
| `SystemSerialNumber` | `CHANGEME` |
| `MLB` | `CHANGEME` |
| `SystemUUID` | `00000000-0000-0000-0000-000000000000` |
| `ROM` | Empty |

Generate one internally consistent identity set and use it only for your machine. Do not copy identifiers from another public EFI.

If you intend to use `config-heliport-working.plist`, place the same identity set there as well.

### 2. Keep `UpdateSMBIOSMode = Create`

The validated configuration uses:

```text
PlatformInfo -> UpdateSMBIOSMode = Create
```

On this board, `Custom` allowed the underlying Gigabyte firmware identity to appear at runtime. Both repository configs therefore use `Create`.

Do not regenerate a working SMBIOS, change ROM/UUID values, or reset NVRAM as a generic troubleshooting step.

### 3. Use the tested firmware / OpenCore settings

| Setting | Value |
|---|---|
| UEFI boot | Enabled |
| CSM | Disabled |
| Secure Boot | Disabled |
| Above 4G Decoding | Enabled |
| Resizable BAR | Enabled |
| `ResizeAppleGpuBars` | `0` |
| `ResizeGpuBars` | `-1` |
| `SetupVirtualMap` | `true` |
| `DisableIoMapper` | `true` |
| `XhciPortLimit` | `false` |
| `SecureBootModel` | `Disabled` |
| `csr-active-config` | `03080000` |

> [!NOTE]
> BIOS updates can change PCI paths. Revalidate GPU, network and USB paths after firmware changes.

## Installation

1. Back up the existing EFI and create a bootable recovery path.
2. Apply the BIOS settings above.
3. Insert your own SMBIOS identifiers into `EFI/OC/config.plist`.
4. Test the EFI from removable media before replacing the system EFI.
5. Boot macOS and verify graphics, networking, audio and USB before applying additional changes.
6. Apply the required OCLP post-install root patches described below.

## Tahoe Wi-Fi and audio root patches

The primary configuration uses the AirportItlwm / legacy wireless path with:

- Lilu 1.7.3
- AMFIPass 1.4.1 with `-amfipassbeta`
- IOSkywalkFamily 1.0
- IO80211FamilyLegacy 12.0
- AirportItlwm 2.3.0
- IntelBluetoothFirmware / IntelBTPatcher 2.5.1
- BlueToolFixup 2.7.3
- the required IOSkywalkFamily block entry
- AX200 device-property injection

After installing or updating macOS, use OpenCore Legacy Patcher only for **Post-Install Root Patch** and apply:

- **Networking: Modern Wireless**
- **Miscellaneous: Modern Audio**

> [!WARNING]
> Do **not** use OCLP's **Build and Install OpenCore** function with this EFI. OCLP is used here only for the required root patches.

## Graphics

The Radeon RX 580 is the active macOS GPU and has verified Ellesmere acceleration and Metal support.

The RTX 3080 remains physically installed for Windows and is disabled only in macOS through PCI device properties. This avoids requiring the card to be removed or disabled in firmware.

## Ethernet

The primary configuration uses:

```text
RTL812xLucy.kext 1.1.1 = enabled
LucyRTL8125Ethernet.kext 1.2.3 = disabled
```

The configured `AppleEthernetRL` block is retained for the Tahoe / legacy-IOSkywalk configuration. The Realtek RTL8125 interface has been validated at **1 Gbps full duplex**.

## USB

The USB map was rebuilt with **USBToolBox in Windows** so ports missing from the previous macOS map could be discovered independently.

The current `UTBMap.kext` contains three controller personalities and includes:

- H500M front USB-C USB2 path verified with an iPhone in macOS
- mapped front and rear USB-A paths
- mapped SuperSpeed paths
- internal-device paths

`UTBDefault.kext` and `USBInjectAll.kext` are not used. Keep `XhciPortLimit = false`.

## HeliPort fallback

`EFI/OC/config-heliport-working.plist` is retained as an **itlwm + HeliPort** fallback configuration.

Do not enable `AirportItlwm.kext` and `itlwm.kext` simultaneously.

## Kext snapshot

| Kext | Version / state |
|---|---|
| Lilu | 1.7.3 |
| WhateverGreen | 1.7.1 |
| VirtualSMC | 1.3.8 |
| AppleALC | 1.9.8 |
| AMFIPass | 1.4.1 |
| RestrictEvents | 1.1.7 |
| USBToolBox | 1.2.0 |
| UTBMap | Custom 1.1 |
| NVMeFix | 1.1.4 |
| AirportItlwm | 2.3.0 |
| itlwm | 2.3.0 — disabled in primary config |
| IntelBluetoothFirmware / IntelBTPatcher | 2.5.1 |
| BlueToolFixup | 2.7.3 |
| RTL812xLucy | 1.1.1 — enabled |
| LucyRTL8125Ethernet | 1.2.3 — disabled in primary config |

## Known limitations

- **AWDL is unavailable** with the current Intel AX200 stack, so wireless Continuity Camera, AirDrop and other AWDL-dependent features are not available.
- **iPhone Mirroring is unsupported on this build** because the PC does not provide the physical Apple T2 / Apple silicon security hardware required by that feature.
- **iOS 27 USB pairing works**, but MobileDeviceUpdater currently finds no matching Apple `DEVICESUPPORT` product for this system.
- OCLP root patches may need to be reapplied after macOS updates.

These limitations are independent of the core boot, graphics, USB, Ethernet, iServices and wired iPhone functionality listed as working above.

## Repository integrity

`SHA256SUMS.txt` contains checksums for the sanitized release files used to verify the repository snapshot.

The public `config.plist` is intentionally different from the private machine-specific configuration because all Apple identity values are removed before publication.

## Privacy and security

Never commit or publish:

- real SMBIOS serial / MLB / SystemUUID / ROM values
- iPhone UDIDs or other device identifiers
- APNs certificates, tokens or private keys
- diagnostic dumps containing account or hardware identity data

## Project principles

This EFI is maintained conservatively:

- make one isolated change at a time
- verify runtime behavior before keeping a change
- preserve a known-good EFI backup
- avoid unnecessary NVRAM resets or SMBIOS regeneration
- treat BIOS updates and macOS updates as changes that may require revalidation

## Credits

This configuration depends on the work of the following projects and communities:

- [Acidanthera / OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- [Dortania OpenCore Install Guide](https://github.com/dortania/OpenCore-Install-Guide)
- [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [OpenIntelWireless / itlwm](https://github.com/OpenIntelWireless/itlwm)
- [OpenIntelWireless / IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [USBToolBox](https://github.com/USBToolBox)
- [OpCore-Simplify](https://github.com/lzhoang2801/OpCore-Simplify)
- [Mieze / RTL812xLucy](https://github.com/Mieze/RTL812xLucy)

## Disclaimer

This repository is provided **as-is** for a specific hardware configuration. Hackintosh behavior can change with BIOS revisions, macOS updates, PCI layout changes and different component revisions.

Do not treat this EFI as a universal configuration for other X570 systems. Test from removable media, keep backups and understand every change before deploying it to a working installation.
