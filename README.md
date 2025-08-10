## Razer Blade 15 (2020 Base) — OpenCore Hackintosh EFI

This repository contains my OpenCore EFI for running macOS on the Razer Blade 15 (2020 Base model).

- **Bootloader**: OpenCore (with OpenCanopy)
- **Target macOS**: macOS 12+ recommended (uses BlueToolFixup for Bluetooth). Earlier versions may require kext changes.
- **GPU**: iGPU-only. The NVIDIA/RTX dGPU is disabled via `-wegnoegpu`.

Use this as a reference or a starting point for your own build. Do not use my serials. See the warnings below.

### Confirmed hardware

- **iGPU**: Intel UHD 630 (enabled)
- **dGPU**: NVIDIA RTX 2060 (disabled on macOS)
- **Wi‑Fi/Bluetooth**: Intel AX210
- **Ethernet**: Supported via LucyRTL8125 kext

### What works

- **Boot and install** via OpenCore with GUI picker
- **Intel UHD 630** acceleration (WhateverGreen, iGPU-only)
- **Audio** (AppleALC, `alcid=11`), internal speakers and headphone jack
- **Wi‑Fi** (AirportItlwm) and **Bluetooth** (IntelBluetoothFirmware + BlueToolFixup)
- **Ethernet** (LucyRTL8125)
- **Battery status** (VirtualSMC + SMCBatteryManager)
- **Keyboard/Trackpad**
  - PS2 keyboard/trackpad base support (VoodooPS2)
  - I2C HID trackpad support and gestures (VoodooI2C + VoodooI2CHID)
- **Brightness keys** (BrightnessKeys + SSDT-PNLF)
- **USB mapping** (USBToolBox + UTBMap + USBPower)
- **NVMe** power/compat fixes (NVMeFix)

### What doesn’t work / notes

- **dGPU (RTX 2060)**: Disabled using `-wegnoegpu` (no NVIDIA acceleration on macOS).
- **Hibernation**: Disabled (`HibernateMode=None`). Use sleep only.
- **AX210**: Use the `AirportItlwm.kext` build that matches your macOS version.

### Repository contents

```
OC/
  ACPI/                 # Custom SSDTs
  Kexts/                # Kernel extensions
  Drivers/              # UEFI drivers (AudioDxe, HfsPlus, OpenCanopy, OpenRuntime)
  Resources/            # OpenCanopy resources (icons, audio, fonts)
  Tools/                # OpenShell, etc.
  config.plist          # OpenCore configuration
BOOT/BOOTx64.efi        # Fallback bootloader
```

### ACPI (SSDTs)

Loaded from `OC/ACPI` and enabled in `config.plist`:

- **SSDT-AWAC.aml**: Replaces AWAC with legacy RTC
- **SSDT-EC-USBX-LAPTOP.aml**: Adds embedded controller + USB power properties
- **SSDT-GPRW.aml**: Wake fixes
- **SSDT-I2C.aml**, **SSDT-I2CxConf.aml**: I2C/I2C HID setup for the trackpad
- **SSDT-PLUG-DRTNIA.aml**: CPU power management (PluginType)
- **SSDT-PNLF.aml**: Backlight device for brightness control
- **SSDT-TPD0.aml**: Touchpad routing
- **SSDT-XOSI.aml**: _OSI -> XOSI shim

ACPI patch in `config.plist` renames `_OSI` to `XOSI`.

### Kexts

From `OC/Kexts` and enabled in `config.plist` (load order managed by OpenCore):

- Core: **Lilu**, **VirtualSMC**, **WhateverGreen**
- Power/Platform: **SMCProcessor**, **SMCSuperIO**, **SMCBatteryManager**, **CPUFriend**, **CPUFriendDataProvider**, **RestrictEvents**, **ECEnabler**, **NVMeFix**
- Input: **VoodooPS2Controller** (with plugins), **VoodooI2C**, **VoodooI2CHID**, **VoodooI2CServices**, **VoodooGPIO**, **VoodooInput**
- Audio: **AppleALC** (use `alcid=11`), **BrightnessKeys**
- Networking: **AirportItlwm** (Intel Wi‑Fi), **IntelBluetoothFirmware**, **IntelBTPatcher**, **BlueToolFixup**, **LucyRTL8125Ethernet**
- USB: **USBToolBox**, **UTBMap**, **USBPower**
- Misc: **NoTouchID**, **CtlnaAHCIPort**

