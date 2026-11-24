# ThinkPad T15 Gen 2 / T14 Gen 2 Tiger Lake Hackintosh Field Guide

A practical, field-tested guide for installing and stabilizing macOS on the Lenovo ThinkPad T15 Gen 2 / T14 Gen 2 platform with Intel Tiger Lake, Iris Xe, and Intel AX201.

> **Important:** This guide documents one specific working configuration. Hackintosh is experimental. Hardware, BIOS, EFI revisions, and macOS updates can change the required patches. Always keep a full EFI backup and a known-good USB installer.

## Status of the documented build

| Item | Value |
|---|---|
| macOS | 15.8.2 / 24H232 |
| Darwin | 24.6.0 |
| Architecture | x86_64h |
| CPU | Intel Core i5-1145G7 |
| iGPU | Intel Iris Xe, PCI ID `0x9A49` |
| Wi-Fi / Bluetooth | Intel AX201, Wi-Fi PCI ID `8086:a0f0` |
| SMBIOS | `MacBookPro16,2` |
| Bootloader | OpenCore 1.0.8 |
| Container UID | `7C436110-AB2A-4BBB-A880-FE41995C9F82` |

Current verified boot arguments:

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d -btleadv
```

Current SIP value:

```text
csr-active-config = 03080000    # 0x803
```

## What this guide covers

- BIOS settings
- OpenCore baseline
- Required kext stack
- WhateverXe + WhateverXe Graphics for Iris Xe
- AMFIAnchor instead of AMFIPass
- ForbiddenItlwm + BTLEAdvFixup for native Wi-Fi, AirDrop, and Bluetooth LE advertising
- USB mapping for internal devices
- Audio, trackpad, brightness, battery, and sensors
- Webcam diagnosis and UVC brightness/exposure tuning
- Verification commands
- Rollback and maintenance notes

## Start here

Read:

1. [Installation and stabilization guide](docs/GUIDE.md)
2. [Verified configuration](docs/CONFIGURATION.md)
3. [Troubleshooting and rollback](docs/TROUBLESHOOTING.md)
4. [Credits and upstream sources](docs/CREDITS.md)

## Repository layout

```text
.
├── README.md
└── docs
    ├── GUIDE.md
    ├── CONFIGURATION.md
    ├── TROUBLESHOOTING.md
    └── CREDITS.md
```

No EFI binaries, serial numbers, or personal NVRAM blobs are included in this repository.

## Disclaimer

This project is for educational and interoperability research. Apple macOS is licensed only for use on Apple-labeled hardware. Use at your own risk.
