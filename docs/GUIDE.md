# ThinkPad T15 Gen 2 / T14 Gen 2 Hackintosh Field Guide

## 1. Hardware target

This guide was assembled from a working Lenovo Tiger Lake system with:

- Lenovo ThinkPad T15 Gen 2 / T14 Gen 2 class hardware
- Intel Core i5-1145G7
- Intel Iris Xe graphics, device ID `0x9A49`
- Intel AX201 CNVio Wi-Fi/Bluetooth
- Realtek ALC257 audio
- Synaptics I2C trackpad
- Sonix / Bison integrated UVC webcam, USB ID `5986:910c`
- macOS Sequoia 15.8.2, build `24H232`

The same broad approach may work on similar Tiger Lake Lenovo laptops, but do not assume that kext versions, boot arguments, or USB maps are universal.

## 2. BIOS preparation

Recommended firmware settings:

- Secure Boot: disabled
- Boot mode: UEFI only
- VT-d: enabled if your EFI config supports it; disable only if boot fails
- Fast Boot: disabled
- Thunderbolt/USB pre-boot features: conservative/default settings
- Total Graphics Memory: 512 MB or higher if exposed

After changing BIOS settings, clear NVRAM once from OpenCore.

## 3. macOS installer

A Sequoia installer was used. Keep one known-good USB installer available.

Useful files from the working setup:

- `InstallAssistant.pkg` for macOS Sequoia, identifier `com.apple.pkg.InstallAssistant.macOSSequoia`, version `24.6.830`
- Offline installer helper: `UnPlugged.command`
- Base System installer directory containing `Install macOS Sequoia.app`

Do not delete the installer until the new installation has been rebooted several times and verified.

## 4. OpenCore baseline

Use a recent OpenCore build; the documented system used OpenCore 1.0.8.

### 4.1 Essential OpenCore settings

- `Booter -> Quirks`: configure according to the Dortania OpenCore install guide for Intel.
- `Kernel -> Quirks`: use standard Intel desktop/laptop recommendations.
- `Misc -> Security`: do not enable insecure development settings on a daily driver.
- `PlatformInfo -> Generic -> SystemProductName`: `MacBookPro16,2`

### 4.2 Boot arguments

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d -btleadv
```

Meaning:

| Argument | Purpose |
|---|---|
| `-disablegfxfirmware` | Avoid loading unsupported Apple GPU firmware for Tiger Lake |
| `-btlfxallowanyaddr` | Allows Bluetooth fixup behavior needed by the Intel Bluetooth stack |
| `-allow3d` | Required for the WhateverXe Metal/3D path |
| `-btleadv` | Required by ForbiddenItlwm/BTLEAdvFixup for Bluetooth LE advertising and AirDrop support |

Do not add debug boot arguments unless actively debugging.

### 4.3 SIP

```text
csr-active-config = 03080000   # 0x803
```

This reduced SIP configuration was used for WhateverXe/AMFIAnchor compatibility. Re-evaluate it after major macOS updates.

## 5. Kext stack

Load Lilu first. The following kexts were present and enabled in the working EFI:

| Kext | Version | Purpose |
|---|---:|---|
| Lilu | 1.7.2 | Patch engine required by many kexts |
| WhateverXe | 0.8.4 | Tiger Lake/iGPU graphics patching |
| WhateverXe Graphics Bundles | 0.4.5 | Graphics/Metal bundle set installed under `/Library/GPUBundles` |
| AMFIAnchor | 0.1.2 | AMFI anchor support |
| AMFIAnchorInjector | 1.0.0 | Injector companion for AMFIAnchor |
| AppleALC | 1.9.8 | Audio |
| VirtualSMC | 1.3.8 | SMC emulation |
| SMCBatteryManager | 1.3.8 | Battery |
| SMCProcessor | 1.3.8 | CPU sensors |
| ECEnabler | 1.0.6 | Embedded controller access |
| BrightnessKeys | - | Brightness keys |
| BlueToolFixup | 2.6.9 | Bluetooth stack compatibility |
| BTLEAdvFixup | 0.9.6 | Bluetooth LE advertising fix for AirDrop |
| ForbiddenItlwm | 0.7.0 | Intel AX201 Wi-Fi / AWDL / AirDrop |
| IntelBluetoothFirmware | 2.5.0 | Intel Bluetooth firmware loader |
| USBToolBox | - | USB map support |
| UTBMap | 1.1 | Generated USB port map |
| VoodooPS2Controller | 2.3.7 | Keyboard and PS/2 input |
| VoodooI2C | 2.9.1 | I2C controller |
| VoodooI2CHID | 1.0 | I2C HID support |
| VoodooI2CSynaptics | 1.0 | Synaptics trackpad |
| HoRNDIS | - | USB tethering, optional |

### Disabled/removed components

These were removed or disabled because they conflicted with the working stack:

- `AMFIPass`
- `HeliPort`
- `itlwm`
- `AirportItlwm`
- `AirportItlwm-Sequoia`
- `IO80211FamilyLegacy`
- `IntelBTPatcher`
- `IntelBluetoothInjector`
- IrisXe Manager / NMX GPU bundles
- HighPoint storage kexts

Do not mix old `itlwm`/HeliPort stacks with ForbiddenItlwm.

## 6. Graphics setup

### 6.1 WhateverXe

Use:

- WhateverXe kext `0.8.4`
- WhateverXe Graphics `0.4.5`

Do not downgrade to old 2-byte WEX patches.

Required boot arguments:

```text
-disablegfxfirmware -btlfxallowanyaddr -allow3d
```

### 6.2 GPU bundles

The working system used `/Library/GPUBundles` containing:

- TGL GL bundle
- TGL Metal bundle
- TGL VA bundle
- XeOpenCL bundle

Verify Metal after boot:

```bash
system_profiler SPDisplaysDataType
```

Expected results include:

- Metal 2
- IntelAccelerator attached
- H.264 and HEVC acceleration
- Compute support

## 7. AMFIAnchor setup

Use AMFIAnchor instead of AMFIPass.

Versions:

- `AMFIAnchor.kext` 0.1.2
- `AMFIAnchorInjector.kext` 1.0.0

Verify:

```bash
log show --last boot --style compact \
  --predicate 'eventMessage CONTAINS[c] "AMFIAnchor"' | tail -80
