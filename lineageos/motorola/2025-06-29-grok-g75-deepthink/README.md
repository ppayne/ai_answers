This project is to build LineageOS for an unsupported device, the Motorola G75 (codename `sorap`).

Previous Motorola devices have been both officially supported (the Motorola G52 codename `rhode`) and unofficially supported (the Motorola G71 codename `corfur` with configuration at [this link](https://github.com/moto-corfur/android_device_motorola_corfur)).

Some specific attributes of the Motorola G75 are taken from `adb shell getprop` and `fastboot getvar all`:

```
[ro.build.product]: [sorap]
[ro.opa.device_model_id]: [motorola-sorap]
[ro.product.device]: [sorap]
[ro.product.build.version.sdk]: [34]
[ro.product.cpu.abi]: [arm64-v8a]
[ro.product.cpu.abilist]: [arm64-v8a,armeabi-v7a,armeabi]
[ro.product.cpu.abilist32]: [armeabi-v7a,armeabi]
[ro.product.cpu.abilist64]: [arm64-v8a]
[ro.product.cpu.pagesize.max]: [4096]
[ro.product.manufacturer]: [motorola]
[ro.product.model]: [moto g75 5G]
[ro.product.motodesktop]: [1]
[ro.product.name]: [paros_g]
[ro.product.odm.brand]: [motorola]
[ro.product.odm.device]: [paros]
[ro.product.odm.manufacturer]: [motorola]
[ro.product.odm.model]: [paros]
[ro.product.odm.name]: [paros]
[ro.product.product.brand]: [motorola]
[ro.product.product.manufacturer]: [motorola]
[ro.product.product.model]: [moto g75 5G]
[ro.product.product.name]: [paros_g]
[ro.product.system.brand]: [motorola]
[ro.product.system.device]: [msi]
[ro.product.system.manufacturer]: [motorola]
[ro.product.system.model]: [moto g75 5G]
[ro.product.system.name]: [paros_g]
[ro.product.system_ext.brand]: [motorola]
[ro.product.system_ext.device]: [msi]
[ro.product.vndk.version]: [34]
[ro.property_service.version]: [2]
[ro.sf.lcd_density]: [400]
[ro.soc.manufacturer]: [QTI]
[ro.soc.model]: [SM6475]
[ro.system.build.version.qcom]: [AU_LINUX_ANDROID_LA.QSSI.14.0.R1.14.00.00.1001.115.00]
[ro.system.product.cpu.abilist]: [arm64-v8a,armeabi-v7a,armeabi]
[ro.system.product.cpu.abilist32]: [armeabi-v7a,armeabi]
[ro.system.product.cpu.abilist64]: [arm64-v8a]
[ro.zygote]: [zygote64_32]
[vendor.camera.aux.packagelist]: [com.motorola.camera2,com.motorola.camera3,com.motorola.motocit]
[vendor.camera.aux.packagelist2]: [com.motorola.ccc,com.android.settings,com.motorola.handycam]
[vendor.display.lcd_density]: [480]
[vendor.display.limit_low_framerate_layer_name_exception_list]: [AOD,Wallpaper,testSurface]
[vendor.ril.gsm.version.baseband]: [M6475_DE311_17.195.01.43R]
[gsm.version.baseband]: [M6475_DE311_17.195.01.43R PAROS_PVT_EMEADSDS_CUST]
[gsm.version.baseband1]: [M6475_DE311_17.195.01.43R PAROS_PVT_EMEADSDS_CUST]
[gsm.version.ril-impl]: [Qualcomm RIL 1.0]
(bootloader) sku: XT2437-3
(bootloader) carrier_sku: XT2437-3
(bootloader) primary-display[0]: tm_ili7807s_678_1080x2388_dsc_120hz_vid
(bootloader) primary-display[1]: _fhd
(bootloader) current-slot: a
(bootloader) running-bl-slot: _a/_a
(bootloader) running-boot-lun: 1
(bootloader) slot-count: 2
(bootloader) slot-successful:_a: no
(bootloader) slot-successful:_b: no
(bootloader) slot-unbootable:_a: no
(bootloader) slot-unbootable:_b: no
(bootloader) slot-retry-count:_a: 7
(bootloader) slot-retry-count:_b: 0
(bootloader) logical-block-size: 0x1000
(bootloader) erase-block-size: 0x1000
(bootloader) max-sparse-size: 268435456
```

After using the `repo init` command to download the LineageOS source code - what steps should be taken, what configuration files are to be modified with which values, to build/compile LineageOS for the Motorola G75?

---

To build LineageOS for the unsupported Motorola G75 (codename `sorap`) after initializing the source code with `repo init`, you need to create and configure device-specific files, adapt configurations from a similar device, set up the kernel and vendor files, and integrate the device into the LineageOS build system. The Motorola G75 runs Android 14 (SDK 34) with a Qualcomm Snapdragon 695 (SM6475) SoC and an ARM64 architecture, as derived from the provided `adb shell getprop` and `fastboot getvar all` outputs. Given its similarities to the unofficially supported Motorola G71 (codename `corfur`), which shares the same SoC, we can leverage its configuration as a starting point, adjusting for `sorap`-specific details. Below are the detailed steps and configuration files to modify, complete with specific values based on the device’s properties.

---

### Step-by-Step Process

#### 1. Initialize and Sync the LineageOS Source Code
You’ve already run `repo init`. Assuming it was executed with the appropriate branch for Android 14 (LineageOS 21.0), the command would be:

```bash
repo init -u https://github.com/LineageOS/android.git -b lineage-21.0
```

Next, download the source code:

```bash
repo sync
```

This retrieves the LineageOS 21.0 source, which aligns with Android 14, matching the G75’s `[ro.product.build.version.sdk]: [34]`.

#### 2. Create the Device Tree Directory
Create a directory for the Motorola G75’s device-specific files within the LineageOS source tree:

```bash
mkdir -p device/motorola/sorap
```

This directory will house all configuration files tailored for `sorap`.

#### 3. Configure Device-Specific Files
You’ll need to create and populate several files in `device/motorola/sorap`, adapting configurations from the Motorola G71’s `corfur` device tree (available at [https://github.com/moto-corfur/android_device_motorola_corfur](https://github.com/moto-corfur/android_device_motorola_corfur)) where possible, and adjusting based on `sorap`’s properties.

##### **File 1: `AndroidProducts.mk`**
This file lists the product makefiles for the build system.

**Content:**

```makefile
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/lineage_sorap.mk
```

**Purpose:** Directs the build system to the `lineage_sorap.mk` file, which defines the build target.

##### **File 2: `lineage_sorap.mk`**
This file defines the product configuration, inheriting device-specific settings and setting branding details.

**Content:**

```makefile
$(call inherit-product, device/motorola/sorap/device.mk)

PRODUCT_NAME := lineage_sorap
PRODUCT_DEVICE := sorap
PRODUCT_BRAND := motorola
PRODUCT_MODEL := moto g75 5G
PRODUCT_MANUFACTURER := motorola
```

**Values Explained:**
- `PRODUCT_NAME`: `lineage_sorap` – The build output identifier, combining LineageOS with the codename.
- `PRODUCT_DEVICE`: `sorap` – Matches `[ro.product.device]: [sorap]` and `[ro.build.product]: [sorap]`.
- `PRODUCT_BRAND`: `motorola` – From `[ro.product.brand]: [motorola]`.
- `PRODUCT_MODEL`: `moto g75 5G` – From `[ro.product.model]: [moto g75 5G]`.
- `PRODUCT_MANUFACTURER`: `motorola` – From `[ro.product.manufacturer]: [motorola]`.

**Note:** While `[ro.product.name]: [paros_g]` appears in the properties, LineageOS typically uses the codename (`sorap`) for `PRODUCT_DEVICE`.

##### **File 3: `device.mk`**
This file specifies device-specific configurations, such as file copies, overlays, and properties. Start by copying `device.mk` from the `corfur` repository and adjust as needed.

**Example Content (Partial):**

```makefile
# Inherit vendor configuration (to be set up later)
$(call inherit-product, vendor/motorola/sorap/sorap-vendor.mk)

# Device characteristics
PRODUCT_AAPT_CONFIG := normal
PRODUCT_AAPT_PREF_CONFIG := 400dpi

# Device properties
PRODUCT_PROPERTY_OVERRIDES += \
    ro.sf.lcd_density=400 \
    ro.soc.manufacturer=QTI \
    ro.soc.model=SM6475
```

**Values Explained:**
- `PRODUCT_AAPT_PREF_CONFIG`: `400dpi` – Matches `[ro.sf.lcd_density]: [400]`. (Note: `[vendor.display.lcd_density]: [480]` exists, but the system density takes precedence unless overridden.)
- `PRODUCT_PROPERTY_OVERRIDES`:
  - `ro.sf.lcd_density=400` – Sets the display density.
  - `ro.soc.manufacturer=QTI` and `ro.soc.model=SM6475` – Reflect `[ro.soc.manufacturer]: [QTI]` and `[ro.soc.model]: [SM6475]`.

**Adjustments from `corfur`:**
- Update file paths or package lists (e.g., for camera apps like `[vendor.camera.aux.packagelist]`) if they differ.
- Include `sorap`-specific overlays or hardware configurations as identified during testing.

##### **File 4: `BoardConfig.mk`**
This file defines hardware-specific settings, such as architecture, partitions, and kernel details. Base it on `corfur`’s `BoardConfig.mk`, modifying for `sorap`.

**Example Content (Partial):**

```makefile
# Architecture
TARGET_ARCH := arm64
TARGET_ARCH_VARIANT := armv8-a
TARGET_CPU_ABI := arm64-v8a
TARGET_CPU_ABI2 :=
TARGET_CPU_VARIANT := generic

TARGET_2ND_ARCH := arm
TARGET_2ND_ARCH_VARIANT := armv7-a-neon
TARGET_2ND_CPU_ABI := armeabi-v7a
TARGET_2ND_CPU_ABI2 := armeabi
TARGET_2ND_CPU_VARIANT := generic

# Platform
TARGET_BOARD_PLATFORM := sm6475
TARGET_BOOTLOADER_BOARD_NAME := sorap

# A/B Partitions
AB_OTA_UPDATER := true
AB_OTA_PARTITIONS := \
    boot \
    system \
    vendor \
    product \
    odm

# Block size
BOARD_FLASH_BLOCK_SIZE := 4096
```

**Values Explained:**
- **Architecture:**
  - `TARGET_ARCH := arm64` – From `[ro.product.cpu.abi]: [arm64-v8a]`.
  - `TARGET_CPU_ABI := arm64-v8a` and 32-bit support via `TARGET_2ND_*` settings – Reflects `[ro.product.cpu.abilist]: [arm64-v8a,armeabi-v7a,armeabi]` and `[ro.zygote]: [zygote64_32]`.
- **Platform:**
  - `TARGET_BOARD_PLATFORM := sm6475` – Matches the Snapdragon 695 SoC `[ro.soc.model]: [SM6475]`.
  - `TARGET_BOOTLOADER_BOARD_NAME := sorap` – Aligns with the codename.
- **A/B Partitions:**
  - `AB_OTA_UPDATER := true` – Enabled due to `(bootloader) slot-count: 2` and `(bootloader) current-slot: a`.
  - `AB_OTA_PARTITIONS` – Lists key partitions inferred from modern Qualcomm A/B devices; verify exact list via `adb shell ls -l /dev/block/by-name`.
- **Block Size:**
  - `BOARD_FLASH_BLOCK_SIZE := 4096` – Matches `(bootloader) logical-block-size: 0x1000` (hex 0x1000 = 4096 bytes).

**Adjustments from `corfur`:**
- Update `TARGET_BOARD_PLATFORM` if `corfur` uses a different naming convention for SM6475.
- Set partition sizes (e.g., `BOARD_BOOTIMAGE_PARTITION_SIZE`) by extracting from the device or matching `corfur` if identical.
- Configure kernel settings (see Step 5).

##### **File 5: `vendorsetup.sh`**
This script adds the device to the build system’s lunch menu.

**Content:**

```bash
add_lunch_combo lineage_sorap-userdebug
```

**Purpose:** Allows selection of the `sorap` target with `lunch lineage_sorap-userdebug`.

#### 4. Set Up the Kernel
The kernel can be prebuilt (extracted from the device) or compiled from source. Given `sorap`’s unsupported status, compiling from source is preferable for customization, assuming Motorola provides kernel source for the G75 or a similar SM6475 device (e.g., at [https://github.com/MotorolaMobilityLLC/kernel-msm](https://github.com/MotorolaMobilityLLC/kernel-msm)).

- **Option 1: Prebuilt Kernel**
  - Extract the kernel (e.g., `boot.img`) from the device using `dd` via ADB with root access.
  - Place it in `device/motorola/sorap/kernel`.
  - In `BoardConfig.mk`, set:
    ```makefile
    TARGET_PREBUILT_KERNEL := device/motorola/sorap/kernel
    ```

- **Option 2: Compile Kernel (Recommended)**
  - Clone the kernel source (e.g., `git clone <kernel-repo> kernel/motorola/sm6475`).
  - Identify the defconfig (e.g., `sorap_defconfig`) from Motorola’s release or adapt `corfur`’s.
  - In `BoardConfig.mk`, set:
    ```makefile
    TARGET_KERNEL_SOURCE := kernel/motorola/sm6475
    TARGET_KERNEL_CONFIG := sorap_defconfig
    ```

#### 5. Set Up Vendor Files
Proprietary blobs (e.g., for camera, modem) are required from the device since `sorap` lacks official support.

- Create `vendor/motorola/sorap`.
- Copy `extract-files.sh` and `proprietary-files.txt` from `corfur`’s vendor tree (adjust paths/names for `sorap`).
- Connect the G75 via ADB with root access and run:
  ```bash
  ./extract-files.sh
  ```
- Alternatively, extract blobs from a stock ROM if available.
- Define the vendor in a file like `sorap-vendor.mk`:
  ```makefile
  PRODUCT_COPY_FILES += $(call find-copy-subdir-files,*,vendor/motorola/sorap/proprietary,system)
  ```

#### 6. Build the ROM
- Initialize the build environment:
  ```bash
  source build/envsetup.sh
  ```
- Select the target:
  ```bash
  lunch lineage_sorap-userdebug
  ```
- Compile:
  ```bash
  mka bacon
  ```
  (`mka bacon` is a LineageOS convention to build the full ROM.)

---

### Configuration Files and Values Summary

| **File**              | **Purpose**                          | **Key Values**                                                                                     |
|-----------------------|--------------------------------------|---------------------------------------------------------------------------------------------------|
| `AndroidProducts.mk`  | Lists product makefiles             | `PRODUCT_MAKEFILES := $(LOCAL_DIR)/lineage_sorap.mk`                                             |
| `lineage_sorap.mk`    | Defines product configuration       | `PRODUCT_NAME := lineage_sorap`<br>`PRODUCT_DEVICE := sorap`<br>`PRODUCT_BRAND := motorola`<br>`PRODUCT_MODEL := moto g75 5G`<br>`PRODUCT_MANUFACTURER := motorola` |
| `device.mk`           | Device-specific settings            | `PRODUCT_AAPT_PREF_CONFIG := 400dpi`<br>`ro.sf.lcd_density=400`<br>`ro.soc.model=SM6475`          |
| `BoardConfig.mk`      | Hardware-specific configurations    | `TARGET_ARCH := arm64`<br>`TARGET_BOARD_PLATFORM := sm6475`<br>`TARGET_BOOTLOADER_BOARD_NAME := sorap`<br>`AB_OTA_UPDATER := true`<br>`BOARD_FLASH_BLOCK_SIZE := 4096` |
| `vendorsetup.sh`      | Adds device to build menu           | `add_lunch_combo lineage_sorap-userdebug`                                                        |

---

### Additional Notes
- **Validation:** After the initial build, flash the ROM and debug errors (e.g., missing blobs, HAL mismatches). Use `logcat` and `dmesg` for troubleshooting.
- **Partition Layout:** Confirm `AB_OTA_PARTITIONS` by checking `/dev/block/by-name` on the device.
- **Hardware Features:** Adjust `device.mk` for specifics like the camera (`[vendor.camera.aux.packagelist]`) or display (`tm_ili7807s_678_1080x2388_dsc_120hz_vid`) as needed.

By following these steps and tailoring the configurations to `sorap`’s specifications, you can compile LineageOS 21.0 for the Motorola G75.


