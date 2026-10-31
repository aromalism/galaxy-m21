# Magisk Root Installation — Samsung Galaxy M21 (SM-M215F)

**Project status:** Completed successfully  
**Device:** Samsung Galaxy M21 (`SM-M215F`, codename `m21`)  
**Host operating system:** Arch Linux  
**Android:** Android 12 / One UI Core 4.1  
**Root solution:** Magisk 31.1 (`31100`)  
**Flashing utility:** Heimdall 2.2.2

## 1. Objective and scope

The objective was to obtain working root access on the Samsung Galaxy M21 using a Magisk-patched boot image, with the preparation and flashing work performed from an Arch Linux computer.

The approach used the stock boot image matching the firmware installed on the phone. That image was patched with Magisk and the resulting image was written to the device's `BOOT` partition. The phone then had to boot into Android successfully, after which root access was checked through Android Debug Bridge (ADB).

This record covers the boot-image-based Magisk installation and the verification performed after the flash. It does not describe a custom ROM installation, custom kernel build, AOSP build, or complete firmware replacement.

## 2. Identifying the device and installed firmware

The target was identified as a Samsung Galaxy M21, model `SM-M215F`, with device codename `m21`. The device uses the Exynos 9611 platform (`universal9611`) and ARM64 architecture.

The software identifiers recorded on the device before the modification were:

| Property | Recorded value |
|---|---|
| Android version | Android 12 |
| Samsung software | One UI Core 4.1 |
| AP / PDA build | `M215FXXS3CWK1` |
| CP / modem build | `M215FDDS3CWL1` |
| CSC build | `M215FODM3CWB1` |
| CSC region | INS (India) |
| Android security patch level | 2023-12-01 |

Matching the firmware to the exact device and installed build was important because the boot image being patched needed to correspond to the software running on the phone. The work therefore used firmware identified for `SM-M215F` with AP build `M215FXXS3CWK1`.

## 3. Obtaining and checking the firmware package

The matching firmware archive used during the work was:

```text
SAMFW.COM_SM-M215F_INS_M215FXXS3CWK1_fac.zip
```

Its MD5 checksum was verified as:

```text
268423eef3d3bd5435df18df7b09db12
```

The firmware package was unpacked on the Arch Linux host. The extracted firmware components included the AP, BL, CP, and CSC packages. The package names recorded during the work were:

```text
AP_M215FXXS3CWK1_CL25741605_QB74561328_REV00_user_low_ship_meta_RKEY_OS12.tar.md5
BL_M215FXXS3CWK1_CL25741605_QB74561328_REV00_user_low_ship.tar.md5
CP_M215FDDS3CWL1_CL2378898_QB25534584_SIGNED.tar.md5
CSC_ODM_M215FODM3CWB1_CL25741605_QB62097098_REV00_user_low_ship.tar.md5
```

The stock `boot.img` was extracted from the matching firmware for use as the Magisk patch source. The stock `recovery.img` was also retained. Both were kept separate from the patched image so that the original firmware images remained available.

## 4. Preparing the Linux environment and tools

The device preparation, firmware handling, and flashing work were performed from Arch Linux. The tools involved in the completed workflow were:

| Tool | Recorded version / detail | Use in the project |
|---|---|---|
| Android Debug Bridge (ADB) | Platform-tools | Communicating with Android and checking root access |
| Heimdall | `2.2.2` | Flashing the patched image from Linux |
| Magisk | `31.1` / version code `31100` | Patching the stock boot image and providing MagiskSU |
| Odin4 | `v7.3.0-a46321b` | Alternative flashing utility investigated, but not used for the successful flash |

The phone was connected to the Arch Linux computer over USB. ADB detected the phone before the flash and remained usable after the phone rebooted.

## 5. Unlocking the bootloader

The device bootloader was unlocked before the patched boot image was flashed. An unlocked bootloader was a prerequisite for the boot-image modification and flashing procedure used in this project.

