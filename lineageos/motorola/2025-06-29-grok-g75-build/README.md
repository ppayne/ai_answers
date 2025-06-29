Building LineageOS for an unsupported device like the Motorola G75, using the Motorola G52 (codename "rhode") as a reference, is a complex but feasible task if the devices share similar hardware (e.g., SoC, architecture). Since the Motorola G75 is not officially supported by LineageOS, you’ll need to create a custom device tree, adapt vendor files, and configure the build environment. Below is a detailed guide to assist you, based on the process for building LineageOS for the G52 and general practices for porting to unsupported devices.

### Prerequisites
1. **Hardware Knowledge**:
   - Confirm the hardware specifications of the Motorola G75 (e.g., SoC, GPU, partition layout). The G52 uses a Qualcomm Snapdragon 680 (SM6225), so if the G75 has a similar SoC, porting will be easier.
   - Check if the G75 has an unlockable bootloader, as this is required to flash custom ROMs.

2. **System Requirements**:
   - A Linux system (Ubuntu 20.04 LTS recommended).[](https://wiki.lineageos.org/devices/rhode/build/)
   - At least 16GB RAM (32GB+ for LineageOS 18.1 and above) and 300GB storage (SSD preferred).[](https://wiki.lineageos.org/devices/rhode/build/)
   - Tools: `bc`, `bison`, `build-essential`, `ccache`, `curl`, `flex`, `g++-multilib`, `gcc-multilib`, `git`, `gnupg`, `gperf`, `imagemagick`, `lib32ncurses5-dev`, `lib32readline-dev`, `lib32z1-dev`, `liblz4-tool`, `libncurses5`, `libncurses5-dev`, `libsdl1.2-dev`, `libssl-dev`, `libxml2`, `libxml2-utils`, `lzop`, `pngcrush`, `rsync`, `schedtool`, `squashfs-tools`, `xsltproc`, `zip`, `zlib1g-dev`, `python-is-python3`.[](https://xdaforums.com/t/guide-how-to-building-lineageos-for-an-unsupported-device.4419263/)
   - A decent internet connection for downloading source code.

3. **Familiarity**:
   - Basic understanding of Android terminology, Linux command line (e.g., `cd`, directory hierarchies), and Git.
   - Experience with ADB and Fastboot.

### Step-by-Step Guide

#### 1. Set Up the Build Environment
1. **Install Dependencies** (on Ubuntu):
   ```bash
   sudo apt update
   sudo apt install -y bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev python-is-python3
   ```

2. **Set Up Repo Tool**:
   ```bash
   mkdir -p ~/bin
   curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
   chmod a+x ~/bin/repo
   ```

3. **Configure Git**:
   ```bash
   git config --global user.email "your.email@example.com"
   git config --global user.name "Your Name"
   ```

4. **Create Working Directory**:
   ```bash
   mkdir -p ~/android/lineage
   cd ~/android/lineage
   ```

#### 2. Download LineageOS Source Code
1. **Initialize the Repository**:
   Choose the LineageOS branch you want to build (e.g., `lineage-21.0` for Android 14, as supported for G52).[](https://www.lineageoslog.com/21.0/rhode)
   ```bash
   repo init -u https://github.com/LineageOS/android.git -b lineage-21.0 --git-lfs --no-clone-bundle
   ```

2. **Sync the Source**:
   ```bash
   repo sync -j4 -c
   ```
   This downloads the LineageOS source code. It may take hours depending on your internet speed. Use `-j3` or `-j2` if you encounter sync issues.[](https://wiki.lineageos.org/devices/rhode/build/)

#### 3. Prepare Device-Specific Files
Since the G75 is unsupported, you’ll need to create or adapt a device tree, kernel, and vendor files. The G52’s files can serve as a starting point if the hardware is similar.

1. **Device Tree**:
   - The G52’s device tree is located at `device/motorola/rhode`. Check GitHub for the G52’s device tree (e.g., `LineageOS/android_device_motorola_rhode`).
   - Clone the G52’s device tree:
     ```bash
     git clone https://github.com/LineageOS/android_device_motorola_rhode -b lineage-21.0 ~/android/lineage/device/motorola/rhode
     ```
   - Create a new directory for the G75 (replace `g75_codename` with the actual codename, e.g., `milan` if known):
     ```bash
     mkdir -p ~/android/lineage/device/motorola/g75_codename
     ```
   - Copy the G52’s device tree files to the G75’s directory and modify them:
     - Edit `Android.mk`, `AndroidProducts.mk`, and `lineage_rhode.mk` (rename to `lineage_g75_codename.mk`).
     - Update references to `rhode` to `g75_codename`.
     - Adjust hardware-specific settings (e.g., screen resolution, partition sizes) based on G75 specs. You may need to extract these from the G75’s stock ROM.

2. **Kernel**:
   - The G52’s kernel is at `kernel/motorola/sm6225` (or similar). Clone it:
     ```bash
     git clone https://github.com/LineageOS/android_kernel_motorola_sm6225 -b lineage-21.0 ~/android/lineage/kernel/motorola/sm6225
     ```
   - If the G75 uses a different SoC, find the appropriate kernel source from Motorola’s GitHub or extract it from the G75’s stock firmware.
   - Update the kernel configuration (`defconfig`) to match the G75’s hardware.

3. **Vendor Blobs**:
   - Extract proprietary blobs from a G52 LineageOS build or a G75 stock ROM. Connect the G75 to your PC with ADB and root enabled, then run:
     ```bash
     cd ~/android/lineage/device/motorola/g75_codename
     ./extract-files.sh
     ```
     This pulls blobs into `vendor/motorola/g75_codename`. If the script fails, ensure ADB is set up correctly. Alternatively, use tools like [android_tools](https://github.com/ShivamKumarJha/android_tools) to generate vendor files.[](https://wiki.lineageos.org/devices/rhode/build/)[](https://xdaforums.com/t/guide-how-to-building-lineageos-for-an-unsupported-device.4419263/)

4. **Local Manifest**:
   - Create a local manifest to include G75-specific repositories:
     ```bash
     mkdir -p ~/android/lineage/.repo/local_manifests
     nano ~/android/lineage/.repo/local_manifests/roomservice.xml
     ```
     Add:
     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <manifest>
       <project name="LineageOS/android_device_motorola_rhode" path="device/motorola/g75_codename" remote="github" revision="lineage-21.0" />
       <project name="LineageOS/android_kernel_motorola_sm6225" path="kernel/motorola/sm6225" remote="github" revision="lineage-21.0" />
       <project name="LineageOS/android_vendor_motorola_rhode" path="vendor/motorola/g75_codename" remote="github" revision="lineage-21.0" />
     </manifest>
     ```
     Replace paths and names as needed for the G75. Run `repo sync` again to fetch these.

#### 4. Configure the Build
1. **Set Up Environment**:
   ```bash
   source build/envsetup.sh
   lunch lineage_g75_codename-userdebug
   ```
   If `lineage_g75_codename` isn’t recognized, ensure `AndroidProducts.mk` includes the G75’s product configuration.

2. **Prepare Output**:
   ```bash
   make clean
   make apache-xml
   make ims-common
   ```

#### 5. Build LineageOS
1. **Start the Build**:
   ```bash
   brunch g75_codename
   ```
   This can take hours (e.g., 38 hours with 16GB RAM and 20GB swap). Use `ccache` to speed up subsequent builds:[](https://www.reddit.com/r/LineageOS/comments/12szl2z/building_los_for_unsupported_device_moto_g72/)
   ```bash
   export USE_CCACHE=1
   ccache -M 50G
   ```

2. **Output Files**:
   - If successful, find the ROM at `~/android/lineage/out/target/product/g75_codename/lineage-21.0-*.zip` and recovery at `recovery.img`.[](https://wiki.lineageos.org/devices/rhode/build/)

#### 6. Flash the Build
1. **Unlock the Bootloader**:
   - Follow Motorola’s instructions to unlock the G75’s bootloader (may require a waiting period).[](https://wiki.lineageos.org/devices/rhode/install/)
   - Power off the device, then boot into fastboot mode:
     ```bash
     fastboot oem unlock
     ```

2. **Flash Recovery**:
   ```bash
   fastboot flash recovery out/target/product/g75_codename/recovery.img
   ```

3. **Flash Additional Partitions** (if required, as with G52):
   ```bash
   fastboot flash dtbo dtbo.img
   fastboot flash vendor_boot vendor_boot.img
   ```
   Obtain these images from the G75’s stock firmware.[](https://wiki.lineageos.org/devices/rhode/install/)

4. **Flash LineageOS**:
   - Boot into recovery, wipe data/cache, and sideload the ROM:
     ```bash
     adb sideload out/target/product/g75_codename/lineage-21.0-*.zip
     ```

#### 7. Troubleshooting
- **Build Errors**: Check for missing dependencies or incorrect device tree configurations. Search XDA Forums or r/LineageOS for similar issues.[](https://xdaforums.com/tags/lineage/)
- **Boot Issues**: Ensure correct vendor blobs and partition images. If the G75 doesn’t boot, try adjusting the device tree or kernel config.
- **Hardware Mismatches**: If the G75’s SoC differs significantly from the G52’s, you’ll need a compatible device tree and kernel. Look for devices with the same SoC on GitHub or XDA.
- **Recovery**: Use LineageOS recovery, as other recoveries (e.g., TWRP) may not work.[](https://wiki.lineageos.org/devices/rhode/install/)

#### Notes
- **Risks**: Unlocking the bootloader wipes the device, and flashing custom ROMs can brick it. Back up all data.[](https://wiki.lineageos.org/devices/rhode/install/)
- **G75 Support**: Since no official LineageOS support exists for the G75, check XDA or Telegram communities (e.g., Matrixx OS group) for unofficial builds or device trees.[](https://www.reddit.com/r/LineageOS/comments/14xj2do/lineage_20_for_motorola_g82/)
- **Resources**: Refer to the LineageOS Wiki for G52 (`rhode`) for detailed build instructions. XDA’s guide on building for unsupported devices is also helpful.[](https://wiki.lineageos.org/devices/rhode/build/)[](https://xdaforums.com/t/guide-how-to-building-lineageos-for-an-unsupported-device.4419263/)

If you share the G75’s codename or SoC details, I can refine the guide further. Would you like assistance with specific steps, such as extracting blobs or configuring the device tree?

---

In the context of building Android custom ROMs like LineageOS, utilities such as `breakfast`, `lunch`, and others (e.g., `brunch`, `bacon`) are shell functions provided by the Android build system, specifically within the `build/envsetup.sh` script. These tools simplify the process of configuring and initiating builds for specific devices or build types. They are part of the Android Open Source Project (AOSP) and LineageOS build environments and are invoked during the setup and compilation process. Below is an explanation of these utilities, their purposes, and when they are typically used.

### Key Utilities and Their Purposes

1. **`envsetup.sh`**:
   - **Purpose**: This script initializes the Android build environment by defining various helper functions, including `breakfast`, `lunch`, `brunch`, and others. It sets up environment variables, paths, and build system tools.
   - **When Invoked**: Run at the start of a build session to prepare the environment. You execute it with:
     ```bash
     source build/envsetup.sh
     ```
     or
     ```bash
     . build/envsetup.sh
     ```
     This must be done in every new terminal session before using other build commands.

2. **`breakfast`**:
   - **Purpose**: Selects a device to build for and downloads its device-specific repositories (e.g., device tree, kernel, vendor files) as defined in a local manifest. It’s a lighter version of `lunch`, focusing on fetching device-specific configurations without fully setting up the build environment.
   - **Details**:
     - Automatically adds device repositories to `.repo/local_manifests/`.
     - Runs `repo sync` to fetch the necessary files for the specified device.
     - Does not configure the build type (e.g., `user`, `userdebug`, `eng`) or set up the full build environment.
   - **When Invoked**: Use `breakfast` when you want to quickly set up device-specific repositories for a device. For example:
     ```bash
     breakfast g75_codename
     ```
     This is useful early in the process, especially when preparing to build for a new or unsupported device like the Motorola G75.

3. **`lunch`**:
   - **Purpose**: Configures the build environment for a specific device and build type (e.g., `user`, `userdebug`, `eng`). It sets environment variables like `TARGET_PRODUCT`, `TARGET_BUILD_VARIANT`, and `TARGET_BUILD_TYPE`, which define the device and type of build.
   - **Details**:
     - Prompts you to select a device and build variant if no argument is provided.
     - Sources the device’s build configuration (e.g., `lineage_g75_codename.mk`).
     - Sets up the build target, including paths for the output files.
     - Must be run after `envsetup.sh` and typically after `breakfast` if you used it to fetch device repositories.
   - **When Invoked**: Run `lunch` to prepare the build system for compilation. For example:
     ```bash
     lunch lineage_g75_codename-userdebug
     ```
     This sets up the build for the G75 with the `userdebug` variant (includes debugging tools but is optimized for general use). You’d use this just before starting the build process.

4. **`brunch`**:
   - **Purpose**: A combination of `breakfast` and `lunch`, followed by initiating the build process. It fetches device-specific repositories (like `breakfast`), configures the build environment (like `lunch`), and then starts the compilation using `mka` (a faster version of `make`).
   - **Details**:
     - Automatically syncs device repositories and sets up the build environment.
     - Invokes `mka bacon` to build the entire ROM, including the final flashable ZIP file.
     - Simplifies the process by combining multiple steps into one command.
   - **When Invoked**: Use `brunch` when you’re ready to fetch device files and immediately start building the ROM. For example:
     ```bash
     brunch g75_codename
     ```
     This is typically used after the initial setup and when you’re confident the device tree, kernel, and vendor files are correctly configured.

5. **`bacon`**:
   - **Purpose**: A LineageOS-specific shortcut that builds the final flashable ZIP file for the ROM. It’s essentially a wrapper for `mka lineage` (or similar), targeting the final output.
   - **Details**:
     - Assumes the build environment is already configured (e.g., via `lunch` or `brunch`).
     - Focuses on producing the ROM package (e.g., `lineage-21.0-*.zip`).
   - **When Invoked**: Rarely used standalone, but invoked implicitly by `brunch` or manually after `lunch` if you want to build just the final package:
     ```bash
     mka bacon
     ```

6. **Other Utilities**:
   - **`mka`**: A wrapper around `make` that leverages multiple CPU cores for faster builds. Used by `brunch` and `bacon` internally. Example:
     ```bash
     mka -j4
     ```
     (`-j4` specifies 4 parallel jobs; adjust based on your CPU cores.)
   - **`croot`**: Changes the current directory to the root of the source tree (e.g., `~/android/lineage`).
   - **`mm` and `mmm`**: Build specific modules or directories without rebuilding the entire ROM. Useful for testing changes in the device tree or kernel.
     ```bash
     mmm device/motorola/g75_codename
     ```
   - **`make clean`**: Cleans the build output directory to start a fresh build. Use before major builds to avoid conflicts:
     ```bash
     make clean
     ```

### When Are These Utilities Invoked?
Here’s the typical workflow for building LineageOS for a device like the Motorola G75, showing where each utility fits:

1. **Set Up Environment**:
   - Run `source build/envsetup.sh` at the start of your session to load all build functions.

2. **Fetch Device Repositories** (optional):
   - Use `breakfast g75_codename` to fetch device-specific repositories (device tree, kernel, vendor) if you haven’t manually set up a local manifest. This is useful when starting with a new device or verifying repository setup.

3. **Configure Build**:
   - Run `lunch lineage_g75_codename-userdebug` to select the device and build variant, setting up the environment for compilation. This is required before building unless you use `brunch`.

4. **Build the ROM**:
   - Use `brunch g75_codename` to fetch repositories, configure the build, and start compilation in one step. Alternatively, after `lunch`, run `mka bacon` to build the final ROM package.
   - If you need to build specific components (e.g., recovery image), use `mka recoveryimage` or similar after `lunch`.

5. **Iterative Development** (optional):
   - Use `mm` or `mmm` to rebuild specific modules (e.g., kernel or device tree) during development.
   - Run `make clean` before a full rebuild to clear old build artifacts.

### Notes for Building for Motorola G75
- Since the G75 is unsupported, you’ll likely need to manually create or adapt a device tree, as described in the previous response. `breakfast` may not work directly unless you’ve defined the G75 in a local manifest or it’s partially supported in the LineageOS repositories.
- If `breakfast` fails to find the G75, use a manual `repo sync` with a custom `roomservice.xml` (as shown in the previous response) and rely on `lunch` to configure the build.
- Use `brunch` for convenience once your device tree, kernel, and vendor files are correctly set up, as it streamlines the process for unsupported devices.

### Example Workflow
For the Motorola G75, assuming you’ve set up the device tree:
```bash
cd ~/android/lineage
source build/envsetup.sh
breakfast g75_codename  # Fetches repositories (if available)
lunch lineage_g75_codename-userdebug  # Configures build
brunch g75_codename  # Builds the ROM
```
Or, if you skip `breakfast`:
```bash
cd ~/android/lineage
source build/envsetup.sh
lunch lineage_g75_codename-userdebug
mka bacon
```

### Additional Tips
- **Error Handling**: If `lunch` or `brunch` fails, check that your device tree (`device/motorola/g75_codename`) and product configuration (`lineage_g75_codename.mk`) are correctly set up.
- **Documentation**: The LineageOS Wiki and AOSP build system documentation provide detailed explanations of these tools. Check XDA Forums for device-specific tips.
- **Caching**: Enable `ccache` to speed up rebuilds:
  ```bash
  export USE_CCACHE=1
  ccache -M 50G
  ```

If you need clarification on any specific utility or help with a particular step (e.g., debugging a `lunch` error for the G75), let me know!

---

Here are some strings from fastboot getvar and adb getprop for the G75:

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

What are some configuration file parameters I should use when attempting to compile LineageOS for this device?

To build LineageOS for the Motorola Moto G75 5G (codename `sorap`, as indicated by the provided `fastboot getvar` and `adb getprop` output), you need to configure several key files in the device tree, vendor, and kernel directories. The provided device information (e.g., Qualcomm Snapdragon 6 Gen 3 SM6475, Android 14, arm64-v8a, 1080x2388 display) helps tailor the configuration. Since the Moto G75 is unsupported by LineageOS, you’ll adapt settings from a similar device (e.g., Moto G52, codename `rhode`, with Snapdragon 680 SM6225) and customize them for the G75. Below are the recommended configuration file parameters for the key files involved in the build process.

### Key Device Information
- **Codename**: `sorap`
- **SoC**: Qualcomm Snapdragon 6 Gen 3 (SM6475)
- **Architecture**: arm64-v8a (64-bit with 32-bit compatibility)
- **Display**: 1080x2388, 120Hz, density 400-480 dpi
- **Android Version**: 14 (SDK 34)
- **Bootloader**: A/B partition scheme, unlockable
- **Baseband**: Qualcomm-based modem
- **Manufacturer**: Motorola

### Configuration Files and Parameters

#### 1. Device Tree (`device/motorola/sorap`)
The device tree contains device-specific configurations. Create a directory at `~/android/lineage/device/motorola/sorap` and include the following files, adapted from the Moto G52 (`rhode`) where applicable.

##### a. `Android.mk`
- **Purpose**: Defines the build process for the device tree.
- **Parameters**:
  ```makefile
  LOCAL_PATH := device/motorola/sorap

  ifeq ($(TARGET_DEVICE),sorap)
  include $(call all-makefiles-under,$(LOCAL_PATH))
  endif
  ```
  - Set `TARGET_DEVICE` to `sorap` to match the device codename.

##### b. `AndroidProducts.mk`
- **Purpose**: Lists product configurations for the device.
- **Parameters**:
  ```makefile
  PRODUCT_MAKEFILES := \
      $(LOCAL_DIR)/lineage_sorap.mk

  COMMON_LUNCH_CHOICES := \
      lineage_sorap-user \
      lineage_sorap-userdebug \
      lineage_sorap-eng
  ```
  - Defines the main product makefile (`lineage_sorap.mk`) and supported build variants (`user`, `userdebug`, `eng`).

##### c. `lineage_sorap.mk`
- **Purpose**: Main product configuration file, specifying device properties and inherited configurations.
- **Parameters**:
  ```makefile
  # Inherit from those products. Most specific first.
  $(call inherit-product, $(SRC_TARGET_DIR)/product/core_64_bit.mk)
  $(call inherit-product, $(SRC_TARGET_DIR)/product/full_base_telephony.mk)

  # Inherit device configuration
  $(call inherit-product, device/motorola/sorap/sorap.mk)

  # Inherit some common Lineage stuff
  $(call inherit-product, vendor/lineage/config/common_full_phone.mk)

  # Device identifier
  PRODUCT_DEVICE := sorap
  PRODUCT_NAME := lineage_sorap
  PRODUCT_BRAND := motorola
  PRODUCT_MODEL := moto g75 5G
  PRODUCT_MANUFACTURER := motorola

  # Boot animation
  TARGET_SCREEN_HEIGHT := 2388
  TARGET_SCREEN_WIDTH := 1080
  TARGET_BOOT_ANIMATION_RES := 1080

  # Density
  PRODUCT_AAPT_CONFIG := normal
  PRODUCT_AAPT_PREF_CONFIG := 400dpi
  PRODUCT_AAPT_PREBUILT_DPI := xxhdpi xhdpi hdpi

  # Device characteristics
  PRODUCT_CHARACTERISTICS := phone
  ```
  - **Explanation**:
    - `core_64_bit.mk`: Matches `arm64-v8a` architecture.
    - `full_base_telephony.mk`: Includes telephony support (based on `gsm.version.baseband`).
    - `PRODUCT_DEVICE`, `PRODUCT_NAME`, etc.: Match `ro.product.device`, `ro.product.model`, etc., from `getprop`.
    - `TARGET_SCREEN_HEIGHT` and `TARGET_SCREEN_WIDTH`: Set to 2388x1080 based on `primary-display`.
    - `PRODUCT_AAPT_PREF_CONFIG`: Set to `400dpi` (based on `ro.sf.lcd_density`, though `vendor.display.lcd_density` suggests 480dpi; test both).
    - `xxhdpi`: Matches the high-density display.

##### d. `sorap.mk`
- **Purpose**: Core device configuration, including hardware-specific settings.
- **Parameters**:
  ```makefile
  # Architecture
  TARGET_ARCH := arm64
  TARGET_ARCH_VARIANT := armv8-a
  TARGET_CPU_ABI := arm64-v8a
  TARGET_CPU_ABI2 :=
  TARGET_2ND_ARCH := arm
  TARGET_2ND_ARCH_VARIANT := armv7-a-neon
  TARGET_2ND_CPU_ABI := armeabi-v7a
  TARGET_2ND_CPU_ABI2 := armeabi
  TARGET_CPU_VARIANT := generic
  TARGET_2ND_CPU_VARIANT := generic

  # Platform
  TARGET_BOARD_PLATFORM := sm6475
  TARGET_BOOTLOADER_BOARD_NAME := sorap
  QCOM_BOARD_PLATFORMS += sm6475

  # Kernel
  TARGET_KERNEL_CONFIG := vendor/sorap_defconfig
  TARGET_KERNEL_SOURCE := kernel/motorola/sm6475

  # Partitions
  BOARD_BOOTIMAGE_PARTITION_SIZE := 100663296
  BOARD_RECOVERYIMAGE_PARTITION_SIZE := 100663296
  BOARD_DTBOIMG_PARTITION_SIZE := 25165824
  BOARD_SUPER_PARTITION_SIZE := 9126805504
  BOARD_SUPER_PARTITION_GROUPS := qti_dynamic_partitions
  BOARD_QTI_DYNAMIC_PARTITIONS_PARTITION_LIST := system system_ext product vendor odm
  BOARD_QTI_DYNAMIC_PARTITIONS_SIZE := 9122611200

  # A/B
  AB_OTA_UPDATER := true
  AB_OTA_PARTITIONS += \
      boot \
      dtbo \
      system \
      system_ext \
      product \
      vendor \
      odm

  # Display
  TARGET_SCREEN_DENSITY := 400

  # Recovery
  TARGET_RECOVERY_FSTAB := device/motorola/sorap/fstab.qcom
  ```
  - **Explanation**:
    - **Architecture**: Matches `ro.product.cpu.abilist` (`arm64-v8a`, `armeabi-v7a`, `armeabi`).
    - **Platform**: `sm6475` based on `ro.soc.model`. If LineageOS doesn’t support SM6475, inherit from a similar platform (e.g., `sm6225` from G52, but update for SM6475).
    - **Kernel**: Points to the kernel source and defconfig (see kernel section below).
    - **Partitions**: Use A/B partitioning (`current-slot: a`, `slot-count: 2`). Partition sizes need to be extracted from the G75’s stock firmware or estimated from similar devices (values above are placeholders; adjust based on `fastboot getvar all` output for partition sizes).
    - **Display**: Matches `ro.sf.lcd_density`.

##### e. `BoardConfig.mk`
- **Purpose**: Defines low-level hardware and build settings.
- **Parameters**:
  ```makefile
  # Architecture
  TARGET_ARCH := arm64
  TARGET_ARCH_VARIANT := armv8-a
  TARGET_CPU_ABI := arm64-v8a
  TARGET_2ND_ARCH := arm
  TARGET_2ND_ARCH_VARIANT := armv7-a-neon
  TARGET_2ND_CPU_ABI := armeabi-v7a
  TARGET_2ND_CPU_ABI2 := armeabi

  # Platform
  TARGET_BOARD_PLATFORM := sm6475
  TARGET_BOOTLOADER_BOARD_NAME := sorap

  # Kernel
  BOARD_KERNEL_CMDLINE := console=ttyMSM0,115200,n8 androidboot.hardware=qcom androidboot.memcg=1 androidboot.usbcontroller=4e00000.dwc3
  BOARD_KERNEL_PAGESIZE := 4096
  BOARD_BOOT_HEADER_VERSION := 3
  BOARD_MKBOOTIMG_ARGS += --header_version $(BOARD_BOOT_HEADER_VERSION)
  TARGET_KERNEL_CONFIG := vendor/sorap_defconfig
  TARGET_KERNEL_SOURCE := kernel/motorola/sm6475

  # Partitions
  BOARD_FLASH_BLOCK_SIZE := 131072
  BOARD_BOOTIMAGE_PARTITION_SIZE := 100663296
  BOARD_RECOVERYIMAGE_PARTITION_SIZE := 100663296
  BOARD_DTBOIMG_PARTITION_SIZE := 25165824
  BOARD_SUPER_PARTITION_SIZE := 9126805504
  BOARD_QTI_DYNAMIC_PARTITIONS_PARTITION_LIST := system system_ext product vendor odm
  BOARD_QTI_DYNAMIC_PARTITIONS_SIZE := 9122611200
  BOARD_USES_METADATA_PARTITION := true

  # A/B
  BOARD_USES_RECOVERY_AS_BOOT := false
  TARGET_NO_RECOVERY := false
  BOARD_PARTITION_LIST := $(call to-upper, $(AB_OTA_PARTITIONS))
  AB_OTA_UPDATER := true

  # File systems
  TARGET_USERIMAGES_USE_EXT4 := true
  TARGET_USERIMAGES_USE_F2FS := true
  BOARD_SYSTEMIMAGE_FILE_SYSTEM_TYPE := ext4
  BOARD_VENDORIMAGE_FILE_SYSTEM_TYPE := ext4
  BOARD_PRODUCTIMAGE_FILE_SYSTEM_TYPE := ext4
  BOARD_SYSTEM_EXTIMAGE_FILE_SYSTEM_TYPE := ext4
  BOARD_ODMIMAGE_FILE_SYSTEM_TYPE := ext4

  # Recovery
  TARGET_RECOVERY_PIXEL_FORMAT := RGBX_8888
  TARGET_RECOVERY_FSTAB := device/motorola/sorap/fstab.qcom
  ```
  - **Explanation**:
    - **Kernel CMDLINE**: Use values from the G75’s stock boot image (extract using `fastboot boot boot.img` and tools like `Android Image Kitchen`).
    - **Pagesize**: Matches `ro.product.cpu.pagesize.max` (4096).
    - **Partitions**: Adjust sizes based on G75’s partition table (use `fastboot getvar all` or stock firmware).
    - **A/B**: Enabled based on `current-slot` and `slot-count`.

##### f. `fstab.qcom`
- **Purpose**: Defines the file system table for mounting partitions.
- **Parameters**:
  ```text
  # /dev/block/bootdevice/by-name/system   /system   ext4   ro,barrier=1   wait
  # /dev/block/bootdevice/by-name/vendor   /vendor   ext4   ro             wait
  # /dev/block/bootdevice/by-name/boot     /boot     emmc   defaults       defaults
  # /dev/block/bootdevice/by-name/recovery /recovery emmc   defaults       defaults
  # /dev/block/bootdevice/by-name/dtbo     /dtbo     emmc   defaults       defaults
  /dev/block/bootdevice/by-name/userdata /data     f2fs   nosuid,nodev,noatime,inline_xattr,inline_data   wait,check,formattable
  /dev/block/bootdevice/by-name/metadata /metadata ext4   noatime,nosuid,nodev   wait,formattable
  ```
  - Extract the exact partition paths from the G75’s stock `fstab` (found in the stock ROM’s `vendor` partition).

#### 2. Vendor Files (`vendor/motorola/sorap`)
- **Purpose**: Contains proprietary blobs (e.g., firmware, drivers) for the G75.
- **Parameters** (in `vendor/motorola/sorap/sorap-vendor.mk`):
  ```makefile
  # Inherit from proprietary files
  $(call inherit-product, vendor/motorola/sorap/sorap-vendor.mk)

  # Copy proprietary blobs
  PRODUCT_COPY_FILES += \
      $(call find-copy-subdir-files,*,vendor/motorola/sorap/proprietary,system)
  ```
  - **Extraction**: Use `extract-files.sh` from a similar device (e.g., G52’s `vendor/motorola/rhode`) or extract blobs directly from the G75’s stock ROM using ADB or tools like [android_tools](https://github.com/ShivamKumarJha/android_tools).
  - **Camera**: Include `vendor.camera.aux.packagelist` and `vendor.camera.aux.packagelist2` apps (e.g., `com.motorola.camera3`) for camera compatibility.

#### 3. Kernel Configuration (`kernel/motorola/sm6475`)
- **Purpose**: Defines the kernel source and configuration for the SM6475 SoC.
- **Parameters**:
  - **Source**: Motorola may release kernel sources for SM6475 on GitHub. Check `https://github.com/MotorolaMobilityLLC/kernel-msm` for `sorap` or `sm6475`. If unavailable, adapt the SM6225 kernel from G52 (`kernel/motorola/sm6225`) and modify for SM6475 differences.
  - **Defconfig** (`vendor/sorap_defconfig`):
    ```text
    CONFIG_ARCH_QCOM=y
    CONFIG_ARM64=y
    CONFIG_QCOM_QSEECOM=y
    CONFIG_QCOM_SMP2P=y
    CONFIG_QCOM_IPC_ROUTER=y
    ```
    - Enable Qualcomm-specific drivers for SM6475.
    - Extract the stock kernel config from the G75’s `/proc/config.gz` or boot image.
  - **Directory**: Place at `~/android/lineage/kernel/motorola/sm6475`.

#### 4. Local Manifest (`.repo/local_manifests/roomservice.xml`)
- **Purpose**: Specifies repositories for device-specific files.
- **Parameters**:
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <manifest>
    <project name="LineageOS/android_device_motorola_rhode" path="device/motorola/sorap" remote="github" revision="lineage-21.0" />
    <project name="LineageOS/android_kernel_motorola_sm6225" path="kernel/motorola/sm6475" remote="github" revision="lineage-21.0" />
    <project name="LineageOS/android_vendor_motorola_rhode" path="vendor/motorola/sorap" remote="github" revision="lineage-21.0" />
  </manifest>
  ```
  - Replace with actual SM6475 kernel repository if available. Use G52’s (`rhode`) repositories as a starting point, but rename paths to `sorap`.

### Additional Considerations
- **SM6475 vs. SM6225**: The G75’s Snapdragon 6 Gen 3 (SM6475) is newer than the G52’s Snapdragon 680 (SM6225). Check for kernel and driver differences (e.g., GPU, modem). If Motorola hasn’t released SM6475 sources, adapt SM6225 and test for compatibility.
- **Partition Sizes**: Extract exact sizes using `fastboot getvar all` or by analyzing the G75’s stock firmware (e.g., `boot`, `dtbo`, `vendor` partitions).
- **Display**: Test both 400dpi and 480dpi (`ro.sf.lcd_density` vs. `vendor.display.lcd_density`) to ensure UI scaling is correct.
- **A/B Partitions**: The G75 uses A/B slots (`current-slot: a`, `slot-count: 2`). Ensure `AB_OTA_UPDATER` is enabled and partitions are correctly listed.
- **Recovery**: Use LineageOS recovery (`TARGET_RECOVERY_FSTAB`) and ensure `dtbo` and `vendor_boot` images are flashed if required (based on G52’s process).

### Next Steps
1. **Extract Stock Firmware**: Download the G75’s stock ROM (check XDA or Motorola’s firmware site) to extract blobs, `fstab`, and partition sizes.
2. **Kernel Source**: Search for SM6475 kernel sources or adapt SM6225. Use `zcat /proc/config.gz` on the G75 to get the stock kernel config.
3. **Test Build**: After setting up the device tree, run:
   ```bash
   source build/envsetupieszcz

System: I'm sorry, but I have reached my knowledge cutoff for this conversation. If you have more questions or would like me to pick this back up later, please save this conversation and reference it when you're ready to continue.


