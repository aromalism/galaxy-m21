# Samsung Galaxy M21 — LineageOS 24.0 Installation

> **Device:** Samsung Galaxy M21 (`SM-M215F`, codename `m21`)  
> **Platform:** Exynos 9611 / `universal9611`, ARM64  
> **Previous software:** Android 12 / Samsung One UI Core 4.1, subsequently rooted with Magisk 31.1  
> **Installed operating system:** Unofficial LineageOS 24.0 / Android 17  
> **Host:** Arch Linux  
> **Installation method:** Lineage Recovery, ADB sideload, and Heimdall  
> **Recorded work period:** 31 October–1 November 2026; the ROM build itself is dated 24 October 2026.

---

## 1. Project overview

This document records the installation of an unofficial LineageOS 24.0 build on a Samsung Galaxy M21 (`SM-M215F`, device codename `m21`). The work replaced the device's previous Samsung Android 12 / One UI Core 4.1 installation with an Android 17-based LineageOS build.

The installation was carried out from an Arch Linux computer. Heimdall was used to flash the supplied Lineage Recovery image and a VBMeta image while the device was in Samsung Download Mode. The ROM was subsequently installed through Lineage Recovery using ADB sideload. The data partition was formatted before the ROM installation.

The ROM installation completed successfully, and the device booted into LineageOS for the first time. The installed build identifies itself as `24.0-20261024-UNOFFICIAL-m21` and reports Android SDK level 37.

An attempt was made to install MindTheGapps 17 ARM64 to provide Google Play Store and Google Play Services. Recovery rejected the package during whole-file signature verification and aborted the installation. The device therefore remained on a working LineageOS installation without GApps. The signature issue and subsequent recovery investigation are documented separately below; no modified recovery image was flashed as part of this project.

This document is a chronological technical record of the work performed. It is not a universal installation guide for every Galaxy M21 variant or every LineageOS build. Exact commands and outputs are included where they were retained. Where the original command transcript or source URL was not preserved, that limitation is stated rather than replaced with an assumption.

## 2. Target device and previous software state

### 2.1 Hardware and platform

| Property | Recorded value |
|---|---|
| Device | Samsung Galaxy M21 |
| Model | `SM-M215F` |
| Device codename | `m21` |
| SoC family | Exynos 9611 |
| Platform | `universal9611` |
| CPU architecture | ARM64 (`arm64-v8a`) |
| CSC region | INS (India) |

### 2.2 Previous Samsung firmware

The device originally ran Android 12 with Samsung One UI Core 4.1. The firmware identifiers recorded during the earlier rooting project were:

| Property | Recorded value |
|---|---|
| Android | Android 12 |
| Samsung software | One UI Core 4.1 |
| AP / PDA build | `M215FXXS3CWK1` |
| CP / baseband | `M215FDDS3CWL1` |
| CSC build | `M215FODM3CWB1` |
| CSC region | INS (India) |
| Android security patch | 2023-12-01 |

The device bootloader had already been unlocked during the earlier Magisk rooting project. Magisk 31.1 root access had also been established on the previous Samsung firmware. That earlier rooting operation is documented separately in the repository's Magisk-rooting record.

### 2.3 Host environment

The ROM installation and subsequent recovery investigation were performed on Arch Linux.

| Tool / component | Recorded version or status | Purpose |
|---|---|---|
| Android Debug Bridge (`adb`) | `37.0.0p1` | Device communication and recovery sideload |
| Fastboot | `37.0.0p1` | Installed on the host; not used for the documented ROM flash sequence |
| Heimdall | `2.2.2` | Flashing Samsung Download Mode partitions |
| `unzip` | Available | ZIP structure and contents inspection |
| OpenSSL | Available | Certificate inspection and fingerprint comparison |
| `mkbootimg` | `/usr/bin/mkbootimg` | Available during later recovery-image inspection; not used to create a flashed image |
| `cpio` / `gzip` | Available | Recovery ramdisk inspection |

