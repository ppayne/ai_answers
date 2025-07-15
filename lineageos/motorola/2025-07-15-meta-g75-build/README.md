# Building LineageOS for Motorola G75 (sorap)

## Step 1: Create Device Tree

Create a new directory for the device tree in the LineageOS source code:

```bash
mkdir -p device/motorola/sorap
```

## Step 2: Configure BoardConfig.mk

Create a new file `BoardConfig.mk` in the device tree directory:

```
device/motorola/sorap/BoardConfig.mk
```

Add the following configurations:

```make
TARGET_ARCH := arm64
TARGET_ARCH_VARIANT := armv8-a
TARGET_CPU_ABI := arm64-v8a
TARGET_CPU_ABI2 :=
TARGET_CPU_VARIANT := generic
TARGET_CPU_VARIANT_RUNTIME := kryo300

TARGET_2ND_ARCH := arm
TARGET_2ND_ARCH_VARIANT := armv8-a
TARGET_2ND_CPU_ABI := armeabi-v7a
TARGET_2ND_CPU_ABI2 := armeabi
TARGET_2ND_CPU_VARIANT := generic
TARGET_2ND_CPU_VARIANT_RUNTIME := cortex-a75

BOARD_KERNEL_CMDLINE := console=ttyMSM0,115200n8 earlycon=msm_geni_serial,0x4a90000 androidboot.hardware=qcom androidboot.console=ttyMSM0 androidboot.memcg=1 lpm_levels.sleep_disabled=1 video=vfb:640x400,bpp=32,memsize=3072000 msm_rtb.filter=0x237 service_locator.enable=1 androidboot.usbcontroller=4e00000.dwc3 swiotlb=0 loop.max_part=7 cgroup.memory=nokmem,nosocket pcie_ports=compat loop.max_part=7 iptable_raw.raw_before_defrag=1 ip6table_raw.raw_before_defrag=1 buildvariant=user
BOARD_KERNEL_BASE := 0x00000000
BOARD_KERNEL_PAGESIZE := 4096
BOARD_KERNEL_IMAGE_NAME := Image.gz
BOARD_KERNEL_SEPARATED_DTBO := true
BOARD_RAMDISK_USE_LZ4 := true
TARGET_KERNEL_SOURCE := kernel/motorola/sm6375
TARGET_KERNEL_CONFIG := vendor/motorola/sm6375-perf_defconfig
BOARD_PREBUILT_DTBOIMAGE := $(DEVICE_PATH)/dtbo.img
BOARD_PREBUILT_DTBIMAGE_DIR := $(DEVICE_PATH)/dtbs

# Partitions
BOARD_BOOTIMAGE_PARTITION_SIZE := 100663296
BOARD_DTBOIMG_PARTITION_SIZE := 25165824
BOARD_RECOVERYIMAGE_PARTITION_SIZE := 104857600
BOARD_SYSTEMIMAGE_PARTITION_SIZE := 3221225472
BOARD_USERDATAIMAGE_PARTITION_SIZE := 48318382080
BOARD_VENDORIMAGE_PARTITION_SIZE := 1073741824
BOARD_PRODUCTIMAGE_PARTITION_SIZE := 1073741824
BOARD_SYSTEM_EXTIMAGE_PARTITION_SIZE := 1073741824
BOARD_ODMIMAGE_PARTITION_SIZE := 1073741824

BOARD_SUPER_PARTITION_SIZE := 12884901888
BOARD_SUPER_PARTITION_GROUPS := qti_dynamic_partitions
BOARD_QTI_DYNAMIC_PARTITIONS_PARTITION_LIST := system system_ext product vendor odm
BOARD_QTI_DYNAMIC_PARTITIONS_SIZE := 12880736256

BOARD_FLASH_BLOCK_SIZE := 4096

TARGET_USERIMAGES_USE_EXT4 := true
TARGET_USERIMAGES_USE_F2FS := true

BOARD_AVB_ENABLE := true
BOARD_AVB_MAKE_VBMETA_IMAGE_ARGS += --flags 3
BOARD_AVB_RECOVERY_KEY_PATH := external/avb/test/data/testkey_rsa4096.pem
BOARD_AVB_RECOVERY_ALGORITHM := SHA256_RSA4096
BOARD_AVB_RECOVERY_ROLLBACK_INDEX := $(PLATFORM_SECURITY_PATCH_TIMESTAMP)
BOARD_AVB_RECOVERY_ROLLBACK_INDEX_LOCATION := 1

# Properties
TARGET_SYSTEM_PROP += $(DEVICE_PATH)/system.prop
```

