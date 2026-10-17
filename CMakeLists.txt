cmake_minimum_required(VERSION 3.20)

project(linux_alioth NONE)

include(ProcessorCount)
ProcessorCount(BUILD_JOBS)

if(NOT BUILD_JOBS OR BUILD_JOBS EQUAL 0)
    set(BUILD_JOBS 1)
endif()

find_program(MAKE_EXECUTABLE make REQUIRED)
find_program(MKBOOTIMG_EXECUTABLE mkbootimg)

set(KERNEL_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(KERNEL_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}/kernel")
set(MODULES_STAGING_DIR "${CMAKE_CURRENT_BINARY_DIR}/modules")
set(BOOT_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/boot")
set(KERNEL_MAKE "${MAKE_EXECUTABLE}" --no-silent -C "${KERNEL_SOURCE_DIR}" O=${KERNEL_BUILD_DIR} LLVM=1 ARCH=arm64 LOCALVERSION=)

set(ALIOTH_BOOT_HEADER_VERSION 2 CACHE STRING "Android boot image header version accepted by alioth ABL")
set(ALIOTH_BOOT_PAGESIZE 4096 CACHE STRING "Android boot image page size")
set(ALIOTH_FLASH_DIR "/mnt/e/Android/k40-linux" CACHE PATH "Windows-visible directory receiving boot.img for fastboot")
set(ALIOTH_CMDLINE "root=PARTLABEL=userdata rootwait rw console=tty0 loglevel=7" CACHE STRING "Kernel command line embedded in boot.img")
set(ALIOTH_WIFI_MAC "02:00:db:b9:13:72" CACHE STRING "Wi-Fi MAC written to the boot.img DTB, 02:00 + androidboot.serialno; empty keeps firmware MAC")
set(ALIOTH_WIFI_NODE "/soc@0/pcie@1c00000/pcie@0/wifi@0" CACHE STRING "DT path of the QCA6390 node")
set(BOOT_DTB "${BOOT_OUTPUT_DIR}/sm8250-xiaomi-alioth.dtb")
string(REPLACE ":" ";" ALIOTH_WIFI_MAC_BYTES "${ALIOTH_WIFI_MAC}")

file(MAKE_DIRECTORY "${KERNEL_BUILD_DIR}")

add_custom_target(kernel ALL
    COMMAND ${KERNEL_MAKE} alioth_defconfig
    COMMAND "${KERNEL_SOURCE_DIR}/scripts/config" --file "${KERNEL_BUILD_DIR}/.config" --disable EFI_ZBOOT
    COMMAND ${KERNEL_MAKE} olddefconfig
    COMMAND ${KERNEL_MAKE} -j${BUILD_JOBS} Image.gz modules dtbs
    BYPRODUCTS
        "${KERNEL_BUILD_DIR}/arch/arm64/boot/Image"
        "${KERNEL_BUILD_DIR}/arch/arm64/boot/Image.gz"
        "${KERNEL_BUILD_DIR}/arch/arm64/boot/dts/qcom/sm8250-xiaomi-alioth.dtb"
    USES_TERMINAL
    VERBATIM
)

add_custom_target(modules
    COMMAND "${CMAKE_COMMAND}" -E rm -rf "${MODULES_STAGING_DIR}"
    COMMAND ${KERNEL_MAKE} INSTALL_MOD_PATH=${MODULES_STAGING_DIR} INSTALL_MOD_STRIP=1 modules_install
    DEPENDS kernel
    USES_TERMINAL
    VERBATIM
)

add_custom_target(rootfs
    COMMAND "${KERNEL_SOURCE_DIR}/alioth/mkrootfs.sh" "${KERNEL_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}"
    DEPENDS modules
    BYPRODUCTS "${BOOT_OUTPUT_DIR}/initrd.img" "${CMAKE_CURRENT_BINARY_DIR}/rootfs.simg"
    USES_TERMINAL
    VERBATIM
)

if(ALIOTH_WIFI_MAC)
    list(LENGTH ALIOTH_WIFI_MAC_BYTES ALIOTH_WIFI_MAC_LEN)
    if(NOT ALIOTH_WIFI_MAC_LEN EQUAL 6)
        message(FATAL_ERROR "ALIOTH_WIFI_MAC must be xx:xx:xx:xx:xx:xx, got ${ALIOTH_WIFI_MAC}")
    endif()
    find_program(FDTPUT_EXECUTABLE fdtput REQUIRED)
    set(BOOT_DTB_FIXUP COMMAND "${FDTPUT_EXECUTABLE}" -t bx "${BOOT_DTB}" "${ALIOTH_WIFI_NODE}" local-mac-address ${ALIOTH_WIFI_MAC_BYTES})
endif()

if(MKBOOTIMG_EXECUTABLE)
    add_custom_target(bootimg
        COMMAND "${CMAKE_COMMAND}" -E copy "${KERNEL_BUILD_DIR}/arch/arm64/boot/dts/qcom/sm8250-xiaomi-alioth.dtb" "${BOOT_DTB}"
        ${BOOT_DTB_FIXUP}
        COMMAND "${MKBOOTIMG_EXECUTABLE}" --header_version ${ALIOTH_BOOT_HEADER_VERSION} --pagesize ${ALIOTH_BOOT_PAGESIZE} --base 0x00000000 --kernel_offset 0x00008000 --ramdisk_offset 0x01000000 --tags_offset 0x00000100 --dtb_offset 0x01f00000 --os_version 11.0.0 --os_patch_level 2021-12 --kernel "${KERNEL_BUILD_DIR}/arch/arm64/boot/Image.gz" --ramdisk "${BOOT_OUTPUT_DIR}/initrd.img" --dtb "${BOOT_DTB}" --cmdline "${ALIOTH_CMDLINE}" -o "${BOOT_OUTPUT_DIR}/boot.img"
        COMMAND "${CMAKE_COMMAND}" -E copy "${BOOT_OUTPUT_DIR}/boot.img" "${ALIOTH_FLASH_DIR}/boot.img"
        DEPENDS kernel
        BYPRODUCTS "${BOOT_OUTPUT_DIR}/boot.img" "${BOOT_DTB}"
        USES_TERMINAL
        VERBATIM
    )
endif()
