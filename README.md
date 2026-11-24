# ThinkPad T15 Gen 2 / T14 Gen 2 Tiger Lake Hackintosh Installation Guide

A complete from-zero installation and stabilization guide for Lenovo ThinkPad T15 Gen 2 / T14 Gen 2 class laptops with Intel Tiger Lake, Iris Xe, and Intel AX201.

This guide documents the exact approach that was applied and verified on the author's system. It is written so that someone else can reproduce the build in a controlled, reversible way.

> **Important:** Hackintosh on non-Apple hardware is experimental. Always keep a full EFI backup, a known-good USB installer, and a rollback path before changing EFI or system files.

---

## 1. Verified target configuration

| Item | Value |
|---|---|
| Laptop class | Lenovo ThinkPad T15 Gen 2 / T14 Gen 2 |
| CPU | Intel Core i5-1145G7 |
| iGPU | Intel Iris Xe, PCI ID `0x9A49` |
| Wi-Fi / Bluetooth | Intel AX201 |
| Wi-Fi PCI ID | `8086:a0f0` |
| Audio | Realtek ALC257 |
| Trackpad | Synaptics I2C |
| Webcam | Sonix / Bison Integrated Camera, USB ID `5986:910c` |
| macOS used here | 15.8.2 |
| Build | `24H232` |
| Darwin | `24.6.0` |
| OpenCore | 1.0.8 |
| SMBIOS | `MacBookPro16,2` |
| SIP | `csr-active-config=03080000` / `0x803` |