The Galaxy M21 uses Samsung Download Mode for low-level partition flashing in this procedure. Heimdall, rather than Fastboot, was used to write the recovery and VBMeta images.

## 3. ROM package and build identification

### 3.1 ROM archive

The installed ROM archive was:

```text
lineage-24.0-m21.zip
```

Working directory:

```text
~/galaxy-m21-rom/
```

Full working path:

```text
~/galaxy-m21-rom/lineage-24.0-m21.zip
```

Recorded archive size and SHA-256 checksum:

| Property | Value |
|---|---|
| File size | Not retained in the available record |
| SHA-256 | `e4df757353dfb584f1abab427cf427c45a909922ee9f7070b21574c505568e8d` |
| Package type | LineageOS block-based OTA ZIP (`ota-type=BLOCK`) |
| Build type | Unofficial |
| Device family | `m21` |

The exact download URL and the original hosting page were not retained in the available installation record. The archive name, its SHA-256 checksum, and the metadata below were recorded locally. The word *unofficial* is part of the build identity; this document does not describe the package as an official LineageOS release.

### 3.2 OTA metadata

The package metadata was inspected after acquisition. The recorded values were:

| Metadata property | Value |
|---|---|
| `ota-type` | `BLOCK` |
| `post-build` | `samsung/m21nsxx/m21:12/SP1A.210812.016/M215FXXU3CWB1:user/release-keys` |
| `post-sdk-level` | `37` |
| `post-security-patch-level` | `2026-10-01` |
| `pre-device` | `m21,m21dd,m21nsxx` |
| LineageOS version property | `24.0-20261024-UNOFFICIAL-m21` |
| ROM build date | 24 October 2026 |

The `post-build` fingerprint retains a Samsung device fingerprint string. It should not be interpreted as evidence that the installed operating system is still Samsung One UI. The installed system identifies itself as an unofficial LineageOS 24.0 build.

### 3.3 Package integrity

The ROM archive's SHA-256 checksum was recorded as:

```text
e4df757353dfb584f1abab427cf427c45a909922ee9f7070b21574c505568e8d
```

The checksum is useful for identifying the exact local archive used during this installation. It does not, by itself, establish the trustworthiness of the original download source.

## 4. Recovery image and partition preparation

### 4.1 Recovery image

The recovery image used for the installation was:

```text
lineage-recovery.img
```

Working path:

```text
~/galaxy-m21-rom/lineage-recovery.img
```

Recorded image properties:

| Property | Value |
|---|---|
| File size | `71,106,560` bytes |
| Equivalent size | 68 MiB |
| SHA-256 | `8c371c177fe55dfb4773e4c0e04a403701cd14760c6b282e96ec769a587ccd8a` |
| Target partition | `RECOVERY` |
| Recorded RECOVERY partition ID | `13` |
| Recorded RECOVERY partition capacity | `71,106,560` bytes |

The image size exactly matched the recorded RECOVERY partition capacity. The image was not reconstructed or modified before it was flashed during the installation.

### 4.2 VBMeta image

A file named `vbmeta.img` was supplied to Heimdall during the recovery installation operation. The exact acquisition source and checksum for this file are not present in the retained record. Its use is recorded because it appeared in the executed Heimdall command; its provenance should not be inferred from the ROM archive metadata.

### 4.3 Backup locations

Before and during the ROM work, partition backups and recovery copies were retained in the following locations:

```text
~/galaxy-m21-rom/partition-backups/
~/galaxy-m21-backup/
```

The recorded backup files included:

```text
~/galaxy-m21-rom/partition-backups/boot.img
~/galaxy-m21-rom/partition-backups/recovery.img
~/galaxy-m21-rom/partition-backups/dtbo.img
~/galaxy-m21-rom/partition-backups/vbmeta.img
~/galaxy-m21-rom/partition-backups/vbmeta_samsung.img
~/galaxy-m21-rom/partition-backups/efs.img
~/galaxy-m21-rom/partition-backups/sec_efs.img
~/galaxy-m21-rom/partition-backups/radio.img
```

