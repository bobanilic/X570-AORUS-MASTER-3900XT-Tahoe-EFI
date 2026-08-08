# X570 AORUS MASTER + Ryzen 9 3900XT — macOS Tahoe EFI

Final tested OpenCore EFI for the Gigabyte X570 AORUS MASTER + Ryzen 9 3900XT platform. macOS uses a Radeon RX 580 while an RTX 3080 remains installed for Windows and is disabled only in macOS.

> [!CAUTION]
> This EFI is hardware-specific. The public configuration is deliberately sanitized. Generate your own MacPro7,1 SMBIOS identifiers before booting, test from removable media first, and keep a known-good EFI backup.

## Final tested state

- **macOS:** Tahoe 26.6.1, build `25G76`
- **SMBIOS:** `MacPro7,1`
- **CPU:** Ryzen 9 3900XT — 12 cores / 24 threads confirmed
- **macOS GPU:** Radeon RX 580 8 GB — Ellesmere acceleration and Metal confirmed
- **Windows GPU:** RTX 3080 retained physically and disabled in macOS by PCI device property
- **Wi-Fi / Bluetooth:** Intel AX200
- **Ethernet:** Realtek RTL8125 using `RTL812xLucy.kext`; 1 Gbps link confirmed with a good cable
- **Audio:** Realtek ALC1220 with AppleALC layout 5 plus OCLP Modern Audio root patch
- **USB:** USBToolBox + custom `UTBMap.kext`; H500M front USB-C detects and communicates with iPhone correctly
- **Apple services:** iCloud, iMessage, FaceTime and APNs/IDS registration working

The final private/live `config.plist` used for validation had SHA-256:

```text
1d7ed6b367fb4d4a2075042f288ec8159c11c3c76ee363deddc0f8c70f7e8767
```

The repository copy is sanitized, so its hash is intentionally different.

## Hardware

| Component | Configuration |
|---|---|
| Motherboard | Gigabyte X570 AORUS MASTER |
| CPU | AMD Ryzen 9 3900XT, 12C / 24T |
| macOS GPU | AMD Radeon RX 580 8 GB |
| Windows GPU | NVIDIA GeForce RTX 3080 |
| Wi-Fi / Bluetooth | Intel Wi-Fi 6 AX200 |
| Audio | Realtek ALC1220 |
| Ethernet | Realtek RTL8125 2.5 GbE; Intel I211 also present |
| Storage | WD_BLACK SN850X 1 TB + Samsung 970 EVO Plus 500 GB |
| Case / front I/O | Cooler Master H500M, 4× front USB-A 3.x + front USB-C |
| SMBIOS | MacPro7,1 |

## Feature status

| Feature | Status | Notes |
|---|---|---|
| macOS boot | Working | Stable on Tahoe 26.6.1 / `25G76` |
| CPU topology | Working | 12 physical / 24 logical threads; AVX/AVX2/FMA exposed |
| RX 580 acceleration | Working | 8 GB VRAM, Metal and AMD framebuffer/accelerator stack confirmed |
| RTX 3080 retained | Working | Disabled only in macOS with `disable-gpu` |
| Intel AX200 Wi-Fi | Working | Native macOS Wi-Fi UI via AirportItlwm + OCLP Modern Wireless root patch |
| Bluetooth | Working | IntelBluetoothFirmware + IntelBTPatcher + BlueToolFixup |
| Ethernet | Working | `RTL812xLucy.kext`; 1 Gbps confirmed |
| ALC1220 audio | Working | AppleALC layout 5 + OCLP Modern Audio root patch |
| USB-A | Working | Custom USBToolBox map |
| H500M front USB-C | Working | iPhone enumeration, pairing and libimobiledevice communication confirmed |
| Rear USB-C | Working | Detected during physical mapping |
| iCloud / iMessage / FaceTime | Working | APNs client identity and IDS registration confirmed |
| Continuity Camera over USB | Working | iPhone works as a QuickTime camera over cable |
| Wireless Continuity Camera | Not working | Current Intel Wi-Fi stack does not create `awdl0` |
| AirDrop / AWDL-dependent features | Not working | Same missing-AWDL limitation |
| iPhone Mirroring | Not expected | Apple requires Apple silicon or an Intel Mac with T2; model spoofing cannot provide T2 hardware/attestation |
| iOS 27 extra DeviceSupport package | Pending Apple availability | USB pairing works, but MobileDeviceUpdater currently finds no matching `DEVICESUPPORT` product |

## Important SMBIOS setting

The working configuration uses:

```text
PlatformInfo -> UpdateSMBIOSMode = Create
```

`Custom` caused the underlying Gigabyte firmware identity to appear at runtime on this machine. Both the primary and HeliPort fallback repository configs use `Create`.

