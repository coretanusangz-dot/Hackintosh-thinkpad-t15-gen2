# Verified Configuration Snapshot

This is the known-good state documented from the working laptop. It is a reference, not a drop-in EFI.

## System

```text
macOS ProductVersion: 15.8.2
BuildVersion: 24H232
Darwin: 24.6.0
Kernel: xnu-11417.140.69.713.47~2/RELEASE_X86_64
Architecture: x86_64h
```

## OpenCore

```text
Version: 1.0.8
SMBIOS: MacBookPro16,2
Serial: redacted in this repository
```

## NVRAM

GUID:

```text
7C436110-AB2A-4BBB-A880-FE41995C9F82
```

Values:

```text
boot-args = -disablegfxfirmware -btlfxallowanyaddr -allow3d -btleadv
csr-active-config = 03080000
prev-lang:kbd = en-US:0
```

## Enabled EFI kexts

```text
Lilu.kext
AMFIAnchor.kext
AMFIAnchorInjector.kext
ECEnabler.kext
VirtualSMC.kext
WhateverXe.kext
HoRNDIS.kext
SMCBatteryManager.kext
SMCProcessor.kext
AppleALC.kext
BlueToolFixup.kext
BTLEAdvFixup.kext
ForbiddenItlwm.kext
IntelBluetoothFirmware.kext
USBToolBox.kext
UTBMap.kext
VoodooPS2Controller.kext
VoodooPS2Controller.kext/Contents/PlugIns/VoodooInput.kext
VoodooPS2Controller.kext/Contents/PlugIns/VoodooPS2Keyboard.kext
VoodooPS2Controller.kext/Contents/PlugIns/VoodooPS2Mouse.kext
VoodooPS2Controller.kext/Contents/PlugIns/VoodooPS2Trackpad.kext
```

VoodooI2C, VoodooI2CHID, VoodooI2CSynaptics, RestrictEvents, and CPUFriend were present in the EFI but disabled in the final snapshot. If your trackpad requires I2C, enable the matching trio and verify carefully.

## Kext versions

```text
AMFIAnchor.kext                 0.1.2
AMFIAnchorInjector.kext         1.0.0
AppleALC.kext                   1.9.8
BTLEAdvFixup.kext               0.9.6
BlueToolFixup.kext              2.6.9
CPUFriend.kext                  1.3.0
ECEnabler.kext                  1.0.6
ForbiddenItlwm.kext             0.7.0
IntelBluetoothFirmware.kext     2.5.0
Lilu.kext                       1.7.2
RestrictEvents.kext             1.1.6
SMCBatteryManager.kext          1.3.8
SMCProcessor.kext               1.3.8
UTBMap.kext                     1.1
VirtualSMC.kext                 1.3.8
VoodooI2C.kext                  2.9.1
VoodooI2CHID.kext               1.0
VoodooI2CSynaptics.kext         1.0
VoodooPS2Controller.kext        2.3.7
WhateverXe.kext                 0.8.4
```

WhateverXe Graphics bundles installed under `/Library/GPUBundles`: `0.4.5`.

## Graphics

```text
iGPU: Intel Iris Xe
PCI ID: 0x9A49
Metal: Metal 2
Accelerator: IntelAccelerator
H.264: supported
HEVC: supported
Compute: supported
```

Do not use old WhateverXe Graphics 0.3.2 or 0.4.0 packages on this build unless testing a rollback.

## Wi-Fi and Bluetooth

```text
Wi-Fi: Intel AX201
Wi-Fi PCI ID: 8086:a0f0
Native interface: en0
Bluetooth USB ID: 8087:0026
Bluetooth address: F4:A4:75:99:BF:DC
Bluetooth state: On
Bluetooth firmware: v256 c256
AWDL: awdl0 up/running
AirDrop: Supported
Auto Unlock: Supported
```

ForbiddenItlwm constraints:

- Use with `-btleadv` on this macOS build.
- Boot once without `-btleadv` before a macOS update.
- No WPA3-only networks.
- No Instant Hotspot, Continuity Camera, or Wi-Fi Sidecar/Universal Control.

## Webcam

```text
Name: Integrated Camera
USB ID: 5986:910c
Vendor: Sonix Technology Co., Ltd. / Bison Electronics
Location ID: 0x14400000 / 1
Speed: 480 Mb/s
Built-In: Yes
```

The webcam was detected normally. The remaining issue was image tuning, not USB detection.

## USB map summary

```text
Internal webcam: USB type 255
Internal Bluetooth: USB type 255
USB-C: type 9
USB-A: type 3
```

Keep `UTBMap.kext`. Do not replace it with `UTBDefault.kext` unless regenerating the map.

## Removed packages/files from Shared volume

The following were present as archives but are not part of the active setup:

```text
AirportItlwm_v2.3.0_stable_Sonoma14.4.kext.zip
HeliPort.dmg
IrisXe-v1.2.5
WhateverXe-Graphics-0.3.2.pkg
WhateverXe-Graphics-0.4.0.pkg
AppleIntelTGLGraphicsVADriver.bundle.bak
```

Keep them as rollback material if desired; do not install them on top of the working stack.