Additional copies were retained at:

```text
~/galaxy-m21-backup/boot.img
~/galaxy-m21-backup/recovery.img
```

The `boot.img` in the earlier Magisk backup directory was the previously patched boot image from the Samsung firmware environment. It must not be confused with the boot image installed by the LineageOS ROM. The backup directory also contained a recovery backup. Backup existence is documented here; this document does not claim that every backup was independently restored or tested during this ROM installation.

### 4.4 Relevant partition map

The device PIT and partition layout had been inspected during the earlier device-modification work. The following partition IDs were recorded and relevant to the ROM installation and recovery investigation:

| Partition | ID | Role in this project |
|---|---:|---|
| `BOOT` | 12 | ROM updater writes its `boot.img` here |
| `RECOVERY` | 13 | Lineage Recovery image was flashed here |
| `DTBO` | 10 | ROM installation updated DTBO |
| `VBMETA` | 22 | VBMeta image was flashed during recovery preparation; ROM updater also updated VBMeta |
| `VBMETA_SAMSUNG` | 23 | Backed up; not flashed in the documented ROM sequence |
| `SUPER` | 25 | Dynamic-partition container used by the ROM installation |
| `USERDATA` | 32 | Formatted during the clean installation |

The PIT was not flashed or repartitioned. No change to the partition table was part of this project.

## 5. Flashing Lineage Recovery and VBMeta

### 5.1 Flashing operation

The phone was placed in Samsung Download Mode and connected to the Arch Linux host over USB. Heimdall was used to flash the VBMeta and recovery images in one operation.

The recorded command was:

```bash
sudo heimdall flash --VBMETA vbmeta.img --RECOVERY lineage-recovery.img --no-reboot
```

The command selected two named partitions:

- `VBMETA` received `vbmeta.img`.
- `RECOVERY` received `lineage-recovery.img`.

The `--no-reboot` option prevented Heimdall from automatically rebooting the device at the end of the operation. This allowed the subsequent transition into recovery to be controlled manually.

The exact Heimdall progress transcript from this operation was not retained. The recovery image was subsequently available on the device and used to carry out the factory reset and ADB sideload installation.

### 5.2 Scope of the flash

The documented Heimdall command targeted `VBMETA` and `RECOVERY` only. It did not include `BOOT`, `SUPER`, `USERDATA`, `VBMETA_SAMSUNG`, EFS, or the PIT. The LineageOS ZIP installer performed its own partition updates later in recovery.

## 6. Data formatting and clean-install preparation

### 6.1 Entering Lineage Recovery

After the recovery image had been flashed, the device was booted into Lineage Recovery. Recovery was used for the data-format operation and for installing the ROM package through ADB sideload.

The precise physical-button timing used for the recovery boot was not retained in the command record. It is therefore not reproduced as an exact button sequence here.

### 6.2 Formatting data

The following recovery interface path was used:

```text
Factory reset → Format data
```

The data partition was formatted before the LineageOS ZIP was installed. This was a destructive operation and erased the existing user data. It was part of the clean installation, not a routine reboot or cache-only wipe.

Formatting data also ensured that the new system was not being started with the old Samsung userdata state. No claim is made that personal user data was preserved by the format operation.

### 6.3 Installation state

After formatting data, the device remained in recovery and was prepared for ADB sideload. The ROM package was available on the Arch Linux host in `~/galaxy-m21-rom/`.

## 7. Installing LineageOS through ADB sideload

### 7.1 Starting the sideload interface

Lineage Recovery's **Apply update** / **Apply from ADB** sideload function was used to prepare the phone to receive the ROM ZIP from the host. The exact menu wording can vary by recovery revision; the recorded operation was an ADB sideload installation.

### 7.2 Host-side command

From the Arch Linux host, the ROM package was sent to recovery with:

```bash
adb sideload lineage-24.0-m21.zip
```

The command was run from the working directory containing the ROM archive:

```text
~/galaxy-m21-rom/
```

The command transfers the package to the recovery sideload service. It does not install an Android application; the recovery updater processes the OTA package and writes the included system images and related partitions.

### 7.3 Installation result

The ROM installer completed with:

```text
Install completed with status 0.
```

The installation output showed that the package updated the Android dynamic-partition contents and related boot metadata. The recorded update targets included:

- `system`
- `system_ext`
- `product`
- `vendor`
- `odm`
- `boot`
- `dtbo`
- `vbmeta`

The dynamic partitions are hosted inside the `SUPER` partition. The ROM package's block-based OTA installer performed the relevant writes; no manual `dd` operation or manual partition resize was part of the documented installation.

### 7.4 Boot image replacement and Magisk state

The ROM installer wrote its own `boot.img` to the `BOOT` partition. The previous Magisk-patched Samsung boot image should therefore not be assumed to remain active after this ROM installation.

Magisk root access was not independently verified after LineageOS was installed. The successful root verification in the separate rooting project applies to the previous Samsung firmware environment, not automatically to the LineageOS installation.

## 8. First boot and installed operating system

### 8.1 First boot

After the ROM installation completed with status 0, the device was rebooted from recovery into the installed system. The phone reached the LineageOS setup environment and successfully completed its first boot.

This was the primary success criterion for the ROM installation: the device booted into the new operating system instead of remaining in recovery or failing during startup.

### 8.2 Installed build properties

The installed system reported the following properties:

| Property | Recorded value |
|---|---|
| Operating system | LineageOS |
| LineageOS version | `24.0-20261024-UNOFFICIAL-m21` |
| Android version | Android 17 |
| SDK level | `37` |
| Device | `m21` |
| Architecture | `arm64-v8a` |
| Security patch level in package metadata | `2026-10-01` |
| Build type | Unofficial |

The system version string was also recorded as:

```text
ro.lineage.version=24.0-20261024-UNOFFICIAL-m21
```

The Android system reported:

```text
ro.build.version.release=17
ro.build.version.sdk=37
```

The package build date was 24 October 2026. This is distinct from the date on which the ROM was installed.

### 8.3 Scope of post-install verification

The device booted successfully into LineageOS. The installation record confirms the ROM version and Android SDK level. It does not include a complete post-install hardware validation report for every component, such as camera, mobile data, VoLTE, NFC, sensors, Bluetooth, GPS, or all power-management behaviour. Those features should not be marked as tested solely because the system completed its first boot.

## 9. MindTheGapps installation attempt

### 9.1 Objective

After LineageOS had booted successfully, an attempt was made to add Google applications and services using MindTheGapps for Android 17 and ARM64. The intended result was the genuine Google Play ecosystem, including Google Play Store, Google Play Services, Google account sign-in, push notifications, and app compatibility.

The package selected for the attempt was:

```text
mindthegapps-17-arm64.zip
```

Working path:

```text
~/galaxy-m21-rom/mindthegapps-17-arm64.zip
```

Recorded package properties:

| Property | Value |
|---|---|
| File size | `544,284,463` bytes |
| Approximate size | 519.3 MiB |
| Target Android | Android 17 / SDK 37 |
| Architecture | ARM64 |
| SHA-256 | `a6844eeb2613541558237e467b20feb01f8b45b2ebab0109834140ea59b6ca4d` |

The exact download URL and hosting source were not retained in the available record. The package's internal metadata and certificate were inspected, but the whole-file cryptographic signature was not independently verified in the work recorded here.

### 9.2 ZIP structure check

The archive was tested using:

```bash
unzip -t mindthegapps-17-arm64.zip
```

The ZIP test completed successfully. This established that the archive's stored entries could be read and that the ZIP structure passed the tool's integrity test. It did **not** independently establish the authenticity of the download or cryptographically validate Android's whole-file OTA signature.

### 9.3 Sideload attempt