### DeviceProperties and boot-args

- iGPU properties set under `PciRoot(0x0)/Pci(0x2,0x0)` for UHD 630 (platform-id, framebuffer patches)
- Audio layout via NVRAM: `alcid=11`
- dGPU disabled: `-wegnoegpu`
- Debug during setup: `debug=0x100 keepsyms=1`

### OpenCore GUI

- Picker: **External** (OpenCanopy) with variant `Acidanthera/GoldenGate`
- Timeout: **5s**, HideAuxiliary: **true**

### SMBIOS

- **Model**: `MacBookPro16,1`

Identifiers in `PlatformInfo -> Generic` are intentionally blank. You must generate and insert your own before going online:

- `SystemSerialNumber`, `MLB`, `SystemUUID`, and `ROM`

Generate with [Dortania’s guide](https://dortania.github.io/OpenCore-Install-Guide/). Tools: OCAT or ProperTree + `GenSMBIOS`.

### BIOS/UEFI settings (recommended)

- Enable AHCI for SATA
- Disable Secure Boot
- Disable Fast Boot
- Enable integrated graphics; set iGPU as primary if available
- Leave virtualization as you prefer; VT-d can remain enabled with `DisableIoMapper=true` in config (already set)

### Installation (quick start)

1. Create a macOS USB installer following the [Dortania install guide](https://dortania.github.io/OpenCore-Install-Guide/).
2. Mount the USB’s EFI partition and replace its `EFI/OC` with this repo’s `OC` folder. Also place `BOOT/BOOTx64.efi` under `EFI/BOOT/`.
3. Open `config.plist` with OCAT or ProperTree:
   - Replace SMBIOS values (`SystemSerialNumber`, `MLB`, `SystemUUID`, `ROM`).
   - Keep `-wegnoegpu` in `boot-args` to disable the RTX dGPU on macOS.
   - If audio is silent, confirm `alcid=11` is present.
   - If Bluetooth/Wi‑Fi misbehaves on your macOS version, use the matching `AirportItlwm` build and verify Intel Bluetooth kexts.
4. Boot the installer via OpenCore, erase target disk as APFS/GUID, and install macOS.
5. After install, copy the working EFI from the USB to the internal disk’s EFI partition.

### Post‑install notes

- USB: A custom map (`UTBMap.kext`) is included. If your ports differ, remap with USBToolBox.
- Trackpad: Uses VoodooI2C + VoodooI2CHID. If gestures fail after updates, clean kext caches or verify I2C SSDTs remain loaded.
- Updates: Update OpenCore and kexts together. Read release notes and follow the [Dortania update guide](https://dortania.github.io/OpenCore-Install-Guide/).

### Known config highlights

- Kernel quirks: `DisableIoMapper=true`, `DisableLinkeditJettison=true`, panic/timeout protections enabled
- UEFI: `ConnectDrivers=true`, HfsPlus, OpenRuntime, OpenCanopy, AudioDxe present
- NVRAM: SIP default config present (`csr-active-config=00000000`), `BlacklistAppleUpdate=true`

### Warnings

- Do not reuse my serials/UUID/MLB/ROM. Generate your own before signing in to Apple services.
- Keep a known-good USB installer handy in case an update breaks boot.
- This EFI is tailored for iGPU-only operation; the RTX dGPU is disabled on macOS.

### Credits

- **Acidanthera** for OpenCore and kexts: Lilu, WhateverGreen, VirtualSMC, AppleALC, NVMeFix, RestrictEvents
- **OpenIntelWireless** for itlwm/AirportItlwm and IntelBluetoothFirmware
- **VoodooI2C** team for I2C trackpad support; **RehabMan/Acidanthera** lineage for PS2
- **USBToolBox** for USB mapping utilities
- **Dortania** for the excellent guides

### License

This repository is for personal/research use. All bundled kexts remain under their respective licenses. Review upstream projects for license terms before redistribution.
