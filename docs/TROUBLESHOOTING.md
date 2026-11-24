# Troubleshooting and Rollback

## Golden rules

1. Change one thing at a time.
2. Back up the full EFI before every change.
3. Reboot and verify after every change.
4. Keep the previous known-good EFI somewhere outside the ESP.
5. Do not debug multiple subsystems simultaneously.

## EFI backup

From macOS, with the EFI partition mounted at `/Volumes/NO NAME`:

```bash
sudo mkdir -p /Users/wahab/hackintosh-backup
sudo cp -a '/Volumes/NO NAME/EFI' \
  "/Users/wahab/hackintosh-backup/efi-manual-$(date +%Y%m%d-%H%M%S)"
```

Roll back by copying the chosen backup EFI back to the ESP.

## OpenCore validation

If you have `ocvalidate`, run it after every `config.plist` edit:

```bash
ocvalidate /path/to/config.plist
```

## Graphics failure

Symptoms:

- no Metal
- black screen after boot
- WindowServer crashes
- renderer shows software rendering

Check:

```bash
system_profiler SPDisplaysDataType
log show --last boot --style compact --predicate 'eventMessage CONTAINS[c] "WhateverXe" OR eventMessage CONTAINS[c] "WindowServer"' | tail -200
```

Known-good graphics rules:

- WhateverXe kext `0.8.4`
- WhateverXe Graphics `0.4.5`
- Keep `-disablegfxfirmware -btlfxallowanyaddr -allow3d`
- Do not reinstall IrisXe Manager/NMX bundles
- Do not use old 2-byte WEX patches

## AMFIAnchor failure

Check:

```bash
log show --last boot --style compact --predicate 'eventMessage CONTAINS[c] "AMFIAnchor"' | tail -100
```

Expected:

```text
InstallStatus=1
AnchorReady=1
LoadedAnchors=b00t0x
ProviderCount=1
RejectedAnchors=0
```

If AMFIAnchor fails, do not substitute AMFIPass blindly. Verify the exact versions first.

## Wi-Fi/AirDrop failure

Check kexts:

```bash
kmutil showloaded | grep -E 'ForbiddenItlwm|BTLEAdvFixup|BlueToolFixup|IntelBluetoothFirmware'
```

Check interfaces:

```bash
ifconfig en0
ifconfig awdl0
networksetup -listallhardwareports
```

Rules:

- Remove old `itlwm`, `AirportItlwm`, `HeliPort`, `IntelBTPatcher`, and `IO80211FamilyLegacy`.
- Keep `BlueToolFixup`, `BTLEAdvFixup`, and `ForbiddenItlwm`.
- Keep `-btleadv` on the documented macOS build.
- Before macOS updates, boot once without `-btleadv`.

If `en0` works but AirDrop does not:

- confirm Wi-Fi is associated
- confirm `awdl0` is up
- confirm Bluetooth is On
- re-check that conflicting kexts are absent

## Webcam too dark

The camera is UVC and can be tuned without EFI changes.

```bash
uvcctl list
uvcctl show
uvcctl save original.json
uvcctl set brightness=160 backlight_compensation=2 power_line_frequency=50hz
```

If still too dark, test manual exposure carefully:

```bash
uvcctl set exposure_auto=off exposure=300
```

Restore:

```bash
uvcctl set exposure_auto=on
uvcctl load original.json
```

Do not set brightness too high; `200` was reported as too bright while clarity stayed poor. `sharpness=5` may help slightly.

## USB devices missing

Check:

```bash
system_profiler SPUSBDataType
ioreg -p IOUSB -l -w0 | grep -A12 -B4 'Integrated Camera'
```

Rules:

- Internal camera and internal Bluetooth should be type `255`.
- USB-C should be type `9`.
- USB-A should be type `3`.

If a built-in device shows as external, re-check the USB map.

## Shutdown stall

A shutdown-stall monitor script was prepared but not yet completed as a fix:

```text
/Users/wahab/shutdown-monitor.sh
/Users/wahab/shutdown-monitor.log
/Library/Logs/DiagnosticReports/shutdown_stall_*.shutdownStall
```

The binary `.shutdownStall` format was not successfully decoded. Treat this as an open item, not a solved problem.

## Known unresolved items

- Shutdown stall diagnostics remain unresolved.
- Webcam image sharpness is limited by the small sensor and lens; UVC tuning only helps modestly.
- ForbiddenItlwm does not provide every Apple continuity feature.