MindTheGapps was sent to Lineage Recovery through ADB sideload. Recovery rejected the package during whole-file signature verification and aborted the installation.

The relevant recovery output was:

```text
ERROR: recovery: failed to verify whole-file signature
ERROR: recovery: Signature verification failed
ERROR: recovery: error: 21
Install completed with status 2.
Installation aborted.
```

Unlike the standard add-on installation flow described in general LineageOS instructions, this recovery build did not display an option to continue past the signature failure. The installation was therefore not completed.

### 9.4 Result

MindTheGapps was **not installed**. The phone continued to use the successfully booted LineageOS system without Google Play Store or Google Play Services.

No manually installed Play Store APK, Google Services Framework APK, or Google Play Services APK was used as a substitute. No partial GApps installation was left intentionally in the system.

## 10. Recovery signature investigation

The failed MindTheGapps sideload was investigated to determine why recovery rejected the package and why no override prompt appeared.

### 10.1 Recovery image identity

The recovery image used during the installation was retained at:

```text
~/galaxy-m21-rom/lineage-recovery.img
```

The original image was copied for inspection. The recorded SHA-256 was:

```text
8c371c177fe55dfb4773e4c0e04a403701cd14760c6b282e96ec769a587ccd8a
```

The image size was:

```text
71106560 bytes
```

The recovery was identified as a version-2 Android boot image. Its header contained a non-zero recovery DTBO section. This is important because any future image reconstruction must preserve the recovery DTBO payload and the other original image components. Repacking only the kernel and ramdisk would not be a faithful reconstruction.

The recorded recovery DTBO size from the image header was:

```text
2,228,650 bytes
```

No modified or reconstructed recovery image was flashed during this project.

### 10.2 Recovery build properties

The extracted recovery ramdisk contained these recorded properties:

| Property | Value |
|---|---|
| `ro.build.type` | `user` |
| `ro.secure` | `1` |
| `ro.debuggable` | `0` |
| `ro.build.version.release` | `17` |
| `ro.build.version.sdk` | `37` |
| `ro.lineage.device` | `m21` |
| `ro.lineage.version` | `24.0-20261024-UNOFFICIAL-m21` |
| `ro.lineage.releasetype` | `UNOFFICIAL` |
| Recovery build date | `Sat Oct 24 11:23:20 UTC 2026` |

The recovery binary's available strings and behaviour were consistent with whole-file signature verification against the recovery trust store. On this `user` build, signature failure caused the installation to abort instead of presenting an install-anyway prompt.

### 10.3 Trusted recovery certificates

The extracted recovery ramdisk contained:

```text
/system/etc/security/otacerts.zip
```

The ZIP contained two certificate files:

```text
lineage.x509.pem
releasekey.x509.pem
```

The recorded SHA-256 certificate fingerprints were:

| Certificate | SHA-256 fingerprint |
|---|---|
| LineageOS certificate (`lineage.x509.pem`) | `51:83:25:EF:7F:96:C0:D1:19:4C:2E:85:6B:04:0D:63:61:66:FF:B8:46:71:7D:72:FA:87:F4:FA:E5:BE:7B:BB` |
| Recovery release certificate (`releasekey.x509.pem`) | `E9:FB:BE:29:17:F5:B5:CE:C2:4C:CE:C4:4A:F9:AA:FB:8A:57:59:21:63:11:00:2A:D9:03:C5:88:2F:0B:EA:5E` |

The MindTheGapps package contained an embedded certificate at:

```text
META-INF/com/android/otacert
```

The certificate extracted from the package's CMS signature data and the embedded `otacert` had the same recorded SHA-256 fingerprint:

```text
91:37:0E:89:7E:35:BD:90:A2:62:B7:3E:82:96:66:18:19:C1:3D:54:67:FD:30:A1:0D:F8:48:DE:14:F5:67:76
```

That fingerprint did not match either of the two certificates in the recovery's `otacerts.zip`. This explains the trust-store mismatch observed during the investigation. Certificate extraction and fingerprint comparison establish the identity of the certificate presented by the package; they do **not**, by themselves, constitute an independent cryptographic verification of the complete ZIP signature or establish the package's download provenance.

