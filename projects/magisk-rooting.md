# Samsung Galaxy M21 — Device Modification and Magisk Rooting

> **Device:** Samsung Galaxy M21 (`SM-M215F`, codename `m21`)  
> **Platform:** Exynos 9611 / `universal9611`, ARM64  
> **Original software:** Android 12 / One UI Core 4.1  
> **Host:** Arch Linux  
> **Objective:** Unlock the bootloader, patch the stock boot image with Magisk, flash it, and verify root access.

## 1. Project overview

This document records the work performed on a Samsung Galaxy M21, from device and firmware identification through bootloader unlocking, firmware-image preparation, Magisk patching, flashing, and post-boot verification.

The phone was successfully rooted by patching its stock boot image with Magisk 31.1 and flashing the resulting image to the `BOOT` partition with Heimdall. A later investigation into enabling multiple Android users was unsuccessful and is documented separately; it was not part of the successful rooting operation.

This is a chronological engineering record, not a generic recipe. Confirmed actions and outputs are recorded as observed. Where an exact intermediate command or output was not retained, that limitation is identified rather than filled with an invented detail.

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

Firmware matching mattered because the boot image used for patching needed to come from the firmware installed on this exact device.

### 2.3 Host tools

The host computer ran Arch Linux. The project used ADB for host-to-phone communication, Heimdall for Samsung Download Mode flashing, and Magisk for boot-image patching and root access. Odin4 was also built and tested, but it did not successfully establish a flashing session.

| Tool | Recorded version / status |
|---|---|
| Heimdall | 2.2.2 |
| Odin4 | v7.3.0-a46321b; session initiation failed |
| Magisk | 31.1; version code `31100` |

## 3. Firmware acquisition and inspection

The matching firmware package was obtained as:

```text
SAMFW.COM_SM-M215F_INS_M215FXXS3CWK1_fac.zip
```

Its size was approximately 4.7 GB. The archive's MD5 checksum was verified:

```text
268423eef3d3bd5435df18df7b09db12
```

The package matched the device's recorded AP build, `M215FXXS3CWK1`, and the INS region.

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

The original `boot.img` and `recovery.img` were retained. The patched `magisk_boot.img` was retained as the image that successfully booted.

## 4. Bootloader unlocking

The bootloader was unlocked on the phone using Samsung's Download Mode unlock flow.

### 4.1 Preparation

The OEM unlocking option was available in Developer options and was enabled before entering Download Mode. Before proceeding, the user confirmed that there was no important device data that needed to be backed up. This was important because Samsung's unlock operation performs a factory reset.

### 4.2 Entering Download Mode

1. Power the phone off.
2. Press and hold **Volume Up + Volume Down**.
3. While holding both volume buttons, connect the phone to the computer using USB.
4. Release the buttons when the Samsung warning / Download Mode screen appears.

### 4.3 Confirming the unlock

1. At the warning screen, long-press **Volume Up** to enter the bootloader-unlock confirmation screen.
2. Review the warning.
3. Press **Volume Up** to confirm unlocking.

The phone then performed the unlock operation and factory reset.

### 4.4 Post-reset verification

After the reset, the phone was set up again and connected to the internet during setup. In Developer options, the OEM unlocking entry was greyed out and displayed:

```text
Bootloader already unlocked
```

That was the observed confirmation that the bootloader had been unlocked successfully.

**Data impact:** The factory reset was an expected consequence of unlocking, not an unexpected failure.

## 5. Magisk boot-image preparation

### 5.1 Source image and Magisk version

The stock `boot.img` extracted from the matching firmware was used as the source. Magisk version 31.1 was used; the application package was:

```text
magisk-31.1.apk
```

### 5.2 Patching result

The stock boot image was patched with Magisk 31.1. The resulting image was named:

```text
magisk_boot.img
```

That was the image later flashed successfully.

The exact taps used inside the Magisk application and the exact file-picker paths selected during patching were not retained in the available command history. The successful result is independently supported by the resulting image, the matching BOOT-partition capacity, the successful flash, and the later MagiskSU verification.

The successful procedure used the patched **boot** image. The recovery image was not flashed as part of this procedure.

## 6. Flashing-tool investigation

The patched image needed to be written to the phone's `BOOT` partition while the phone was in Samsung Download Mode.

### 6.1 Odin4 attempt

Odin4 source was obtained and built on Arch Linux. The resulting executable reported:

```text
v7.3.0-a46321b
```

Odin4 was attempted first, but session initiation failed. It was not used for the successful flash. The exact Odin4 error text was not retained, so no specific error code or cause is asserted here.

### 6.2 Heimdall installation and inspection

Heimdall was installed on Arch Linux. Its version was checked with:

```bash
heimdall --version
```

Recorded output:

```text
2.2.2
```

Heimdall successfully communicated with the phone in Download Mode. The device's PIT information was read to identify partition names, IDs and capacities.

The recorded partition details were:

| Partition | ID | Capacity |
|---|---:|---:|
| `BOOT` | 12 | 61,865,984 bytes |
| `RECOVERY` | 13 | 71,106,560 bytes |

The patched `magisk_boot.img` was 61,865,984 bytes, exactly matching the reported BOOT partition capacity. This check was performed before flashing. The recovery image was retained but not flashed.

## 7. Flashing the patched boot image

With the phone in Download Mode and connected to the Arch Linux computer, the following command was used:

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

The command completed successfully. The phone was then rebooted and Android started successfully.

The successful flash targeted `BOOT`. The stock recovery image was not flashed, and no recovery-partition modification was required for this root method.

## 8. First boot and root verification

After flashing, the phone booted successfully into Android. ADB was used from Arch Linux to communicate with the running phone.

The recorded ADB serial was:

```text
RZ8NB02YQ4J
```

The device appeared in:

```bash
adb devices
```

### 8.1 Root identity

The following command was run from the host:

```bash
adb shell su -c "id"
```

Observed output:

```text
uid=0(root) gid=0(root) groups=0(root) context=u:r:magisk:s0
```

This confirmed that the command executed with UID 0 and the Magisk SELinux domain.

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

SELinux remained in `Enforcing` mode after rooting. Root access was working without changing SELinux to permissive mode.

### 8.5 Interactive shell behaviour

During later shell use, an interactive `su` attempt from an ADB shell did not visibly change the shell identity. This initially looked like a possible root-shell issue, but command-based verification had already proved that MagiskSU worked.

The following command was used to start a persistent root shell:

```sh
su -c sh
```

Inside that shell, identity could be checked with:

```sh
id
```

The command distinctions are important:

- `su -c "id"` runs `id` as root and then exits.
- `su -c sh` starts a shell as root.
- `su id` is not the correct syntax for running `id`; it attempts to execute a command named `id` through `su`.
- `su -c cd /data` does not change the current shell's directory because `cd` runs in a separate child shell.
- `sudo` is not normally available in Android's shell. Magisk root commands use `su`.

The root result was confirmed by the UID 0 output, MagiskSU version, Magisk path, and successful boot.

## 9. Working files and cleanup

The working firmware directory was cleaned after the successful root operation. The recorded retained files in `~/Downloads/M21-firmware/` included:

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

Temporary or redundant files removed during cleanup included:

- `magisk_patched-31100_R3DbE.tar`, because the actual patched `magisk_boot.img` was retained.
- `magisk_patched_recovery.img`, which was not used.
- Intermediate stock and patched recovery-kernel files.
- `boot.img.lz4` and `recovery.img.lz4`.
- An Odin4 log and image-inspection directories that were no longer required.

A SHA-256 checksum recorded for the patched AP TAR archive before it was removed was:

```text
063e5d508674e1f2faa8c17f96ffe993a7ebed24e242a7b1dce54c720296f9c5
```

This checksum belongs to the patched AP TAR archive, not to `magisk_boot.img`.

The exact final contents of the separate long-term archive directory were not verified in the retained record. The list above describes the recorded working-directory contents after cleanup and does not guarantee that every file was later copied to the archive.

## 10. Follow-up investigation: multiple Android users

After rooting, a separate investigation began to determine whether the firmware could support another Android user. This was not part of the successful root operation.

### 10.1 Existing users

From the phone's root shell:

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

An ADB command was first entered inside the Android shell:

```sh
adb shell pm create-user Aromal
```

It returned:

```text
sh: adb: inaccessible or not found
```

This happened because `adb` is a host-side command and was being invoked inside the phone's shell. The Android package-manager command `pm`, however, can be run directly on the phone.

### 10.3 User-creation attempt

The command was then run in the correct context:

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

The provisioning state was checked:

```sh
settings get global device_provisioned
```

Output:

```text
1
```

The firmware reported a maximum supported user count of one. The provisioning value indicated that the device had completed initial setup.

### 10.5 Overlay search

A targeted search for user-related active resource overlays was run:

```sh
cmd overlay list | grep -i user
```

No matching lines were returned. This only established that the command output contained no matching overlay names; it did not establish whether multi-user support could be enabled through another mechanism.

A further search was proposed for the framework configuration keys:

```sh
grep -R -n -E 'config_multiuserMaximumUsers|config_enableMultiUserUI' \
  /system/framework /system/etc /product/etc /vendor/etc 2>/dev/null
```

No output from this search was preserved in the project record. No framework configuration was changed, and no second user was created.

**Status:** Multi-user support remained unavailable through the normal user-creation interface. The firmware reported a maximum of one user. This follow-up investigation was not completed as a modification.

## 11. Problems encountered and how they were handled

| Problem | Observation | Action / outcome |
|---|---|---|
| Odin4 session initiation | Odin4 v7.3.0-a46321b did not establish a flashing session | Switched to Heimdall 2.2.2; Heimdall communicated with the phone and completed the BOOT flash |
| Confirming the correct target partition | The BOOT partition and image size needed to be checked | Read PIT information; BOOT was ID 12 with a capacity matching `magisk_boot.img` |
| Interactive `su` did not visibly elevate | Interactive shell behaviour was confusing | Verified with `su -c "id"` and used `su -c sh` for a persistent root shell |
| ADB command entered on the phone | `adb` was not found inside Android's shell | Ran ADB on Arch Linux and Android `pm` commands on the phone |
| Creating a second Android user | `pm create-user` returned maximum-user-limit error code 6 | Checked `pm get-max-users`; firmware reported a maximum of one user |
| zsh unmatched wildcard | zsh returned `no matches found` when a wildcard matched no files | Used explicit paths or inspected files rather than relying on an unmatched wildcard |

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

1. Identify the exact device and firmware build.
2. Obtain and verify the matching firmware package.
3. Enable OEM unlocking and unlock the bootloader through Samsung Download Mode.
4. Complete the resulting factory reset and verify the unlocked state in Developer options.
5. Extract the stock boot image.
6. Patch that image with Magisk 31.1.
7. Test Odin4, encounter a session-initiation failure, and move to Heimdall.
8. Read the device PIT and verify the BOOT partition details.
9. Flash `magisk_boot.img` to `BOOT`.
10. Reboot and verify MagiskSU root access, Magisk's path, and SELinux enforcement.

The central result was not merely the installation of a root-management application. The phone booted successfully with the patched boot image, and root access was independently verified from the Android shell.

The later multi-user investigation exposed a firmware-reported limit of one Android user. It remains a documented unsuccessful investigation, not a completed feature.
