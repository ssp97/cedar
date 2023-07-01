# Allwinner CedarX Driver for Mainline Linux 5.15
VideoEngine driver based on A40i SDK  
Ion driver based on Google Android Ion

## Install ve driver

Put ve folder file in `drivers/staging/media/sunxi/cedar_ve`  
Add source include to `drivers/staging/media/sunxi/Kconfig`
```
# SPDX-License-Identifier: GPL-2.0
config VIDEO_SUNXI
    bool "Allwinner sunXi family Video Devices"
    depends on ARCH_SUNXI || COMPILE_TEST
    help
      If you have an Allwinner SoC based on the sunXi family, say Y.

      Note that this option doesn't include new drivers in the
      kernel: saying N will just cause Kconfig to skip all the
      questions about Allwinner media devices.

if VIDEO_SUNXI

source "drivers/staging/media/sunxi/cedar_ve/Kconfig"

endif
```

Add obj to `drivers/staging/media/sunxi/Makefile`
```
obj-$(CONFIG_VIDEO_SUNXI_CEDAR_VE)    += cedar_ve/
```
## Install ion support

Put ion folder file in `drivers/staging/android/`  
Add source include to `drivers/staging/Kconfig`  
```
source "drivers/staging/android/Kconfig"
```

Add obj to `drivers/staging/Makefile`  
```
obj-$(CONFIG_ANDROID)		+= android/
```

Add obj to `drivers/staging/android/Makefile`
```
obj-$(CONFIG_VIDEO_SUNXI_CEDAR_ION) += ion/
```

Add source include to `drivers/staging/android/Kconfig` in the  `if ANDROID` condition.
```
# SPDX-License-Identifier: GPL-2.0
menu "Android"

config ANDROID
    bool "Android Drivers"
    default y if ARCH_SUNXI
    help
      Enable support for various drivers needed on the Android platform

if ANDROID

config ASHMEM
    bool "Enable the Anonymous Shared Memory Subsystem"
    depends on SHMEM
    help
      The ashmem subsystem is a new shared memory allocator, similar to
      POSIX SHM but with different behavior and sporting a simpler
      file-based API.

      It is, in theory, a good memory allocator for low-memory devices,
      because it can discard shared memory units when under memory pressure.

source "drivers/staging/android/ion/Kconfig"

endif # if ANDROID

endmenu
```

## DeviceTree
Example for Allwinner H3 (The "7" in the example represents CLK_PLL_VE because CLK_PLL_VE is not defined in sun8i-h3-ccu.h.)
```
/ {
    reserved-memory {
        #address-cells = <1>;
        #size-cells = <1>;
        ranges;

        cma_pool: cma@4a000000 {
            compatible = "shared-dma-pool";
            size = <0x6000000>;
            alloc-ranges = <0x4a000000 0x6000000>;
            reusable;
            linux,cma-default;
        };

        ion_carveout: vecarveout@50600000 {
            compatible = "ion-region";
            size = <0xFA00000>;
            alloc-ranges = <0x50600000 0xFA00000>;
            no-map;
        };
    };

    ve: ve@1c0e000 {
        compatible = "allwinner,sunxi-cedar-ve";
        reg = <0x01c0e000 0x1000>,
            <0x01c00000 0x10>,
            <0x01c20000 0x800>;
        clocks = <&ccu 7>, <&ccu CLK_VE>, 
                <&ccu CLK_DRAM_VE>, <&ccu CLK_BUS_VE>;
        clock-names = "pll", "mod", "ram", "ahb";
        resets = <&ccu RST_BUS_VE>;
        interrupts = <GIC_SPI 58 IRQ_TYPE_LEVEL_HIGH>;
        allwinner,sram = <&ve_sram 1>;
    };
            
    ion: ion {
        compatible = "allwinner,sunxi-ion";
        status = "disabled";
        heap_carveout@0{
            compatible = "allwinner,carveout";
            heap-name  = "carveout";
            heap-id    = <0x4>;
            heap-base  = <0x50600000>;
            heap-size  = <0xFA00000>;
            heap-type  = "ion_carveout";
            memory-region = <&ion_carveout>;
        };
    };
}
```

## Compile
Enable Driver in
```
> Device Drivers > Staging drivers > Android
[*]  Android Drivers
<*>    Allwinner CedarX Ion Driver

> Device Drivers > Staging drivers > Media staging drivers
[*]  Allwinner sunXi family Video Devices
[M]  Allwinner CedarX VideoEngine Driver
```

... and here we go.


## Userspace library
https://github.com/aodzip/libcedarc