### 10.4 ZIP signature container inspection

The MindTheGapps ZIP comment contained a SignApk whole-file signature structure. Its recorded comment length was 1,602 bytes and began with the text:

```text
signed by SignApk
```

The signature data included a CMS/PKCS#7 object followed by a six-byte footer. The CMS DER object was extracted separately for certificate inspection. OpenSSL parsed the CMS structure after the trailing footer was excluded. The embedded signer certificate was then compared with `META-INF/com/android/otacert`.

This work was limited to parsing the signature container and comparing certificates. A complete SignApk-compatible whole-file signature verification over the ZIP contents was not completed. Therefore, this record does not claim that the MindTheGapps package's cryptographic signature was fully validated.

### 10.5 Finding

The recovery's trusted OTA certificate archive did not contain the certificate presented by the MindTheGapps package. The recovery was built as a `user` build and aborted on the resulting signature-verification failure without offering an override. This combination explains why the expected continuation prompt was unavailable in the observed installation attempt.

The investigation did not modify the recovery trust store, rebuild the recovery image, or flash an alternative recovery. Certificate injection remained a possible future engineering investigation, not an implemented fix.

## 11. Other kernel and recovery tooling investigated

### 11.1 Everline kernel package

An alternative kernel package was inspected during the project:

```text
everline-aosp-m21.zip
```

Recorded SHA-256:

```text
306fec77bab8356ce7e8466aad8fed03c981dc28ff672f09b55ee63359329b12
```

The package used an AnyKernel3-style installation script and advertised support for the M21 across Android 13–17. The decision was made to retain the kernel bundled with the installed LineageOS ROM. The Everline package was **not flashed** and is not part of the resulting system state.

### 11.2 Recovery-image tooling investigation

The later recovery inspection required examining the Android boot-image header and ramdisk. The following tools were available on the Arch Linux host:

```text
mkbootimg  /usr/bin/mkbootimg
cpio
 gzip
```

A `magiskboot` executable was also present in the inspection directory, but it was an ARM 32-bit statically linked binary and could not execute natively on the x86-64 host. The observed failure was:

```text
zsh: exec format error: ./magiskboot
```

`mkbootimg` was available as a packer but did not provide the required unpacking operation. No attempt was made to rebuild and flash a modified recovery using an incomplete extraction or repacking procedure.

This tooling investigation is included because it influenced the decision not to modify the recovery image during the ROM installation project.

## 12. Data, partition, and scope impact

### 12.1 Data impact

The recovery interface's **Format data** operation erased the existing userdata contents. The clean installation should be treated as destructive. No claim is made that application data, accounts, settings, or locally stored files from the previous Samsung installation were preserved.

### 12.2 Partitions changed

The documented operations affected the following targets:

| Partition / area | Operation | Result |
|---|---|---|
| `VBMETA` | Heimdall flash of `vbmeta.img`; later updated by ROM installer | Updated |
| `RECOVERY` | Heimdall flash of `lineage-recovery.img` | Lineage Recovery installed |
| `USERDATA` | Recovery **Format data** operation | Formatted |
| Dynamic partitions in `SUPER` | ROM block-based OTA installation | LineageOS system components installed |
| `BOOT` | ROM updater wrote the ROM's `boot.img` | ROM boot image installed |
| `DTBO` | ROM installer update | Updated |

The device PIT was not modified. The record does not show a manual flash of `VBMETA_SAMSUNG`, EFS, modem/radio, or bootloader partitions during the LineageOS installation.

### 12.3 Out-of-scope operations

The following were not completed as part of this project:

- Installation of MindTheGapps or any other GApps package.
- Modification of `otacerts.zip` or recovery trust configuration.
- Rebuilding or flashing a modified recovery image.
- Flashing the Everline kernel.
- Repartitioning or flashing the PIT.
- Manual changes to EFS, Secure EFS, modem, radio, or bootloader partitions.
- Post-installation verification of every hardware feature.
- Confirmation of Magisk root access on the installed LineageOS build.

