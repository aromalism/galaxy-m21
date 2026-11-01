# Samsung Galaxy M21 — Device Modification and Magisk Rooting

> **Device:** Samsung Galaxy M21 (`SM-M215F`, codename `m21`)  
> **Platform:** Exynos 9611 / `universal9611`, ARM64  
> **Original software:** Android 12 / One UI Core 4.1  
> **Host:** Arch Linux  
> **Objective:** Bootloader unlocking, Magisk boot-image patching, flashing, and root-access verification.

## 1. Project overview

This document records the device-modification process performed on a Samsung Galaxy M21, from device and firmware identification through bootloader unlocking, firmware-image preparation, Magisk patching, flashing, and post-boot root verification.

Root access was established by patching the stock boot image with Magisk 31.1 and flashing the resulting image to the `BOOT` partition with Heimdall. A subsequent investigation into enabling multiple Android users was unsuccessful and is documented separately because it was not part of the rooting process.

The document is structured as a chronological technical record rather than a general rooting tutorial. Actions and outputs are retained as recorded. Where an exact intermediate command or output is unavailable, that limitation is stated rather than replaced with an assumption.

## 2. Device identification and baseline

### 2.1 Hardware and platform

| Property | Recorded value |
|---|---|
| Device | Samsung Galaxy M21 |
| Model | `SM-M215F` |
| Codename | `m21` |
| SoC family | Exynos 9611 |
| Platform | `universal9611` |
| Architecture | ARM64 |

### 2.2 Original software and firmware

| Property | Recorded value |
|---|---|
| Android | Android 12 |
| Samsung software | One UI Core 4.1 |
| AP / build | `M215FXXS3CWK1` |
| CP / baseband | `M215FDDS3CWL1` |
| CSC | `M215FODM3CWB1` |
| CSC region | INS (India) |
| Security patch | 2023-12-01 |

The firmware package was matched to the build installed on the device so that the boot image used for patching came from the same firmware version.

### 2.3 Host tools

The host system was Arch Linux. ADB was used for communication with the device, Heimdall for flashing in Samsung Download Mode, and Magisk for boot-image patching and root management. Odin4 was also built and tested, but it failed to establish a flashing session.

| Tool | Recorded version / status |
|---|---|
| Heimdall | 2.2.2 |
| Odin4 | v7.3.0-a46321b; session initiation failed |
| Magisk | 31.1; version code `31100` |

## 3. Firmware acquisition and inspection

The matching firmware package was obtained:

```text
SAMFW.COM_SM-M215F_INS_M215FXXS3CWK1_fac.zip
```

The archive was approximately 4.7 GB. The package MD5 checksum was verified:

```text
268423eef3d3bd5435df18df7b09db12
```

The package was checked against the device's recorded AP build, `M215FXXS3CWK1`, and the INS region.

The firmware was extracted into:

```text
~/Downloads/M21-firmware/
```

Relevant extracted files included:

| File | Role |
|---|---|
| `boot.img` | Stock boot image; source for Magisk patching |
| `recovery.img` | Stock recovery image; retained for reference |
| `AP_M215FXXS3CWK1_CL25741605_QB74561328_REV00_user_low_ship_meta_RKEY_OS12.tar.md5` | AP firmware archive |
| `BL_M215FXXS3CWK1_CL25741605_QB74561328_REV00_user_low_ship.tar.md5` | Bootloader firmware archive |
| `CP_M215FDDS3CWL1_CL2378898_QB25534584_SIGNED.tar.md5` | Modem/baseband firmware archive |
| `CSC_ODM_M215FODM3CWB1_CL25741605_QB62097098_REV00_user_low_ship.tar.md5` | CSC firmware archive |
| `magisk-31.1.apk` | Magisk application |
| `magiskboot-31.1-arm64.so` | MagiskBoot ARM64 binary retained during inspection |
| `magisk_boot.img` | Patched boot image ultimately flashed |

The original `boot.img` and `recovery.img` were retained, along with the patched `magisk_boot.img` that successfully booted.

## 4. Bootloader unlocking

The bootloader was unlocked using Samsung's Download Mode unlock procedure.

### 4.1 Preparation