Do not change a working machine's SMBIOS/ROM/UUID or reset NVRAM as a generic troubleshooting step.

## Generate your own Apple identifiers

The public repository contains placeholders only. Before booting, replace these values under `PlatformInfo -> Generic` in `EFI/OC/config.plist`:

| Key | Repository value |
|---|---|
| `SystemSerialNumber` | `CHANGEME` |
| `MLB` | `CHANGEME` |
| `SystemUUID` | `00000000-0000-0000-0000-000000000000` |
| `ROM` | Empty |

Use one internally consistent identity set and never reuse identifiers from another public EFI. If you want the HeliPort fallback to be immediately usable, place the same identity set in `config-heliport-working.plist`.

## BIOS / OpenCore notes

Tested setup:

- UEFI boot / CSM disabled
- Secure Boot disabled
- Above 4G Decoding enabled
- Resizable BAR enabled
- `ResizeAppleGpuBars = 0`
- `ResizeGpuBars = -1`
- `SetupVirtualMap = true`
- `DisableIoMapper = true`
- `XhciPortLimit = false`
- `SecureBootModel = Disabled` for the root-patched Tahoe Wi-Fi/audio setup
- `csr-active-config = 03080000`

BIOS updates can change PCI paths. Recheck hardware paths after firmware changes.

## Tahoe Intel Wi-Fi / OCLP root patches

The primary config uses the native-Wi-Fi path with:

- Lilu 1.7.3
- AMFIPass 1.4.1 and `-amfipassbeta`
- IOSkywalkFamily 1.0
- IO80211FamilyLegacy 12.0
- AirportItlwm 2.3.0
- IntelBluetoothFirmware / IntelBTPatcher 2.5.1
- BlueToolFixup 2.7.3
- required IOSkywalkFamily block entry and AX200 device-property injection

After a macOS update, use OCLP only for **Post-Install Root Patch**. The tested 26.6.1 installation applies:

- **Networking: Modern Wireless**
- **Miscellaneous: Modern Audio**

Do **not** use OCLP's **Build and Install OpenCore** function with this custom EFI.

## Ethernet

Primary config:

```text
RTL812xLucy.kext 1.1.1 = enabled
LucyRTL8125Ethernet.kext 1.2.3 = disabled
```

The configured `AppleEthernetRL` block is retained for the Tahoe/legacy-IOSkywalk setup. A previous 100 Mbps negotiation issue was a bad cable, not the driver; a replacement cable negotiates at 1 Gbps full duplex.

## USB map

The final USB map was rebuilt with USBToolBox in Windows so ports absent from the old macOS map could be discovered independently.

The current `UTBMap.kext` contains three controller personalities and includes the H500M front USB-C USB2 path that was verified with an iPhone in macOS, plus mapped rear/front Type-A, SuperSpeed and internal-device paths.

`UTBDefault.kext` and `USBInjectAll.kext` are not used. Keep `XhciPortLimit = false`.

## HeliPort fallback

`EFI/OC/config-heliport-working.plist` is retained as an `itlwm` + HeliPort fallback. Do not enable `AirportItlwm.kext` and `itlwm.kext` simultaneously.

## Kext versions in this snapshot

| Kext | Version / state |
|---|---|
| Lilu | 1.7.3 |
| WhateverGreen | 1.7.1 |
| VirtualSMC | 1.3.8 |
| AppleALC | 1.9.8 |
| AMFIPass | 1.4.1 |
| RestrictEvents | 1.1.7 |
| USBToolBox | 1.2.0 |
| UTBMap | custom 1.1 |
| NVMeFix | 1.1.4 |
| AirportItlwm | 2.3.0 |
| itlwm | 2.3.0; disabled in primary config |
| IntelBluetoothFirmware / IntelBTPatcher | 2.5.1 |
| BlueToolFixup | 2.7.3 |
| RTL812xLucy | 1.1.1; enabled |
| LucyRTL8125Ethernet | 1.2.3; disabled in primary config |

## Privacy

Do not commit real Apple identifiers, network-derived ROM values, iPhone identifiers, APNs certificates/tokens/private keys, or diagnostic dumps that contain them.

## Credits

- Dortania OpenCore Install Guide
- Acidanthera OpenCorePkg and kext projects
- OpCore-Simplify
- OpenIntelWireless itlwm / AirportItlwm
- OpenIntelWireless IntelBluetoothFirmware
- USBToolBox
- OpenCore Legacy Patcher

## Disclaimer

Hackintosh configurations are experimental and hardware-specific. Keep backups and make isolated changes. Do not treat this EFI as universal for other X570 boards, BIOS revisions or PCI layouts.