The exact sequence of screens, button combinations, and commands used during the unlocking stage was not retained in the project record. For that reason, this document records the confirmed state of the device rather than reconstructing an unverified sequence.

Bootloader unlocking can erase user data and reduce the device's security. The consequences and device-specific instructions should be checked before repeating this operation on another phone.

## 6. Extracting the stock boot and recovery images

After unpacking the matching firmware, the stock boot image was available as:

```text
boot.img
```

The stock recovery image was also extracted:

```text
recovery.img
```

The stock boot image was used as the source for the Magisk patch. The stock recovery image was preserved separately. It was not modified or flashed during the successful root installation.

## 7. Patching the stock boot image with Magisk

Magisk version 31.1 was used to patch the stock `boot.img`. The patched output was saved as:

```text
magisk_boot.img
```

The resulting image was approximately 59 MB. Its exact size was later confirmed as **61,865,984 bytes**.

A separate patched recovery image was also present during the preparation and inspection work, but it was not used for the successful installation. The image selected for flashing was `magisk_boot.img`, and the intended destination was the `BOOT` partition.

## 8. Trying Odin4 and selecting Heimdall

Odin4 was investigated as a possible flashing tool on Linux. The source used was version `v7.3.0-a46321b`, and a local build was available. The attempt did not establish a successful flashing session; it failed during session initiation. Odin4 was therefore not used to write the patched image.

Heimdall was installed on the Arch Linux host. Its installed version was:

```text
2.2.2
```

Heimdall successfully communicated with the phone in download mode and became the flashing utility used for the actual operation.

## 9. Inspecting the partition table

Before flashing, the phone's Partition Information Table (PIT) was inspected to identify the relevant partition entries and their reported capacities.

The recorded entries were:

| Partition | PIT partition ID | Reported capacity |
|---|---:|---:|
| `BOOT` | 12 | 61,865,984 bytes |
| `RECOVERY` | 13 | 71,106,560 bytes |

The patched image's exact size, 61,865,984 bytes, matched the reported capacity of the `BOOT` partition. This was a useful consistency check when confirming the target partition for the image.

## 10. Flashing the patched boot image with Heimdall

With the phone in download mode and `magisk_boot.img` available on the Arch Linux host, Heimdall was used to flash the image to the `BOOT` partition.

The successful command was:

```bash
sudo heimdall flash --BOOT magisk_boot.img --no-reboot --verbose
```

The command's options were used as follows:

- `flash` started a partition-flashing operation.
- `--BOOT magisk_boot.img` selected the `BOOT` partition and supplied the patched image.
- `--no-reboot` prevented Heimdall from automatically rebooting the phone when the operation finished.
- `--verbose` enabled detailed operation output.

Heimdall completed the `BOOT` partition flash successfully. The operation used the patched boot image; the `RECOVERY` partition was not flashed. This Heimdall operation was the successful flashing step, unlike the earlier Odin4 attempt.

## 11. First boot after the flash

After the flash operation, the phone was rebooted. It successfully started Android 12 instead of remaining unable to boot. ADB communication was available again after startup, allowing verification to continue.

The successful boot was the first practical confirmation that the patched boot image was compatible with the installed firmware and that the flash had left the phone in a working state.

## 12. Verifying Magisk and root access

Root access was checked from the Arch Linux host through ADB using MagiskSU:

```bash
adb shell su -c "id"
```

The command returned:

```text
uid=0(root) gid=0(root) groups=0(root) context=u:r:magisk:s0
```

The `uid=0(root)` result confirmed that the command was executing with root privileges. The reported security context was `u:r:magisk:s0`.

The MagiskSU version was checked with:

```bash
adb shell su -v
```

Output:

```text
31.1:MAGISKSU
```

The Magisk version code was checked with:

```bash
adb shell su -V
```

Output:

```text
31100
```

The Magisk runtime path was checked with:

```bash
adb shell su -c "magisk --path"
```

