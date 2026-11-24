# Credits and Sources

This guide is a synthesis of the working configuration applied to the author's ThinkPad T15/T14 Gen 2 Tiger Lake Hackintosh. It is not original reverse engineering except for the field notes, verification steps, and the combination of settings documented here.

## Core projects

- OpenCore / acidanthera  
  https://github.com/acidanthera/OpenCorePkg

- Lilu / acidanthera  
  https://github.com/acidanthera/Lilu

- AppleALC / acidanthera  
  https://github.com/acidanthera/AppleALC

- VirtualSMC / acidanthera  
  https://github.com/acidanthera/VirtualSMC

- WhateverGreen / acidanthera, referenced for historical EFI archive only  
  https://github.com/acidanthera/WhateverGreen

- BlueToolFixup / acidanthera  
  https://github.com/acidanthera/BlueToolFixup

- VoodooI2C / alexdated  
  https://github.com/alexdated/VoodooI2C

- VoodooPS2Controller / acidanthera  
  https://github.com/acidanthera/VoodooPS2Controller

- USBToolBox / USBAAppleBtBcd  
  https://github.com/USBAAppleBtBcd/USBToolBox

- ECEnabler / 5ilvrtug  
  https://github.com/5ilvrtug/ECEnabler

- BrightnessKeys / acidanthera  
  https://github.com/acidanthera/BrightnessKeys

- CPUFriend / acidanthera  
  https://github.com/acidanthera/CPUFriend

- RestrictEvents / acidanthera  
  https://github.com/acidanthera/RestrictEvents

## Graphics

- WhateverXe / b00t0x  
  https://github.com/b00t0x/WhateverXe

- WhateverXe Graphics / b00t0x  
  https://github.com/b00t0x/WhateverXe-Graphics

- AMFIAnchor / b00t0x  
  https://github.com/b00t0x/AMFIAnchor

- IrisXe Manager / b00t0x, investigated but removed from the final setup  
  https://github.com/b00t0x/IrisXe-Manager

## Networking

- ForbiddenItlwm / retropineaple  
  https://github.com/retropineaple/ForbiddenItlwm

- itlwm and AirportItlwm / OpenIntelWireless, previous-generation stack  
  https://github.com/OpenIntelWireless/itlwm

- AirportItlwm-Sequoia / laobamac, researched but not installed  
  https://github.com/laobamac/itlwm

- OpenIntelWireless documentation  
  https://openintelwireless.github.io/itlwm/FAQ

- HeliPort / OpenIntelWireless, previous stack, removed from final setup  
  https://github.com/OpenIntelWireless/HeliPort

## Webcam / UVC tooling

- uvcctl / tapsterbot  
  https://github.com/tapsterbot/uvcctl

- CameraController / itaybre, researched as an alternative UVC control GUI  
  https://github.com/itaybre/CameraController

- UVCProbeFix / danielnmaldonado, researched but not installed on build `24H232`  
  https://github.com/danielnmaldonado/UVCProbeFix

- ToneSync / elskow, researched as an alternative color-tuning tool  
  https://github.com/elskow/ToneSync

## Installer tooling

- UnPlugged / corpnewt  
  https://github.com/corpnewt/UnPlugged

## Documentation references

- Dortania OpenCore Install Guide  
  https://dortania.github.io/OpenCore-Install-Guide/

- OpenCore Configuration.tex  
  https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.tex

## Hardware-specific community reports

- Lenovo ThinkPad T14 Gen 2 / Tiger Lake reports and Linux hardware records were used only to identify hardware IDs and camera behavior.

- Linux hardware database entry for USB camera `5986:910c`  
  https://linux-hardware.org/?id=usb%3A5986-910c

## Notes on provenance

Some local archive files in `/Volumes/Shared` predate this guide and were used as historical references:

- `hackintosh-session.md`
- `EFI-internal-2026-11-01.tar.gz`
- `EFI-pre-v9-2026-11-01.tar.gz`
- `EFI-backups/`
- `InstallAssistant.pkg`
- `UnPlugged.command`

No Apple installer, kext binary, serial number, or personal NVRAM blob is included in this repository.