**OEM unlocking** was enabled in Developer options before entering Download Mode. The bootloader-unlock operation performs a factory reset; any required device data should therefore be backed up before proceeding.

### 4.2 Entering Download Mode

1. The phone was powered off.
2. **Volume Up + Volume Down** were pressed and held.
3. While both volume buttons were held, the phone was connected to the computer over USB.
4. The buttons were released when the Samsung warning / Download Mode screen appeared.

### 4.3 Confirming the unlock

1. At the warning screen, **Volume Up** was long-pressed to enter the bootloader-unlock confirmation screen.
2. The warning was reviewed.
3. **Volume Up** was pressed to confirm the unlock.

The bootloader was then unlocked, and the phone performed a factory reset.

### 4.4 Post-reset verification

After the reset, the phone was set up again and connected to the internet during setup. Developer options was then checked. The OEM unlocking entry was greyed out and displayed:

```text
Bootloader already unlocked
```

This message confirmed that the bootloader was unlocked.

**Data impact:** The factory reset was an expected part of the bootloader-unlock process, not an unexpected failure.

## 5. Magisk boot-image preparation

### 5.1 Source image and Magisk version

The stock `boot.img` extracted from the matching firmware was used as the source image. Magisk version 31.1 was used; the application package was:

```text
magisk-31.1.apk
```

### 5.2 Patching result

The stock boot image was patched with Magisk 31.1. The resulting image was named:

```text
magisk_boot.img
```

This image was subsequently flashed successfully.

The exact buttons selected inside the Magisk application and the precise file-picker paths used during patching are not available in the retained record. Those details are therefore omitted rather than reconstructed. The resulting image was subsequently flashed, its size matched the BOOT-partition capacity, the flash completed successfully, and MagiskSU root access was verified.

The patched **boot** image was used for the successful procedure. The recovery image was not flashed.

## 6. Flashing-tool investigation

The patched image needed to be written to the device's `BOOT` partition while the phone was in Samsung Download Mode.

### 6.1 Odin4 attempt

The Odin4 source was obtained and built on Arch Linux. The resulting executable reported:

```text
v7.3.0-a46321b
```

Odin4 was tested first, but session initiation failed. It was not used for the successful flash. The exact error text is no longer available; no specific error code or cause is therefore attributed to the failure.

### 6.2 Heimdall installation and inspection

Heimdall was installed on Arch Linux, and its version was checked with:

```bash
heimdall --version
```

Recorded output:

```text
2.2.2
```

Heimdall successfully communicated with the device in Download Mode. The device PIT information was read to identify partition names, IDs, and capacities.

The recorded partition details were:

| Partition | ID | Capacity |
|---|---:|---:|
| `BOOT` | 12 | 61,865,984 bytes |
| `RECOVERY` | 13 | 71,106,560 bytes |

The patched `magisk_boot.img` was 61,865,984 bytes, exactly matching the reported BOOT-partition capacity. This was checked before flashing. The recovery image was retained but not flashed.

## 7. Flashing the patched boot image

With the phone in Download Mode and connected to the Arch Linux host, the following command was executed:

```bash
sudo heimdall flash --BOOT magisk_boot.img --no-reboot --verbose
```

Argument meanings:

| Argument | Meaning |
|---|---|
| `sudo` | Runs Heimdall with the permissions required for USB access |
| `heimdall` | Samsung flashing utility |
| `flash` | Starts a partition-flashing operation |
| `--BOOT` | Selects the BOOT partition |
| `magisk_boot.img` | Image supplied for the BOOT partition |
| `--no-reboot` | Prevents Heimdall from automatically rebooting when the operation completes |
| `--verbose` | Requests detailed operation output |

The command completed successfully. The phone was then rebooted, and Android started successfully.

Only the `BOOT` partition was flashed. The stock recovery image was not flashed, and the recovery partition did not require modification for this root method.

## 8. First boot and root verification

After flashing, the phone booted into Android successfully. ADB from Arch Linux was used for device communication.

The recorded ADB serial was:

```text
RZ8NB02YQ4J
```

The device appeared in:

```bash
adb devices
```

### 8.1 Root identity

