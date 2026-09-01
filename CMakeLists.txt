cmake_minimum_required(VERSION 3.20)

project(linux_alioth NONE)

include(ProcessorCount)
ProcessorCount(BUILD_JOBS)

if(NOT BUILD_JOBS OR BUILD_JOBS EQUAL 0)
    set(BUILD_JOBS 1)
endif()

find_program(MAKE_EXECUTABLE make REQUIRED)

set(KERNEL_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(KERNEL_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}/kernel")

file(MAKE_DIRECTORY "${KERNEL_BUILD_DIR}")

add_custom_target(kernel ALL
    COMMAND "${MAKE_EXECUTABLE}" --no-silent -C "${KERNEL_SOURCE_DIR}" O=${KERNEL_BUILD_DIR} LLVM=1 ARCH=arm64 LOCALVERSION= alioth_defconfig
    COMMAND "${KERNEL_SOURCE_DIR}/scripts/config" --file "${KERNEL_BUILD_DIR}/.config" --disable EFI_ZBOOT
    COMMAND "${MAKE_EXECUTABLE}" --no-silent -C "${KERNEL_SOURCE_DIR}" O=${KERNEL_BUILD_DIR} LLVM=1 ARCH=arm64 LOCALVERSION= olddefconfig
    COMMAND "${MAKE_EXECUTABLE}" --no-silent -C "${KERNEL_SOURCE_DIR}" O=${KERNEL_BUILD_DIR} LLVM=1 ARCH=arm64 LOCALVERSION= -j${BUILD_JOBS} Image modules dtbs
    BYPRODUCTS
        "${KERNEL_BUILD_DIR}/arch/arm64/boot/Image"
        "${KERNEL_BUILD_DIR}/arch/arm64/boot/dts/qcom/sm8250-xiaomi-alioth.dtb"
    USES_TERMINAL
    VERBATIM
)
