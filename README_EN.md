# Linux 7.2.2 for Xiaomi alioth

[简体中文](README.md) | [English](README_EN.md)

Linux 7.2.2 mainline kernel port for the Redmi K40 / POCO F3 (alioth).

## Building

Install the dependencies:

```bash
sudo apt install bc bison clang cmake device-tree-compiler dwarves flex libelf-dev libssl-dev lld llvm make
```

Clone the repository:

```bash
git clone https://github.com/Techuouo520/linux7.2.2-alioth-mainline.git
cd linux7.2.2-alioth-mainline
```

Build:

```bash
mkdir build
cd build
cmake ..
cmake --build .
```

The build uses `arch/arm64/configs/alioth_defconfig`.

Build outputs:

```text
build/kernel/arch/arm64/boot/Image
build/kernel/arch/arm64/boot/dts/qcom/sm8250-xiaomi-alioth.dtb
```

## License

GPL-2.0-only