The following command was run from the host computer:

```bash
adb shell su -c "id"
```

Observed output:

```text
uid=0(root) gid=0(root) groups=0(root) context=u:r:magisk:s0
```

This output confirmed that the command ran with UID 0 in the Magisk SELinux domain.

### 8.2 MagiskSU version

```bash
adb shell su -v
```

Output:

```text
31.1:MAGISKSU
```

The version code was checked with:

```bash
adb shell su -V
```

Output:

```text
31100
```

### 8.3 Magisk path

```bash
adb shell su -c "magisk --path"
```

Output:

```text
/debug_ramdisk
```

### 8.4 SELinux status

```bash
adb shell su -c getenforce
```

Output:

```text
Enforcing
```

SELinux remained in `Enforcing` mode after rooting. Root access was available without changing SELinux to permissive mode.

### 8.5 Interactive shell behaviour

During later shell use, an interactive `su` attempt from an ADB shell did not visibly change the shell identity. This initially appeared to be a possible root-shell issue, but command-based checks had already confirmed that MagiskSU was working.

The following command starts a persistent root shell:

```sh
su -c sh
```

The shell identity was checked with:

```sh
id
```

The following command distinctions were established during troubleshooting:

- `su -c "id"` runs `id` as root and then exits.
- `su -c sh` starts a shell as root.
- `su id` is not the correct syntax for running `id`; it attempts to execute a command named `id` through `su`.
- `su -c cd /data` does not change the current shell's directory because `cd` runs in a separate child shell.
- `sudo` is not normally available in Android's shell. Magisk root commands use `su`.

Root access was confirmed through the UID 0 output, the MagiskSU version, the Magisk path, and the successful boot.

## 9. Working files and cleanup

After the rooting process, the working firmware directory was cleaned up. The files recorded as retained in `~/Downloads/M21-firmware/` included:

```text
AP_M215FXXS3CWK1_CL25741605_QB74561328_REV00_user_low_ship_meta_RKEY_OS12.tar.md5
BL_M215FXXS3CWK1_CL25741605_QB74561328_REV00_user_low_ship.tar.md5
boot.img
CP_M215FDDS3CWL1_CL2378898_QB25534584_SIGNED.tar.md5
CSC_ODM_M215FODM3CWB1_CL25741605_QB62097098_REV00_user_low_ship.tar.md5
magisk-31.1.apk
magiskboot-31.1-arm64.so
magisk_boot.img
recovery.img
```

The original firmware ZIP remained separately in `~/Downloads`.

The following temporary or redundant files were removed during cleanup:

- `magisk_patched-31100_R3DbE.tar`, because the actual patched `magisk_boot.img` was retained.
- `magisk_patched_recovery.img`, which was not used.
- Intermediate stock and patched recovery-kernel files.
- `boot.img.lz4` and `recovery.img.lz4`.
- An Odin4 log and image-inspection directories that were no longer required.

Before removal of the patched AP TAR archive, its SHA-256 checksum was recorded:

```text
063e5d508674e1f2faa8c17f96ffe993a7ebed24e242a7b1dce54c720296f9c5
```

This checksum belongs to the patched AP TAR archive, not to `magisk_boot.img`.

The exact final contents of the separate long-term archive directory were not verified in the retained record. The list above describes the recorded working-directory contents after cleanup and does not establish that every file was subsequently copied to the archive.

## 10. Follow-up investigation: multiple Android users

After rooting, a separate investigation examined whether another Android user could be enabled in the firmware. This was not part of the successful rooting operation.

### 10.1 Existing users

From the device root shell, the following command was run:

```sh
pm list users
```

Output:

```text
Users:
        UserInfo{0:Owner:c13} running
```

Only the Owner profile (user ID `0`) was listed.

### 10.2 Initial command-context mistake

An ADB command was initially entered from inside the Android shell:

```sh
adb shell pm create-user Aromal
```

It returned:

```text
sh: adb: inaccessible or not found
```

This occurred because `adb` is a host-side command and was entered inside the device shell. The Android package-manager command `pm`, by contrast, can be run directly on the phone.

### 10.3 User-creation attempt

The user-creation command was then run directly in the device shell:

