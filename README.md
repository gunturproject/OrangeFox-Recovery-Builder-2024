# OrangeFox 12.1 builder for Infinix X6515

This GitHub Actions workflow builds an OrangeFox recovery-as-boot image for the X6515/H6127 Android 12 firmware targeted by the device tree. It is a build and static-validation workflow, not a device certification or flashing workflow.

## Run

Open **Actions → OrangeFox - Build X6515 → Run workflow**. Reviewed defaults are:

- OrangeFox manifest: `12.1` (`fox_12.1`)
- Device tree: `gunturproject/android_device_infinix_Infinix-X6515`
- Device tree branch: `vnd_x6515_h6127-user-12-SP1A.210812.016-308958-release-keys`
- Target: `bootimage` (recovery-as-boot)
- Size mode: `standard`
- Repo sync jobs: `16`

Inputs are passed through environment variables, validated, and shell-quoted before use. This workflow only accepts HTTPS GitHub repositories, normalized relative tree paths, the X6515 device name, the reviewed OrangeFox 12.1 manifest, and the bootimage target.

## Image-size modes

`standard` is the default and preserves normal OrangeFox extras. The stock `boot_a` and `boot_b` partitions are 32 MiB, so the workflow rejects any output larger than 33,554,432 bytes and checks the Android boot-image header/components before uploading it.

`drastic` sets OrangeFox's experimental `FOX_DRASTIC_SIZE_REDUCTION=1`. Upstream documents that mode as a last resort because it removes many OrangeFox extra binaries and needs extensive feature testing. Use it only if a standard build exceeds the device's fixed partition limit; a smaller image is not automatically a better or smoother recovery.

The workflow uses the OrangeFox vanilla profile and disables MIUI-specific features for this non-Xiaomi A/B device. It does not enable LZMA ramdisk compression. Theme rendering and responsiveness still need measurement on the physical phone.

## Validation and artifacts

Before compiling, CI checks the X6515 product/tree identity, MT6761 target, prebuilt kernel and DTB hashes, boot header v2 parameters, explicit kernel/ramdisk/tags/DTB offsets, the 32 MiB partition limit, F2FS and fstab crypto configuration, stock fingerprint, and selected recovery modules. After building, it verifies the output's absolute kernel, ramdisk, tags, and 64-bit DTB load addresses against the supplied stock header. It records the device-tree commit, OrangeFox sync-script commit, resolved `manifest.lock.xml`, build provenance, image SHA-256, and image artifact. Output validation is structural only; CI does not boot or flash the image.

The artifact contains `boot.img`, its SHA-256 sidecar, source commit identifiers, provenance, and the resolved manifest lockfile. Builds are uploaded as workflow artifacts; this workflow does not publish a GitHub Release or flash a device.

## Important compatibility limits

A compile does not prove touchscreen, display, brightness, USB/ADB/MTP/OTG, slot switching, backup/restore, dynamic partitions, formatting, encryption/decryption, Keymaster/TEE, or AVB behavior. The device tree currently lacks an audited `proprietary-files.txt` and packaged vendor crypto/Keymaster/TEE services. Enabling crypto flags is not proof that encrypted `/data` will decrypt.

The scatter file gives the physical `super` capacity but cannot prove logical partition extents; use a clean production `super.img` and `lpdump`. The supplied package lists A/B vendor_boot partitions but no vendor_boot image payload, so this workflow builds only `boot.img` and does not assert the contents or runtime role of vendor_boot.

Do not permanently flash an untested image. Keep and test a stock firmware recovery path, and validate all required functions on the exact X6515/H6127 firmware first. See the device-tree README and `tools/validate_x6515_firmware.py` for the read-only static firmware audit.