Verified boot arguments:

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d -btleadv
```

---

## 2. Important limitations

Before starting, understand what this build does and does not provide.

### Working

- macOS boots and runs on Tiger Lake Iris Xe
- Metal 2 through WhateverXe
- H.264/HEVC acceleration
- Native Intel Wi-Fi through ForbiddenItlwm
- `awdl0` up
- AirDrop supported
- Auto Unlock supported
- Bluetooth with IntelBluetoothFirmware + BlueToolFixup + BTLEAdvFixup
- Battery/sensors through VirtualSMC family
- Audio through AppleALC
- Trackpad through Voodoo I2C stack if enabled/mapped correctly
- Internal webcam detected as a standard UVC camera

### Not fully supported

- Instant Hotspot
- Continuity Camera
- Sidecar / Universal Control over Wi-Fi
- WPA3-only Wi-Fi networks
- Shutdown stall issue is not fully solved
- Webcam image quality is limited by the laptop camera hardware; UVC tuning only helps modestly

---

## 3. Disclaimer

This guide is for educational and interoperability research. Apple macOS is licensed only for use on Apple-labeled hardware. Use this information at your own risk.

---

# PART A — Prepare the machine

## Step 1 — Record your hardware IDs

Before installing anything, boot Linux or Windows and record:

```text
CPU: Intel Core i5-1145G7
iGPU PCI ID: 8086:9A49
Wi-Fi PCI ID: 8086:A0F0
Audio: Realtek ALC257
Webcam USB ID: 5986:910C
Bluetooth USB ID: 8087:0026
```

Useful Linux commands:

```bash
lspci -nn
lsusb
```

On macOS later:

```bash
system_profiler SPUSBDataType
system_profiler SPCameraDataType
system_profiler SPDisplaysDataType
```

The webcam on this machine reports:

```text
Product ID: 0x910c
Vendor ID: 0x5986
Manufacturer: Sonix Technology Co., Ltd.
Location ID: 0x14400000 / 1
Built-In: Yes
```

---

## Step 2 — BIOS settings

Enter Lenovo BIOS setup and use these settings:

```text
Secure Boot: Disabled
Boot Mode: UEFI only
Fast Boot: Disabled
VT-d: Enabled if your EFI supports it; disable only if boot fails
Total Graphics Memory: 512 MB or higher if exposed
USB boot: Enabled
Thunderbolt/USB preboot: default or conservative settings
```

If a setting is not exposed in BIOS, skip it.

After changing BIOS settings, clear NVRAM once from OpenCore later.

---

## Step 3 — Disk layout

The working installation used a separate APFS partition for macOS.

Example layout:

| Partition | Use |
|---|---|
| EFI | Shared FAT32 EFI partition |
| Windows | Existing Windows |
| Windows Recovery | Do not modify |
| Data | Existing shared data |
| MSR | Do not modify |
| Linux root | Existing Linux |
| Shared exFAT | Cross-OS storage |
| macOS APFS | macOS target |

Rules:

- Do not erase the whole disk.
- Do not touch Windows recovery or Microsoft Reserved partitions.
- Install macOS only into the prepared APFS container.
- Keep the EFI partition FAT32 and large enough for OpenCore.

---

# PART B — Download the required files

All files should be downloaded from official GitHub releases or trusted project pages. Avoid random mirrors.

## Step 4 — Download OpenCore

Source:

```text
https://github.com/acidanthera/OpenCorePkg/releases
```

Download the latest stable release, preferably OpenCore 1.0.8 or newer.

Also download from the same release:

```text
Docs/Configuration.tex
Utilities/ocvalidate
```

`ocvalidate` is used to validate `config.plist`.

---

## Step 5 — Download the macOS Sequoia installer

The working setup used a Sequoia `InstallAssistant.pkg`.

Possible sources:

```text
https://support.apple.com/en-us/102655
```

or known Apple installer catalog mirrors if Apple no longer serves the desired build.

The local package was identified as:

```text
identifier: com.apple.pkg.InstallAssistant.macOSSequoia
version: 24.6.830
```

After expanding or installing the package, it provides:

```text
Install macOS Sequoia.app
```

The shared volume also contained:

```text
macdmg3/macOS Base System/Install macOS Sequoia.app
```

with app version `20.6.03`.

---

## Step 6 — Download UnPlugged

Source:

```text
https://github.com/corpnewt/UnPlugged
```

Releases:

```text
https://github.com/corpnewt/UnPlugged/releases
```

`UnPlugged.command` is used to create a macOS USB installer from an `InstallAssistant.pkg` or from an existing `Install macOS Sequoia.app`.

---

## Step 7 — Download required EFI kexts

### Core kexts

| Kext | Source |
|---|---|
| Lilu | https://github.com/acidanthera/Lilu/releases |
| AppleALC | https://github.com/acidanthera/AppleALC/releases |
| VirtualSMC | https://github.com/acidanthera/VirtualSMC/releases |
| BlueToolFixup | https://github.com/acidanthera/BlueToolFixup/releases |
| VoodooPS2Controller | https://github.com/acidanthera/VoodooPS2Controller/releases |
| BrightnessKeys | https://github.com/acidanthera/BrightnessKeys/releases |
| CPUFriend | https://github.com/acidanthera/CPUFriend/releases |
| RestrictEvents | https://github.com/acidanthera/RestrictEvents/releases |
| USBToolBox | https://github.com/USBAAppleBtBcd/USBToolBox/releases |
| ECEnabler | https://github.com/5ilvrtug/ECEnabler/releases |

### Input kexts

| Kext | Source |
|---|---|
| VoodooI2C | https://github.com/alexdated/VoodooI2C/releases |
| VoodooI2CHID | https://github.com/alexdated/VoodooI2C/releases |
| VoodooI2CSynaptics | https://github.com/alexdated/VoodooI2CSynaptics/releases |

### Graphics and AMFI kexts

| Component | Source |
|---|---|
| WhateverXe | https://github.com/b00t0x/WhateverXe/releases |
| WhateverXe Graphics | https://github.com/b00t0x/WhateverXe-Graphics/releases |
| AMFIAnchor | https://github.com/b00t0x/AMFIAnchor/releases |

Use these versions for the documented build:

```text
WhateverXe.kext                 0.8.4
WhateverXe Graphics bundles     0.4.5
AMFIAnchor.kext                 0.1.2
AMFIAnchorInjector.kext         1.0.0
```

### Wi-Fi/Bluetooth kexts

| Component | Source |
|---|---|
| ForbiddenItlwm | https://github.com/retropineaple/ForbiddenItlwm/releases |
| IntelBluetoothFirmware | https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases |
| HoRNDIS, optional | https://github.com/joshuawright11/honrdis/releases |

For this macOS build, the working versions were:

```text
ForbiddenItlwm.kext             0.7.0
BTLEAdvFixup.kext               0.9.6
BlueToolFixup.kext              2.6.9
IntelBluetoothFirmware.kext     2.5.0
```

Do not mix these with old `itlwm`, `AirportItlwm`, HeliPort, `IntelBTPatcher`, or `IO80211FamilyLegacy`.

---

## Step 8 — Download optional webcam tools

For webcam tuning:

```text
https://github.com/tapsterbot/uvcctl/releases
```

The working version was:

```text
uvcctl 0.1.0
```

Alternative GUI researched but not required:

```text
https://github.com/itaybre/CameraController/releases
```

`UVCProbeFix` was researched but not installed:

```text
https://github.com/danielnmaldonado/UVCProbeFix/releases
```

Do not install UVCProbeFix blindly on build `24H232`; its patch fingerprint targets other builds.

---

# PART C — Build the OpenCore USB installer

## Step 9 — Create the USB installer

Use a USB drive of at least 16 GB.

1. Run `UnPlugged.command`.
2. Select the USB drive.
3. Select `InstallAssistant.pkg` or `Install macOS Sequoia.app`.
4. Let the script create the installer.
5. Verify that the USB contains:
   ```text
   EFI/
   com.apple.recovery.boot/
   BaseSystem.dmg
   BaseSystem.chunklist
   ```

Keep the USB installer until the system is stable.

---

## Step 10 — Build a clean OpenCore EFI

On the USB EFI partition, create:

```text
EFI/OC/ACPI
EFI/OC/Drivers
EFI/OC/Kexts
EFI/OC/Resources
EFI/BOOT
```

Copy from the OpenCore release:

```text
OpenCore.efi            -> EFI/OC/OpenCore.efi
BOOTx64.efi             -> EFI/BOOT/BOOTx64.efi
OpenRuntime.efi         -> EFI/OC/Drivers/OpenRuntime.efi
OpenCanopy.efi          -> EFI/OC/Drivers/OpenCanopy.efi
OpenHfsPlus.efi         -> EFI/OC/Drivers/OpenHfsPlus.efi
```

Recommended SSDTs from the original working plan:

```text
SSDT-PLUG.aml
SSDT-PNLF.aml
SSDT-USBX.aml
SSDT-AWAC.aml
```

`SSDT-AWAC.aml` may cause boot issues on some firmware. Remove it if boot stalls.

---

## Step 11 — Copy the kexts

For the first install, keep the EFI minimal:

```text
Lilu.kext
VirtualSMC.kext
SMCBatteryManager.kext
SMCProcessor.kext
AppleALC.kext
USBToolBox.kext
UTBMap.kext or UTBDefault.kext
WhateverXe.kext
AMFIAnchor.kext
AMFIAnchorInjector.kext
```

After macOS is installed, add the networking stack:

```text
BlueToolFixup.kext
BTLEAdvFixup.kext
ForbiddenItlwm.kext
IntelBluetoothFirmware.kext
```

Enable input kexts only if needed for your hardware:

```text
VoodooPS2Controller.kext
VoodooI2C.kext
VoodooI2CHID.kext
VoodooI2CSynaptics.kext
```

In the final verified snapshot, the VoodooI2C trio was present but disabled. Your machine may need them enabled.

---

## Step 12 — Configure `config.plist`

Use `config.plist` from OpenCore's example as a base, then set the following.

### PlatformInfo

```text
SystemProductName = MacBookPro16,2
```

Generate your own values with GenSMBIOS:

```text
https://github.com/corpnewt/GenSMBIOS
```

Do not copy the serial number from this guide or from any random repository.

### NVRAM boot arguments

GUID:

```text
7C436110-AB2A-4BBB-A880-FE41995C9F82
```

For first installation:

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d
```