```sh
pm create-user Aromal
```

Android returned:

```text
Error: android.os.ServiceSpecificException: Cannot add user. Maximum user limit is reached. (code 6)
Error: couldn't create User.
```

### 10.4 Checking the reported limit

```sh
pm get-max-users
```

Output:

```text
Maximum supported users: 1
```

The provisioning state was checked with:

```sh
settings get global device_provisioned
```

Output:

```text
1
```

The firmware reported a maximum supported user count of one. The provisioning value indicated that the phone had completed its initial setup.

### 10.5 Overlay search

The active resource overlays were searched for user-related entries with:

```sh
cmd overlay list | grep -i user
```

The search returned no matching lines or overlay names. This result did not establish whether multi-user support could be enabled through another mechanism.

A search for relevant framework configuration keys was also performed:

```sh
grep -R -n -E 'config_multiuserMaximumUsers|config_enableMultiUserUI' \
  /system/framework /system/etc /product/etc /vendor/etc 2>/dev/null
```

The output from this search was not preserved. No framework configuration was changed, and no second user was created.

**Status:** Another user could not be created through Android's normal user-creation interface. The firmware reported a maximum of one user. This investigation did not result in a system modification.

## 11. Problems encountered and how they were handled

| Problem | Observation | Action / outcome |
|---|---|---|
| Odin4 session initiation | Odin4 v7.3.0-a46321b did not establish a flashing session | The flashing process was moved to Heimdall 2.2.2, which communicated with the device and completed the BOOT flash |
| Confirming the correct target partition | The BOOT partition and image size required verification | The PIT information identified BOOT as ID 12, with a capacity matching `magisk_boot.img` |
| Interactive `su` did not visibly elevate | The interactive shell identity did not visibly change | Root was verified with `su -c "id"`; `su -c sh` opened a persistent root shell |
| ADB command entered on the phone | The host-side `adb` command was entered inside Android's shell, where it was unavailable | ADB was run on Arch Linux, while Android `pm` commands were run directly on the device |
| Creating a second Android user | `pm create-user` returned maximum-user-limit error code 6 | `pm get-max-users` reported a firmware maximum of one user |
| zsh unmatched wildcard | zsh returned `no matches found` when a wildcard matched no files | Explicit paths or file inspection were used instead of relying on an unmatched wildcard |

## 12. Final technical state

| Component / property | Recorded final state |
|---|---|
| Device | Samsung Galaxy M21 (`SM-M215F`) |
| Firmware build | `M215FXXS3CWK1` |
| Android | Android 12 / One UI Core 4.1 |
| Bootloader | Unlocked |
| Root solution | Magisk 31.1 |
| Root implementation | `31.1:MAGISKSU` |
| Magisk version code | `31100` |
| Flashed partition | `BOOT` |
| Flashing utility | Heimdall 2.2.2 |
| Boot after flashing | Successful |
| Root identity | `uid=0(root)` |
| Magisk path | `/debug_ramdisk` |
| SELinux | `Enforcing` |
| Additional Android user | Not created; reported maximum is 1 |

## 13. Conclusion

The Galaxy M21 was successfully moved from its original stock-firmware state to a bootloader-unlocked and Magisk-rooted state.

The completed workflow was:

1. The device and firmware build were identified.
2. The matching firmware package was obtained and verified.
3. OEM unlocking was enabled, and the bootloader was unlocked through Samsung Download Mode.
4. The resulting factory reset was completed, and the unlocked state was verified in Developer options.
5. The stock boot image was extracted.
6. The image was patched with Magisk 31.1.
7. Odin4 was tested; after session initiation failed, the process was moved to Heimdall.
8. The device PIT was read, and the BOOT partition details were verified.
9. `magisk_boot.img` was flashed to `BOOT`.
10. The phone was rebooted, and MagiskSU root access, the Magisk path, and SELinux enforcement were verified.

The result was more than the installation of a root-management application: the device booted successfully with the patched boot image, and root access was independently verified from the Android shell.

The subsequent multi-user investigation showed that the firmware reported a limit of one Android user. It is recorded as an unsuccessful investigation, not as a completed feature.