## Step 3: Configure device.mk

Create a new file `device.mk` in the device tree directory:

```
device/motorola/sorap/device.mk
```

Add the following configurations:

```make
# Inherit from those products. Most specific first.
$(call inherit-product, $(SRC_TARGET_DIR)/product/core_64_bit.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/full_base_telephony.mk)

# Get non-open-source specific aspects
$(call inherit-product, vendor/motorola/sorap/sorap-vendor.mk)

# Properties
-include $(LOCAL_PATH)/properties.mk

PRODUCT_PACKAGES += \
    android.hardware.biometrics.fingerprint@2.1-service \
    android.hardware.drm@1.3-service.clearkey \
    android.hardware.graphics.composer@2.4-service \
    android.hardware.health@2.1-impl \
    android.hardware.health@2.1-service \
    android.hardware.usb@1.3-service.basic \
    vendor.qti.hardware.display.allocator-service \
    vendor.qti.hardware.display.composer-service \
    vendor.qti.hardware.display.mapper@4.0-service \
    vendor.qti.hardware.perf@2.2-service

# VNDK
PRODUCT_PACKAGES += \
    vndk_package

# VNDK-SP
PRODUCT_PACKAGES += \
    vndk-sp

# VNDK-EXT
PRODUCT_PACKAGES += \
    vndk-ext

# HIDL
PRODUCT_PACKAGES += \
    libhidltransport \
    libhwbinder

# Soong namespaces
PRODUCT_SOONG_NAMESPACES += \
    $(LOCAL_PATH) \
    hardware/google/interfaces \
    hardware/google/pixel \
    hardware/qcom-caf/common/libqti-perfd-client \
    hardware/qti \
    vendor/motorola

# Overlays
DEVICE_PACKAGE_OVERLAYS += $(LOCAL_PATH)/overlay

# Shipping API level
PRODUCT_SHIPPING_API_LEVEL := 31

# VNDK API level
PRODUCT_TARGET_VNDK_VERSION := 34

# Partitions
PRODUCT_BUILD_SUPER_PARTITION := false
PRODUCT_USE_DYNAMIC_PARTITIONS := true
```

## Step 4: Configure AndroidProducts.mk

Create a new file `AndroidProducts.mk` in the device tree directory:

```
device/motorola/sorap/AndroidProducts.mk
```

Add the following configurations:

```make
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/lineage_sorap.mk

COMMON_LUNCH_CHOICES := \
    lineage_sorap-user \
    lineage_sorap-userdebug \
    lineage_sorap-eng
```

## Step 5: Configure lineage_sorap.mk

Create a new file `lineage_sorap.mk` in the device tree directory:

```
device/motorola/sorap/lineage_sorap.mk
```

Add the following configurations:

```make
# Inherit from those products. Most specific first.
$(call inherit-product, $(SRC_TARGET_DIR)/product/core_64_bit.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/full_base_telephony.mk)

# Inherit some common Lineage stuff.
$(call inherit-product, vendor/lineage/config/common_full_phone.mk)

# Inherit from sorap device
$(call inherit-product, device/motorola/sorap/device.mk)

# Device identifier. This must come after all inclusions
PRODUCT_DEVICE := sorap
PRODUCT_NAME := lineage_sorap
PRODUCT_BRAND := motorola
PRODUCT_MODEL := moto g75 5G
PRODUCT_MANUFACTURER := motorola

PRODUCT_GMS_CLIENTID_BASE := android-motorola

PRODUCT_BUILD_PROP_OVERRIDES += \
    PRIVATE_BUILD_DESC="paros_g-user 13 TP1A.220624.014 9b001 release-keys"

BUILD_FINGERPRINT := motorola/paros_g/paros:13/TP1A.220624.014/9b001:user/release-keys
```

## Step 6: Configure properties.mk

Create a new file `properties.mk` in the device tree directory:

```
device/motorola/sorap/properties.mk
```

Add the following configurations:

```make
# Properties
PRODUCT_PROPERTY_OVERRIDES += \
    ro.product.motodesktop=1 \
    ro.product.name=paros_g \
    ro.product.odm.brand=motorola \
    ro.product.odm.device=paros \
    ro.product.odm.manufacturer=motorola \
    ro.product.odm.model=paros \
    ro.product.odm.name=paros \
    ro.product.product.brand=motorola \
    ro.product.product.manufacturer=motorola \
    ro.product.product.model=moto g75 5G \
    ro.product.product.name=paros_g \
    ro.product.system.brand=motorola \
    ro.product.system.device=msi \
    ro.product.system.manufacturer=motorola \
    ro.product.system.model=moto g75 5G \
    ro.product.system.name=paros_g \
    ro.product.system_ext.brand=motorola \
    ro.product.system_ext.device=msi \
    ro.product.vndk.version=34 \
    ro.property_service.version=2 \
    ro.sf.lcd_density=400 \
    ro.soc.manufacturer=QTI \
    ro.soc.model=SM6475 \
    ro.system.build.version.qcom=AU_LINUX_ANDROID_LA.QSSI.14.0.R1.14.00.00.1001.115.00 \
    ro.system.product.cpu.abilist=arm64-v8a,armeabi-v7a,armeabi \
    ro.system.product.cpu.abilist32=armeabi-v7a,armeabi \
    ro.system.product.cpu.abilist64=arm64-v8a \
    ro.zygote=zygote64_32 \
    vendor.camera.aux.packagelist=com.motorola.camera2,com.motorola.camera3,com.motorola.motocit \
    vendor.camera.aux.packagelist2=com.motorola.ccc,com.android.settings,com.motorola.handycam \
    vendor.display.lcd_density=480 \
    vendor.display.limit_low_framerate_layer_name_exception_list=AOD,Wallpaper,testSurface \
    vendor.ril.gsm.version.baseband=M6475_DE311_17.195.01.43R \
    gsm.version.baseband=M6475_DE311_17.195.01.43R PAROS_PVT_EMEADSDS_CUST \
    gsm.version.baseband1=M6475_DE311_17.195.01.43R PAROS_PVT_EMEADSDS_CUST \
    gsm.version.ril-impl=Qualcomm RIL 1.0
```

## Step 7: Add vendor files

Create a new directory for vendor files:

```bash
mkdir -p vendor/motorola/sorap
```

Add proprietary vendor files to this directory.

## Step 8: Initialize repo and sync

Run the following commands to initialize the repo and sync the source code:

```bash
repo init -u https://github.com/LineageOS/android.git -b lineage-20
repo sync
```

## Step 9: Build LineageOS

Run the following commands to build LineageOS:

```bash
source build/envsetup.sh
lunch lineage_sorap-userdebug
make -j$(nproc) bacon
```

## Step 10: Flash the build

After the build is complete, flash the resulting images to the device using fastboot:

```bash
fastboot flash boot boot.img
fastboot flash dtbo dtbo.img
fastboot flash recovery recovery.img
fastboot flash system system.img
fastboot flash vendor vendor.img
fastboot flash product product.img
fastboot flash system_ext system_ext.img
fastboot flash odm odm.img
```