Output:

```text
/debug_ramdisk
```

SELinux mode was checked with:

```bash
adb shell su -c getenforce
```

Output:

```text
Enforcing
```

These checks established that MagiskSU was working, that the installed Magisk version was 31.1, and that SELinux remained in `Enforcing` mode.

## 13. Understanding the Android shell and root-shell commands

During the verification work, a distinction was made between the normal Android ADB shell and a root shell. A normal `adb shell` session starts with the Android `shell` user. The command form `su -c` successfully ran individual commands as root.

To start a persistent root shell from the Android shell, the working command was:

```sh
su -c sh
```

Once inside that root shell, the identity could be checked with:

```sh
id
```

The expected identity was `uid=0(root)`.

The command forms are not interchangeable:

```sh
su -c "id"
```

runs one command as root, while:

```sh
su -c sh
```

starts a shell with root privileges. A command such as `su id` is not the equivalent of `su -c "id"`; it attempts to execute a command named `id` through `su` rather than passing it as a command with the `-c` option.

The Android shell does not normally include the Linux-host `sudo` command. `sudo` was used on Arch Linux to run Heimdall with the required host permissions; it was not expected to be available inside Android.

## 14. Final outcome

The primary objective was achieved. The Samsung Galaxy M21 booted successfully after the patched boot image was flashed, and MagiskSU returned a root identity through ADB.

The verified result was:

- Target device: Samsung Galaxy M21 (`SM-M215F`, codename `m21`).
- Firmware baseline: AP `M215FXXS3CWK1`, CP `M215FDDS3CWL1`, CSC `M215FODM3CWB1`.
- Root solution: Magisk 31.1 (`31100`).
- Patched image: `magisk_boot.img`.
- Flashing utility: Heimdall 2.2.2 on Arch Linux.
- Destination partition: `BOOT` (PIT partition ID 12).
- Flash result: successful.
- Post-flash boot: Android 12 started successfully.
- Root verification: `uid=0(root)` with context `u:r:magisk:s0`.
- Magisk runtime path: `/debug_ramdisk`.
- SELinux mode: `Enforcing`.

**Result: Magisk root was successfully installed and verified on the Samsung Galaxy M21.**

## 15. Scope boundaries and retained limitations

This project documents a boot-image-based Magisk root installation. The following activities were not part of the successful root procedure:

- A custom ROM or complete operating-system replacement was not installed.
- A custom kernel was not built or installed.
- AOSP was not built from source.
- The `RECOVERY` partition was not flashed.
- The separate patched recovery image was not used.
- The system partition was not manually modified to remove preinstalled applications.
- No preinstalled application is claimed to have been permanently removed.

The exact bootloader-unlocking sequence was not retained. The full Odin4 failure log and the exact firmware-extraction command history were also not retained. The document records the confirmed outcome of those stages without inventing missing command transcripts.

## 16. Technical precautions

- Use firmware intended for the exact device model and verify that its build matches the software installed on the device. The recorded target was `SM-M215F` with AP build `M215FXXS3CWK1`.
- Do not treat boot images from another model or firmware revision as interchangeable.
- Preserve the unmodified stock boot and recovery images before modifying firmware images.
- Confirm the intended partition before flashing. In this project, `magisk_boot.img` was flashed to `BOOT`, not `RECOVERY`.
- Distinguish commands run on the Arch Linux host from commands run inside Android's shell.
- Verify root by checking the result of a root command, rather than relying only on the presence of the Magisk application.
- The observed final SELinux state was `Enforcing`; SELinux was not disabled to obtain root.
- These results apply to the recorded device and firmware combination. They should not be assumed to be a universal procedure for other Samsung models or different Galaxy M21 firmware versions.

---

**Project status:** Completed successfully.  
**Primary achievement:** Verified MagiskSU root access on Samsung Galaxy M21 (`SM-M215F`) using an Arch Linux host and Heimdall.
