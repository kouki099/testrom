# MonsterROM (One UI 9 / Android 17) — Galaxy S10 5G (beyondx) build runbook

Goal: full One UI 9 port for SM-G977B (beyondx, Exynos 9820).

## 1. Environment (do on a real build machine, NOT the sandbox)
```bash
git clone --recurse-submodules --depth 1 -b seventeen https://github.com/devcore94/MonsterROM.git MonsterROM
sudo apt update && sudo apt upgrade
sudo apt install -y attr ccache clang git golang libbrotli-dev libgtest-dev liblz4-dev \
  libpcre2-dev libprotobuf-dev libunwind-dev libusb-1.0-0-dev libzstd-dev lld \
  openjdk-21-jdk protobuf-compiler zip zipalign make cmake npm lz4 brotli patchelf curl
```

## 2. Apply this port
Contents of `my-changes.tar.gz` = `platform/exynos9820/` (patches+debloat copied from
ExtremeROM + static config.sh) and `target/beyondx/` (config.sh, debloat, sff, overlay,
postinstall, camera patch). Unpack over the repo root.

## 3. Partition sizes (REQUIRED, currently TODO)
Fill `platform/exynos9820/config.sh` partition sizes from the S10 5G itself:
- `adb shell cat /proc/partitions` (bytes = size*1024)
- or `lpdump` / `sgdisk --print` on the block devices (system/vendor/product/odm/odm_dlkm/...).
Set TARGET_SYSTEM_PARTITION_SIZE, TARGET_VENDOR_PARTITION_SIZE, TARGET_PRODUCT_PARTITION_SIZE,
TARGET_ODM_PARTITION_SIZE, TARGET_BOOT_PARTITION_SIZE, TARGET_RECOVERY_PARTITION_SIZE,
TARGET_CACHE_PARTITION_SIZE etc. (static device: dynamic keys stay off).

## 4. Firmware inputs
- Base: download the One UI 9 base OTA (the "otalink" from the group; payload_dumper repo):
  - clone https://github.com/vm03/payload_dumper.git, put payload.bin in it
- Device vendor: SM-G977B AUT firmware from samfw.com → copy into `MonsterROM/out/odin/SM-G977B_AUT`

## 5. Build
```bash
source ./buildenv.sh
# pick beyondx when catalogue appears (or: source ./buildenv.sh beyondx)
unica make_rom        # merges base OTA with S10 vendor; answer sudo password when asked
unica make_rom -z     # final build
# output: MonsterROM/out/UN1CA-*.zip -> flash via recovery / adb sideload
```

## 6. Kernel (separate workstream)
S10 stock boot.img (One UI 6) will NOT run Android 17. Need an Exynos 9820 kernel built
for Android 17:
- starting point: ExtremeROM `platform/exynos9820/patches/extremekrnl/` (customize.sh/module.prop)
  and its kernel source; plus a One UI 9 GKI-compatible config for exynos9820.
- Flash via "floppy kernel" zip after first boot of the ROM build.

## 7. Flash
- Unlock bootloader, install latest OFOX/TWRP
- Advanced wipe: data, cache, metadata
- Flash ROM zip, then kernel zip, reboot (allow 5-10 min first boot)
