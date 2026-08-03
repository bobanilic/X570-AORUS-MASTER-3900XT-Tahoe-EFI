# X570 AORUS MASTER + Ryzen 9 3900XT — macOS Tahoe EFI

A tested OpenCore EFI for a Gigabyte X570 AORUS MASTER, Ryzen 9 3900XT, Radeon RX 580, NVIDIA RTX 3080 and Intel AX200. The default `config.plist` uses the native macOS Wi-Fi interface through AirportItlwm and the Tahoe Modern Wireless root patch.

> [!CAUTION]
> This is a hardware-specific reference, not a universal EFI. Generate your own SMBIOS values before booting it. Test from a USB drive first and keep a known-good EFI backup.

## Tested hardware

| Component | Configuration |
|---|---|
| Motherboard | Gigabyte X570 AORUS MASTER |
| CPU | AMD Ryzen 9 3900XT, 12 cores / 24 threads |
| macOS GPU | AMD Radeon RX 580 8 GB |
| Windows-only GPU | NVIDIA GeForce RTX 3080, disabled in macOS by PCI path |
| Wi-Fi / Bluetooth | Intel Wi-Fi 6 AX200, PCI ID `8086:2723` |
| Audio | Realtek ALC1220, AppleALC layout 5 |
| Ethernet hardware | Realtek RTL8125 2.5 GbE and Intel I211 |
| Storage | WD_BLACK SN850X 1 TB and Samsung 970 EVO Plus 500 GB |
| SMBIOS | MacPro7,1 |
| macOS | Tahoe 26.x; working snapshot tested on build `25F80` |

The AX200 ACPI path used by this exact board is:

```text
PciRoot(0x0)/Pci(0x1,0x2)/Pci(0x0,0x0)/Pci(0x3,0x0)/Pci(0x0,0x0)
```

Do not assume this path is identical on another board or BIOS configuration. Verify it with `gfxutil`.

## Status

| Feature | Status | Notes |
|---|---|---|
| macOS boot | Working | OpenCore configuration generated and refined with OpCore-Simplify |
| Radeon RX 580 | Working | Primary macOS GPU |
| RTX 3080 retained in the PC | Working | Disabled only in macOS with `disable-gpu` at its PCI path |
| Intel AX200 Wi-Fi | Working | Native macOS Wi-Fi menu via AirportItlwm + Modern Wireless root patch |
| Intel AX200 Bluetooth | Working | IntelBluetoothFirmware, IntelBTPatcher and BlueToolFixup |
| ALC1220 audio | Working | AppleALC layout 5; no VoodooHDA |
| USB | Working | USBToolBox + custom UTBMap; UTBDefault is not used |
| NVMe | Working | Includes NVMeFix |
| AirDrop / AWDL | Not working on the tested Tahoe build | `awdl0` is not created, so peer discovery fails even though Wi-Fi and Bluetooth work |
| Personal Hotspot / Continuity features requiring AWDL | Not expected to work | Same missing-AWDL limitation |

## Important: generate your own Apple identifiers

Both supplied configurations are deliberately sanitized. Before booting, open `EFI/OC/config.plist` and replace these `PlatformInfo -> Generic` placeholders:

| Key | Supplied value | Required action |
|---|---|---|
| `SystemSerialNumber` | `CHANGEME` | Generate a unique MacPro7,1 serial |
| `MLB` | `CHANGEME` | Generate a matching board serial |
| `SystemUUID` | `00000000-0000-0000-0000-000000000000` | Generate a unique UUID |
| `ROM` | Empty | Usually use the MAC address of the built-in primary network interface, without separators |

[GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) can generate the MacPro7,1 values. Never reuse identifiers from another public EFI.

Repeat the same values in `config-heliport-working.plist` if you want that file to be a ready-to-use fallback.

## BIOS and OpenCore notes

This snapshot was built for the working X570 firmware setup, including:

- UEFI boot with CSM disabled
- Secure Boot disabled
- Above 4G Decoding enabled
- Resizable BAR enabled; OpenCore uses `ResizeAppleGpuBars = 0` and `ResizeGpuBars = -1`
- `SetupVirtualMap = true` for this board/firmware combination
- IOMMU may remain enabled because `DisableIoMapper = true`

BIOS updates can change PCI paths or firmware behaviour. Recheck the Wi-Fi and RTX 3080 paths after an update.

## Native Intel Wi-Fi installation on Tahoe

The default configuration already contains and enables:

- Lilu 1.7.3
- AMFIPass 1.4.1 with the `-amfipassbeta` boot argument
- IOSkywalkFamily 1.0
- IO80211FamilyLegacy 12.0
- AirportItlwm 2.3.0 Ventura build
- the required IOSkywalkFamily block entry
- the AX200 Broadcom-compatible device-property injection

It also sets `csr-active-config` to `03080000` and `SecureBootModel` to `Disabled`, as required by this root-patched setup.

After booting this EFI:

1. Install the [OCLP 3.0.0 Nightly amfipassbeta Tahoe build](https://github.com/kgp-macPro/OCLP-lzhoang2801-amfipassbeta/releases).
2. Open **Post-Install Root Patch**.
3. Confirm it detects **Networking: Modern Wireless**.
4. Start root patching and reboot when it finishes.
5. Reset NVRAM once if the native Wi-Fi interface does not appear on the first boot.

A manually installed KDK is not always required for the Modern Wireless patch. If OCLP requests one, install the KDK matching the exact value returned by `sw_vers -buildVersion`; do not substitute a near-matching build.

Root patches normally need to be reapplied after a macOS update. Keep another way to reach the internet available during updates.

## HeliPort fallback

`EFI/OC/config-heliport-working.plist` is the known-good `itlwm` fallback. It keeps native Wi-Fi patch components disabled and does not require the Modern Wireless root patch.

To use it:

1. Preserve the current native `config.plist` as a backup.
2. Copy `config-heliport-working.plist` to `config.plist`.
3. Install [HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases).
4. Reset NVRAM and boot.

Do not enable `AirportItlwm.kext` and `itlwm.kext` simultaneously.

## EFI contents

- ACPI: SSDT-EC, SSDT-USB-Reset and SSDT-USBX
- Audio: AppleALC
- Bluetooth: BlueToolFixup, IntelBluetoothFirmware and IntelBTPatcher
- GPU patching: WhateverGreen
- Storage: CtlnaAHCIPort and NVMeFix
- USB: USBToolBox and UTBMap
- AMD stability: AppleMCEReporterDisabler and the configured AMD kernel patches
- Wi-Fi: native AirportItlwm stack plus disabled `itlwm` fallback

## Credits

- [Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)
- [Acidanthera OpenCorePkg](https://github.com/acidanthera/OpenCorePkg) and its kext projects
- [OpCore-Simplify](https://github.com/lzhoang2801/OpCore-Simplify)
- [OpenIntelWireless itlwm / AirportItlwm](https://github.com/OpenIntelWireless/itlwm)
- [OpenIntelWireless IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [USBToolBox](https://github.com/USBToolBox/kext)
- [OCLP Tahoe amfipassbeta fork](https://github.com/kgp-macPro/OCLP-lzhoang2801-amfipassbeta)

## Disclaimer

Hackintosh configurations are experimental and hardware-specific. This repository is provided as-is. Back up the working EFI and important data before changing firmware, OpenCore, kexts, macOS, root patches or SMBIOS values.