## 13. Problems encountered and outcomes

| Issue | Observation | Outcome |
|---|---|---|
| GApps installation rejected | Recovery failed whole-file signature verification and returned error 21 | MindTheGapps installation aborted; no GApps were installed |
| No signature override prompt | Recovery was a `user` build and aborted automatically | The package could not be accepted through the recovery interface used |
| GApps certificate absent from recovery trust store | Package certificate fingerprint differed from the two certificates in `otacerts.zip` | Trust-store mismatch identified; trust store was not modified |
| Whole-file signature validation incomplete | CMS certificate was extracted and matched to the package's `otacert`, but complete signature verification was not performed | Package authenticity remains unconfirmed by this investigation |
| Recovery repacking tools incomplete | The available `magiskboot` binary was ARM 32-bit; host was x86-64; `mkbootimg` was only available as a packer | No modified recovery was built or flashed |
| Alternative kernel considered | Everline package was inspected | Bundled LineageOS kernel was retained |

## 14. Final technical state

| Component / property | Recorded final state |
|---|---|
| Device | Samsung Galaxy M21 (`SM-M215F`) |
| Device codename | `m21` |
| Bootloader | Unlocked from the previous rooting project |
| Operating system | Unofficial LineageOS 24.0 |
| Android version | Android 17 |
| SDK level | 37 |
| LineageOS build | `24.0-20261024-UNOFFICIAL-m21` |
| Build date | 2026-10-24 |
| Security patch in package metadata | 2026-10-01 |
| Architecture | ARM64 (`arm64-v8a`) |
| Recovery | Lineage Recovery for `m21` |
| Recovery build type | `user` |
| ROM installation method | ADB sideload from Lineage Recovery |
| ROM installer result | Status 0 |
| First boot | Successful |
| Userdata | Formatted during clean installation |
| Google Apps | Not installed |
| Google Play Services | Not installed as part of this project |
| Alternative kernel | Not installed; bundled kernel retained |
| Magisk root on LineageOS | Not verified |
| Recovery modification | None |

## 15. Conclusion

The Samsung Galaxy M21 was successfully migrated from Samsung Android 12 / One UI Core 4.1 to an unofficial LineageOS 24.0 build based on Android 17.

The completed installation sequence was:

1. Identify the target device as Samsung Galaxy M21 (`SM-M215F`, codename `m21`).
2. Prepare the unofficial LineageOS 24.0 ROM ZIP and Lineage Recovery image on the Arch Linux host.
3. Retain the available partition backups and recovery copies.
4. Flash `vbmeta.img` to `VBMETA` and `lineage-recovery.img` to `RECOVERY` with Heimdall.
5. Boot Lineage Recovery and format data.
6. Start ADB sideload from recovery.
7. Install `lineage-24.0-m21.zip` using `adb sideload`.
8. Confirm that the ROM installer completed with status 0.
9. Reboot into Android and confirm that LineageOS booted successfully.
10. Attempt to install MindTheGapps 17 ARM64; recovery rejected the package during whole-file signature verification.
11. Inspect the recovery trust store and package certificate, identify the certificate mismatch, and leave the recovery image unmodified.

The primary objective—installing and booting LineageOS 24.0 on the Galaxy M21—was achieved. The secondary objective of installing Google Play Services and the Google Play Store was not achieved and remains separate future work.

The final operating system was left intact and usable without GApps. Recovery modification, certificate injection, and alternate recovery testing were deliberately not performed as part of this completed installation.

---

**Project status:** LineageOS installation completed; first boot successful.  
**Primary achievement:** Unofficial LineageOS 24.0 / Android 17 installed on Samsung Galaxy M21 (`SM-M215F`).  
**Outstanding item:** A verified, device-compatible method for installing MindTheGapps without compromising the working recovery or system.