After adding ForbiddenItlwm/BTLEAdvFixup:

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d -btleadv
```

### SIP

```text
csr-active-config = 03080000
```

This equals `0x803`.

### Kernel Add order

Lilu must load first.

Recommended order:

```text
Lilu.kext
AMFIAnchor.kext
AMFIAnchorInjector.kext
ECEnabler.kext
VirtualSMC.kext
WhateverXe.kext
AppleALC.kext
BlueToolFixup.kext
BTLEAdvFixup.kext
ForbiddenItlwm.kext
IntelBluetoothFirmware.kext
USBToolBox.kext
UTBMap.kext
VoodooPS2Controller.kext
VoodooPS2Controller.kext/Contents/PlugIns/*
```

### Validate

```bash
ocvalidate /path/to/EFI/OC/config.plist
```

Do not boot an invalid config.

---

# PART D — Install macOS

## Step 13 — Boot the installer

1. Plug in the USB installer.
2. Boot from USB or from OpenCore on the internal EFI.
3. At the OpenCore picker, choose:
   ```text
   Install macOS Sequoia
   ```
4. If boot fails, try:
   ```text
   -v
   ```
   temporarily for verbose output.

---

## Step 14 — Erase the target APFS container

Open Disk Utility:

1. Select the prepared macOS APFS partition/container.
2. Erase as:
   ```text
   Name: Macintosh HD
   Format: APFS
   Scheme: GUID Partition Map
   ```
3. Quit Disk Utility.
4. Select `Install macOS Sequoia`.
5. Choose the erased APFS volume.

Do not erase the whole physical disk.

---

## Step 15 — First boot

After installation:

1. The system reboots.
2. At OpenCore, choose the internal macOS installation.
3. Complete Setup Assistant.
4. Do not install extra kexts yet.
5. Do not enable FileVault until the system is stable.

---

# PART E — Post-install stabilization

## Step 16 — Verify basic boot

Run:

```bash
sw_vers
uname -a
system_profiler SPDisplaysDataType
system_profiler SPUSBDataType
```

Expected:

```text
macOS 15.8.2 / 24H232
Darwin 24.6.0
Display shows IntelAccelerator
```

---

## Step 17 — Generate the USB map

1. Boot macOS with a temporary/default USB map.
2. Run USBToolBox on Windows or the chosen mapping workflow.
3. Generate `UTBMap.kext`.
4. Copy it to:
   ```text
   EFI/OC/Kexts/UTBMap.kext
   ```
5. Add it to `Kernel -> Add`.
6. Remove `UTBDefault.kext` if present.

Known-good port types:

```text
Internal webcam: type 255
Internal Bluetooth: type 255
USB-C: type 9
USB-A: type 3
```

Validate again:

```bash
ocvalidate /path/to/config.plist
```

Reboot and verify:

```bash
system_profiler SPUSBDataType | grep -A12 -B4 'Integrated Camera'
```

---

## Step 18 — Install WhateverXe Graphics bundles

WhateverXe kext goes in EFI. The graphics bundles go inside macOS.

1. Download WhateverXe kext `0.8.4`.
2. Copy to:
   ```text
   EFI/OC/Kexts/WhateverXe.kext
   ```
3. Download WhateverXe Graphics `0.4.5`.
4. Install the graphics bundle package or place the bundles under:
   ```text
   /Library/GPUBundles
   ```
5. Reboot.

Verify:

```bash
system_profiler SPDisplaysDataType
```

Expected:

```text
Metal 2
IntelAccelerator
H.264: Yes
HEVC: Yes
```

Do not use old packages:

```text
WhateverXe-Graphics-0.3.2.pkg
WhateverXe-Graphics-0.4.0.pkg
```

They are only historical rollback files.

---

## Step 19 — Install AMFIAnchor

1. Copy `AMFIAnchor.kext` `0.1.2` to EFI.
2. Copy `AMFIAnchorInjector.kext` `1.0.0` to EFI.
3. Add both after Lilu in `Kernel -> Add`.
4. Do not install AMFIPass.
5. Reboot.

Verify:

```bash
log show --last boot --style compact \
  --predicate 'eventMessage CONTAINS[c] "AMFIAnchor"' | tail -80
```

Expected:

```text
InstallStatus=1
AnchorReady=1
LoadedAnchors=b00t0x
ProviderCount=1
RejectedAnchors=0
```

---

## Step 20 — Enable audio, battery, sensors, keyboard, trackpad

Add these kexts if not already present:

```text
AppleALC.kext
SMCBatteryManager.kext
SMCProcessor.kext
ECEnabler.kext
BrightnessKeys.kext
VoodooPS2Controller.kext
VoodooI2C.kext
VoodooI2CHID.kext
VoodooI2CSynaptics.kext
```

Reboot after each small group.

Verify:

```bash
system_profiler SPPowerDataType
system_profiler SPApplePartitionMapDataType
ioreg -p IOUSB -l -w0 | grep -i -A8 -B4 synaptics
```

If the trackpad fails, disable the I2C trio and rely on VoodooPS2 trackpad only, as in the final verified snapshot.

---

## Step 21 — Install ForbiddenItlwm networking stack

Remove or disable these first:

```text
itlwm.kext
AirportItlwm.kext
AirportItlwm-Sequoia.kext
IO80211FamilyLegacy.kext
IntelBTPatcher.kext
IntelBluetoothInjector.kext
HeliPort.app
AMFIPass.kext
```

Then install:

```text
ForbiddenItlwm.kext
BTLEAdvFixup.kext
BlueToolFixup.kext
IntelBluetoothFirmware.kext
```

Add `-btleadv` to boot-args:

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d -btleadv
```

Reboot.

Verify:

```bash
kmutil showloaded | grep -E 'ForbiddenItlwm|BTLEAdvFixup|BlueToolFixup|IntelBluetoothFirmware'
ifconfig en0
ifconfig awdl0
route -n get default
ping -c 3 1.1.1.1
```

Expected:

```text
org.forbiddenitlwm.ForbiddenItlwm (0.7.0)
org.forbiddenitlwm.BTLEAdvFixup (0.9.6)
en0 has IP address
awdl0 is up/running
```

Check AirDrop:

```bash
system_profiler SPNetworkDataType | grep -i -A3 -B3 airdrop
```

Expected:

```text
AirDrop: Supported
Auto Unlock: Supported
```

Known harmless oddity:

```text
networksetup -getairportnetwork en0
```

may say “not associated” while `en0`, AWDL, AirDrop, and internet still work.

---

## Step 22 — Webcam setup and tuning

The camera is standard UVC and should appear without a special kext.

Verify:

```bash
system_profiler SPCameraDataType
system_profiler SPUSBDataType | grep -A12 -B4 'Integrated Camera'
```

Expected:

```text
Integrated Camera
Vendor ID: 0x5986
Product ID: 0x910c
Built-In: Yes
```

Install `uvcctl`:

```bash
curl -fL -o uvcctl-macos.zip \
  https://github.com/tapsterbot/uvcctl/releases/download/v0.1.0/uvcctl-macos.zip
unzip uvcctl-macos.zip
xattr -d com.apple.quarantine ./uvcctl-macos/uvcctl
chmod +x ./uvcctl-macos/uvcctl
./uvcctl-macos/uvcctl list
./uvcctl-macos/uvcctl show
```

Save the original profile:

```bash
./uvcctl-macos/uvcctl save original-5986-910c.json
```

Observed defaults:

```text
brightness              128     0..255
contrast                32      0..255
saturation              64      0..100
hue                     0       -180..180
sharpness               3       0..7
gamma                   120     90..150
backlight_compensation  1       0..2
power_line_frequency    50hz
exposure_auto           on
```

A conservative tuning attempt:

```bash
./uvcctl-macos/uvcctl set brightness=160 backlight_compensation=2 power_line_frequency=50hz
./uvcctl-macos/uvcctl save balanced-5986-910c.json
```

Manual exposure test:

```bash
./uvcctl-macos/uvcctl set exposure_auto=off exposure=300
```

Restore auto exposure:

```bash
./uvcctl-macos/uvcctl set exposure_auto=on
```

Restore original:

```bash
./uvcctl-macos/uvcctl load original-5986-910c.json
```

Notes:

- `brightness=200` was too bright and did not improve clarity.
- `sharpness=5` may help slightly.
- UVC settings reset after reboot/sleep/replug unless reapplied.

---

# PART F — Final verification

## Step 23 — Full verification command set

Run after every EFI change:

```bash
sw_vers
uname -a

system_profiler SPDisplaysDataType
system_profiler SPCameraDataType
system_profiler SPUSBDataType | grep -A12 -B4 'Integrated Camera'

kmutil showloaded | grep -E \
'Lilu|WhateverXe|AMFIAnchor|ForbiddenItlwm|BTLEAdvFixup|BlueToolFixup|IntelBluetoothFirmware|AppleALC|VirtualSMC'

ifconfig en0
ifconfig awdl0
route -n get default
ping -c 3 1.1.1.1
```

Expected summary:

```text
Metal 2
IntelAccelerator
AMFIAnchor: InstallStatus=1, AnchorReady=1, RejectedAnchors=0
Wi-Fi en0 active
awdl0 up
AirDrop supported
Bluetooth on
Camera detected as 5986:910c
```

---

## Step 24 — Back up the final EFI

Mount the EFI partition and copy the whole EFI directory:

```bash
sudo mkdir -p /Users/wahab/hackintosh-backup
sudo cp -a '/Volumes/NO NAME/EFI' \
  "/Users/wahab/hackintosh-backup/efi-good-$(date +%Y%m%d-%H%M%S)"
```

Keep at least two backups:

```text
current known-good
previous known-good
```

---

# PART G — Maintenance rules

## macOS updates

Before updating macOS:

1. Boot once without `-btleadv`.
2. Confirm the system boots normally.
3. Perform the macOS update.
4. After reboot, re-add `-btleadv` if AirDrop/Bluetooth advertising requires it.
5. Re-verify Metal, AMFIAnchor, Wi-Fi, AWDL, AirDrop, audio, and camera.

## Do not mix stacks

Never run these together on the documented setup:

```text
ForbiddenItlwm + itlwm
ForbiddenItlwm + AirportItlwm
BTLEAdvFixup + IntelBTPatcher
AMFIAnchor + AMFIPass
```

## Change control

Follow this rule:

```text
one change -> reboot -> verify -> backup
```

If something breaks, roll back the last EFI backup, not several changes at once.

---

# PART H — Where to find additional details

This README is the full installation path. More detailed reference notes are in:

```text
docs/GUIDE.md
docs/CONFIGURATION.md
docs/TROUBLESHOOTING.md
docs/CREDITS.md
```

Important upstream sources:

```text
OpenCorePkg:        https://github.com/acidanthera/OpenCorePkg
Lilu:               https://github.com/acidanthera/Lilu
WhateverXe:         https://github.com/b00t0x/WhateverXe
WhateverXe Graphics:https://github.com/b00t0x/WhateverXe-Graphics
AMFIAnchor:         https://github.com/b00t0x/AMFIAnchor
ForbiddenItlwm:     https://github.com/retropineaple/ForbiddenItlwm
IntelBluetoothFW:   https://github.com/OpenIntelWireless/IntelBluetoothFirmware
USBToolBox:         https://github.com/USBAAppleBtBcd/USBToolBox
VoodooI2C:          https://github.com/alexdated/VoodooI2C
VoodooPS2Controller:https://github.com/acidanthera/VoodooPS2Controller
AppleALC:           https://github.com/acidanthera/AppleALC
VirtualSMC:         https://github.com/acidanthera/VirtualSMC
uvcctl:             https://github.com/tapsterbot/uvcctl
UnPlugged:          https://github.com/corpnewt/UnPlugged
GenSMBIOS:          https://github.com/corpnewt/GenSMBIOS
Dortania Guide:     https://dortania.github.io/OpenCore-Install-Guide/
```

---

## Final note

No EFI binaries, personal serial numbers, or NVRAM blobs are included in this repository. Generate your own SMBIOS values, keep your own EFI backups, and verify every step on your own hardware.
