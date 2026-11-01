# Samsung Galaxy M21

This repository is a technical record of work carried out on a Samsung Galaxy M21. It is intended to keep device-specific projects, modifications, experiments, and their outcomes together in one place.

Each substantial project is documented separately so that the repository can grow without being tied to one particular modification. Rooting, firmware work, operating-system experiments, and other device-level projects can each have their own document.

## Device information

| Property | Recorded value |
|---|---|
| Device | Samsung Galaxy M21 |
| Model | `SM-M215F` |
| Device codename | `m21` |
| SoC / platform | Exynos 9611 / `universal9611` |
| CPU architecture | ARM64 |
| Android version | Android 12 |
| Samsung software | One UI Core 4.1 |
| AP / PDA build | `M215FXXS3CWK1` |
| CP / modem build | `M215FDDS3CWL1` |
| CSC build | `M215FODM3CWB1` |
| CSC region | INS (India) |
| Android security patch level | 2023-12-01 |

The software identifiers above describe the firmware baseline recorded during the initial project documented here. They may change if the device is updated or modified later.

## Project records

### 1. Magisk root installation

A boot-image-based root installation carried out from Arch Linux. The matching stock boot image was patched with Magisk 31.1, flashed to the `BOOT` partition using Heimdall, and verified after Android booted successfully.

**Document:** [Magisk root installation](projects/magisk-rooting.md)


### 2. LineageOS 24.0 ROM installation

Installation of an unofficial LineageOS 24.0 (Android 17) build on the Samsung Galaxy M21 (`SM-M215F`, codename `m21`) using Arch Linux. The process included recovery and VBMeta flashing, data formatting, ROM installation through ADB sideload, first-boot verification, partition backups, and investigation of MindTheGapps integration and recovery signature verification.

**Document:** [LineageOS 24.0 ROM installation](projects/lineageos-installation.md)

---

This repository is specific to the `SM-M215F` device. Procedures and results documented here should not be assumed to apply unchanged to other Samsung models or firmware versions.