```

Known-good indicators:

```text
InstallStatus=1
AnchorReady=1
LoadedAnchors=b00t0x
ProviderCount=1
RejectedAnchors=0
```

## 8. Wi-Fi, Bluetooth, and AirDrop

### 8.1 ForbiddenItlwm

Use the matching Sequoia release of ForbiddenItlwm. The working version reported:

```text
org.forbiddenitlwm.ForbiddenItlwm (0.7.0)
org.forbiddenitlwm.BTLEAdvFixup (0.9.6)
```

Use:

- `ForbiddenItlwm.kext`
- `BTLEAdvFixup.kext`
- `BlueToolFixup.kext`
- `IntelBluetoothFirmware.kext`
- boot argument `-btleadv`

Remove conflicting WiFi/Bluetooth kexts before booting this stack.

### 8.2 Verification

Check loaded kexts:

```bash
kmutil showloaded | grep -E 'ForbiddenItlwm|BTLEAdvFixup|BlueToolFixup|IntelBluetoothFirmware'
```

Check native interface:

```bash
ifconfig en0
route -n get default
ping -c 3 1.1.1.1
```

Check AWDL:

```bash
ifconfig awdl0
```

Check AirDrop support:

```bash
system_profiler SPNetworkDataType | grep -i -A2 -B2 airdrop
```

Known-good results:

- `en0` has an IP address and internet works
- `awdl0` is up/running
- System Profiler reports `AirDrop: Supported`
- System Profiler reports `Auto Unlock: Supported`
- Bluetooth is On and uses USB transport

Known harmless oddity:

- `networksetup -getairportnetwork en0` may say “not associated”
- System Profiler may show `Status: Network Service Inactive`
- despite `en0`, AWDL, AirDrop, and internet all working

### 8.3 Limitations

ForbiddenItlwm does not fully support every Apple continuity feature:

- Instant Hotspot: unsupported
- Sidecar/Universal Control over Wi-Fi: unsupported
- Continuity Camera: unsupported
- WPA3-only networks: unsupported
- AirDrop requires the Mac to be connected to Wi-Fi

Before macOS updates, boot once without `-btleadv`.

## 9. USB mapping

The working system uses `UTBMap.kext` generated with USBToolBox.

Documented port typing:

- Internal webcam: type `255`
- Internal AX201 Bluetooth: type `255`
- Type-C: type `9`
- Type-A: type `3`

The integrated webcam appears at USB location `0x14400000`.

Why this matters:

- Internal cameras and Bluetooth controllers need correct internal port typing.
- An incorrect USB map can make built-in devices behave like external devices or break power management.

Do not replace a working map with `UTBDefault.kext` unless you are rebuilding from scratch.

## 10. Audio, input, and sensors

Enable:

- AppleALC for Realtek audio
- VoodooPS2Controller for keyboard
- VoodooI2C + VoodooI2CHID + VoodooI2CSynaptics for trackpad
- SMCBatteryManager and SMCProcessor for battery and sensors
- BrightnessKeys for keyboard brightness control
- ECEnabler for EC-backed readings

## 11. Webcam

### 11.1 Detected hardware

The integrated camera is:

```text
Vendor: 0x5986
Product: 0x910c
Name: Integrated Camera
Location: 0x14400000
Speed: USB 2.0 / 480 Mb/s
Built-In: Yes
```

Verify:

```bash
system_profiler SPCameraDataType
system_profiler SPUSBDataType | grep -A12 -B4 'Integrated Camera'
```

### 11.2 UVC brightness/exposure tool

For webcam image tuning, the working system used `uvcctl` from GitHub:

```bash
uvcctl list
uvcctl show
uvcctl save original.json
uvcctl set brightness=160 backlight_compensation=2 power_line_frequency=50hz
uvcctl save tuned.json
uvcctl load original.json
```

Observed controls for the Sonix `5986:910c` camera:

| Control | Default | Range |
|---|---:|---:|
| brightness | 128 | 0..255 |
| contrast | 32 | 0..255 |
| saturation | 64 | 0..100 |
| hue | 0 | -180..180 |
| sharpness | 3 | 0..7 |
| gamma | 120 | 90..150 |
| backlight_compensation | 1 | 0..2 |
| power_line_frequency | 50hz | off/50hz/60hz |
| exposure_auto | on | on/off |
| exposure | auto | 4..1250 |

Practical notes:

- Raising brightness too high can look washed out without improving clarity.
- `sharpness=5` may improve detail slightly.
- Manual exposure can brighten the image but may introduce flicker/noise.
- On 50 Hz mains, keep `power_line_frequency=50hz`.
- UVC settings may reset after reboot, sleep, or camera power loss.

### 11.3 UVCProbeFix note

`UVCProbeFix` was investigated for black-screen/PROBE problems but not installed.

Reason:

- It targets specific UVC camera IDs/fingerprints.
- The documented system runs macOS build `24H232`, while the Sequoia patch fingerprint targets `24A335`.
- The camera ID `5986:910c` is not directly listed in the plugin personalities.

Do not install it blindly on this build.

## 12. Stable verification checklist

After every EFI change, reboot and run:

```bash
sw_vers
uname -a
system_profiler SPDisplaysDataType
kmutil showloaded | grep -E 'WhateverXe|AMFIAnchor|ForbiddenItlwm|BTLEAdvFixup|BlueToolFixup|Lilu'
ifconfig en0
ifconfig awdl0
system_profiler SPCameraDataType
```

Graphics success looks like:

- Metal 2
- IntelAccelerator attached
- H.264/HEVC supported

Wi-Fi success looks like:

- `en0` active
- internet reachable
- `awdl0` up
- AirDrop supported

AMFIAnchor success looks like:

- `InstallStatus=1`
- `AnchorReady=1`
- `LoadedAnchors=b00t0x`
- `RejectedAnchors=0`

## 13. Maintenance rules

- Keep at least two EFI backups: current known-good and previous known-good.
- Do not update macOS and kexts at the same time.
- Before a macOS update, boot once without `-btleadv`.
- After an update, re-check Metal, AMFIAnchor, Wi-Fi, AWDL, AirDrop, and audio.
- Avoid installing old itlwm/HeliPort alongside ForbiddenItlwm.
- Keep a USB installer and `UnPlugged.command` available.

## 14. Suggested build order

1. Prepare BIOS and USB installer.
2. Build minimal OpenCore EFI.
3. Boot installer and install macOS.
4. Add USB mapping.
5. Add graphics stack: Lilu + WhateverXe + GPU bundles.
6. Add AMFIAnchor.
7. Add audio, battery, sensors, keyboard, and trackpad.
8. Add ForbiddenItlwm stack.
9. Tune webcam if needed.
10. Verify and back up the final EFI.

See [CONFIGURATION.md](CONFIGURATION.md) for the exact known-good config and [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for rollback procedures.
